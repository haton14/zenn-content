---
title: "Goのイテレータの挙動をなんとかして理解する"
emoji: "🐦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["go"]
published: false
published_at: 2024-09-15 17:19
---

## はじめに

私はGoのイテレータの挙動が初めてifやforを書いたとき並みに理解できず、ちょっとこれは書く機会すら訪れないかも。。。と焦りました。

私の独自解釈にはなりますですが、少しでも多くの方がイテレータの実装方法を理解できるように説明していきます。


## まずはiterパッケージなしで、range-over-funcから

ここではyiled()はforブロックに戻って実行するための関数として解釈してください。
このyiled()の挙動さえ理解できればなんとなくイテレータも実装できるようになってきます。

理解するポイントとしては以下です。
- for文に関数hogeを渡しているいる。
- 渡した関数の中でyiledを呼び出すとforブロックに戻る。
    - yiledはforブロックの中身が入った関数と思ってもいいかも。
- forブロックが中断なく最後行まで実行されると、yiledがtrueを返した上で、関数のyiled呼び出し箇所へ戻る。
- 関数が最後まで実行されるとfor文も抜ける。

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

出力結果

```bash
$ go run .
hoge内だよ
loop内だよ
true
```

