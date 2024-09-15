---
title: "Goのイテレータの挙動をなんとかして理解する"
emoji: "🐦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["go"]
published: true
published_at: 2024-09-15 23:55
---

## はじめに

Go1.23から使えるようになったイテレータをワクワクしながら実装してみようとしました。
しかし、シグネチャの理解すら難しく、皆さんが公開してくれている解説記事を読んでも手が動かせない状態になっていました。
特にyiledの挙動がシグネチャから読み取ることができずに理解に時間がかかりました。

もしかするとGoのイテレータの実装には一定のハードルがあるかもしれません。
そこで、1人でも多くの方がイテレータを詳しく解説している記事を読むことや、実装自体のとっかかりになれるように、私の独自解釈を説明してみます。


## まずはiterパッケージなしで、range-over-funcから

ここではyiled()はforブロックの中に戻って実行するための関数として解釈してください。
このyiled()の挙動さえ理解できればなんとなくイテレータも実装できるようになってきます。

```go
package main

import (
	"fmt"
)

func main() {
	// 1. forに渡したhoge()を呼び出し
	for range hoge {
		// 4. yiledの呼び出しによりforブロック内が実行される
		fmt.Println("loop内だよ")
	}
}

// 2. hoge()を実行
func hoge(yield func() bool) {
	fmt.Println("hoge内だよ")
	// 3. forブロックに戻ってloopを一回実行するためのyiled()が呼び出される
	forResult := yield()
	// 5. yield()の戻り値boolはforブロックが最後の行に中断なく実行されればtrueで帰ってくる
	// forブロックにはbreakなど中断する処理がないので、forResultにtrueが入ってくる
	fmt.Println(forResult)
}
```

理解するポイントとしては以下です。
- for文に関数hogeを渡している。
- 渡した関数hogeの中でyiledを呼び出すとforブロックの中に戻る。
    - yiledはforブロックの中身が入った関数と思ってもいいかもしれないです。
- forブロックが中断なく最後の行まで実行されると、yiledがtrueを返した上で、関数hogeのyiled呼び出し箇所へ戻る。
- 関数hogeが最後の行まで実行されるとfor文も抜ける。

出力結果

```bash
$ go run .
hoge内だよ
loop内だよ
true
```

## iterパッケージの雰囲気を理解する

iterパッケージにはイテレータを実装するのに便利な方が提供されています。

```go
type (
	Seq[V any]     func(yield func(V) bool)
	Seq2[K, V any] func(yield func(K, V) bool) // 本記事ではこっちは実装しません。
)
```

range-over-funcで実装したhoge()と比較するとyiled()の引数に値を渡せるようになってそうですね。
この引数は`for v:=range hoge`と書くとき、vに入ってくる値になってきます。
iterパッケージを利用する上で、理解するポイントとしては以下です。

- yiledを呼び出すとforブロックの中に戻る。(大事なことなので)
- yiled(v)の引数は`for v:=range hoge`でforブロックに渡す変数v
- yiledの戻り値boolはforブロックが最後の行まで中断なく実行されたかの結果が入っている(大事なことなので)


```go
package main

import (
	"fmt"
	"iter"
)

func main() {
	// 1. forに渡したgetListValueを呼び出し
	for i := range getListValue([]int{4, 2, 32, 44, 5}) {
		// 5.yiled(v)で渡した変数vがiに代入され、標準出力される
		fmt.Println(i)
		if i == 44 {
			// 7. forブロックが最後の行に到達せずに、途中で中断される
			break
		}
	}
}

// 2. getListValue()を実行
func getListValue(list []int) iter.Seq[int] {
	return func(yield func(int) bool) {
		// 3. listの中身を今までのfor分で中身を取り出し、変数vに代入
		for _, v := range list {
			// 4. vをforブロックに渡し、forブロックを実行
			if !yield(v) {
				// 8. forブロックが中断されたので、yiledはfalseを返す
				return // 9. forブロックに渡している関数getListValueが終了するので、for文自体も抜ける
			}
			// 6. forブロックが最後まで実行されたので、yiledはtrueを返し、if文の中には入らずこの行まで実行される
			// `for _, v := range list`に戻りforを実行
		}
	}
}

```

出力結果

```bash
4
2
32
44
```

## 終わりに

yiledはforブロックの中身が入った関数と解釈するとことで、イテレータの挙動を理解した気になれる気がします。
この記事は理解した気になるためのとっかかりになることを目的としているので、他の詳しいイテレータの記事を読んだりして正しく理解を深めてください。