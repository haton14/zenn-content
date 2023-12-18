---
title: "http clientとしてのリクエストの挙動を変えずに、I/F変更や共通処理剥がしをしたい"
emoji: "🐦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["go"]
published: true
published_at: 2023-12-18 17:19
---
プロダクト内でdao的なI/Fとして切り出されているGoのhttp clientがあり、以下のように中身は複雑に共通化され実際の挙動がどうなっているかわからないという状況はまれにあります。

```go
package dao

// ハイコンテキストな引数(実はjson string)
func Hoge(paramater string) (*HogeResponse, error) {
  // 共通化された関数内では複雑な条件分岐をしている
  path := "/hoge"
  req, err := createHttpRequest(path, paramater)
  return req.call()
}
```

こういう時には呼び出し元からデバッグ実行を繰り返し実際のリクエスト内容を調査することが多いかと思いますが、メモしていなければ寝たら忘れてしまいます。
このような辛い思いをすると、後のもの達の幸せを願い複雑に共通化されたdaoを紐解き、あるべきI/Fへの変更や共通処理剥がしをしようとする優しい方もいるかと思います。

そんな方達のために、http clientの挙動を変えずにI/Fを変えるのに役立ちそうなテスト方法を紹介します。今回はあまり記事になっていることが多くなさそうなリクエストのみについて書きます。

## 結論

ダミーのhttpサーバーを動かしてくれる`httptest.NewServer`の中で、`*http.Request`に詰めている値を直接比較する。
request内容の比較テストがPASSしていれば、http clientとしての挙動を変えずにdaoとしての処理やI/Fを変えることができたことになる。

```go
package dao_test

import (
  "fmt"
  "io"
  "net/http"
  "net/http/httptest"
  "net/url"
  "testing"
  "github.com/stretchr/testify/assert"
  "myproduct/dao"
)

func TestHoge_Request(t *testing.T) {
  assert := assert.New(t)
  svr := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
  // request check
    for k, v := range r.Header {
      httpHeaderCheckHelper(t, k, v)
    }
    assert.Equal("/hoge", r.URL.Path)
    assert.Equal(url.Values{}, r.URL.Query())
    assert.Equal("POST", r.Method)
    // streamなのでbodyが後続で読めなくなるが、request内容のテストとして割り切る
    body, err := io.ReadAll(r.Body)
    if err != nil {
      assert.NoError(err)
    }
    assert.Equal(`{"name":"hoge","secret":"cjadfmemflapd"}`, string(body))
    // dummy response (よしなに)
    fmt.Fprintln(w, `{"dummy": "dummy"}`)
  }))
  defer svr.Close()
  resp, err := dao.Hoge(`{"name":"hoge","secret":"cjadfmemflapd"}`)
  assert.Nil(err)
  assert.NotNil(resp)
}

func httpHeaderCheckHelper(t *testing.T, key string, value []string) {
  if key == "User-Agent" {
    return
  }
  wantHeader := map[string][]string{
    "Accept":          {"application/json"},
    "Content-Type":    {"application/json;charset=UTF-8"},
    "Authorization":   {"Basic U0VSVklDRV9BUElfS0VZOlNFUlZJQ0VfQVBJX1NFQ1JFVA=="},
    "Accept-Encoding": {"gzip"},
  }
  assert.Equal(t, wantHeader[key], value)
}
```

## テストを作っていく過程

愚直にやるだけなので簡単です。
私の場合はですが、実際のプロダクトをローカルでvscodeからデバッグ実行し、標準のhttp packageの中にコードジャンプして`*http.Request`の中身を確認してメモします。
そのメモ内容を`httptest.NewServer`に書いて終わりです。

## 冒頭に書いた関数のI/F変更例

reqeust bodyをjson stringからstructに変更し、どんな値が渡されるのか型からわかるようになりました。

```go
package dao

import myproduct/dto

func Hoge(hoge dto.HogeRequest) (*HogeResponse, error) {
  hogeJson, err := json.Marshal(hoge)
  path := "/hoge"
  // [TODO]必ず共通関数をはがす！！
  req, err := createHttpRequest(path, string(hogeJson))
  return req.call()
}
```

## まとめ

- httptest.NewServerを使って*http.Requestに詰めている値を直接比較する
- テストを作る過程は簡単で、まずhttp packageの中で*http.Requestの中身を確認し、それらをhttptest.NewServerに書くだけ
- request内容の比較テストがPASSしていれば、http clientとしての挙動を変えずにdaoとしての処理やI/Fを安全に変えることができる
