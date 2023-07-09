---
title: "TinyGo+VSCodeでWasmを開発する環境を整える"
emoji: "🐦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["go"]
published: true
published_at: 2023-07-09 12:34
---
知人に質問されたのでざっくり書く。

## TinyGo公式docに従ってシュッとインストールする

<https://tinygo.org/getting-started/install/>
バージョンがでるところまでやってください。

```bash
$ tinygo version
tinygo version 0.28.1 linux/amd64 (using go version go1.20 and LLVM version 15.0.0)
```

## TinyGo拡張を入れる

<https://marketplace.visualstudio.com/items?itemName=tinygo.vscode-tinygo>

## ビルドターゲットをwasmにする

### VSCodeのコマンドパレッドで「TinyGo target」と入力

pick a targetはwasmを選択してください。
![VSCodeのコマンドパレッド操作](/images/tinygo-wasm-for-cloudflare-workers/command.png)

syumaiさんの<https://github.com/syumai/workers>などのテンプレを生成してVSCodeで開いてみてください。

コードジャンプがWasmでも機能するようになりました。
