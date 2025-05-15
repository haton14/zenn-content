---
title: "MacでHomebrewを使用せずにGoをセットアップする(Ubuntuも)"
emoji: "🐦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["go"]
published: true
published_at: 2022-09-20 16:00
---
Homebrewは便利ですが、時々Goとは直接関係ないXcodeをアップデートしろとMacに怒られ10分以上待たされることがしばしばあります。
10分もあれば`Hello World!`が何回も書けますね。。
そんな時間が勿体ないと感じる方向けにGo公式のArchiveからローカルにインストールする方法を紹介します。

[Goをすでに直接入れている方はこの部分をコピペすれば最新になります](https://zenn.dev/haton14/articles/go-setup-for-mac#%E3%83%AD%E3%83%BC%E3%82%AB%E3%83%AB%E3%81%AEmac%E3%81%ABgo%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB)

## ローカルのMacにGoをインストール済みの場合

全て消してください。

```bash
# Homebrewの場合
$ brew uninstall go
```

GOROOT, GOPATHを環境変数として設定済みの場合は`.zshrc`or`.bashrc` or `.profile`から設定の記述を削除し、ディレクトリも削除します。

```bash
# GOROOT設定されているか確認
$ echo $GOROOT
# 設定されていたらディレクトリ削除、GOROOT設定されていない場合は不要
$ rm -rf $GOROOT

# GOPATH設定されているか確認
$ echo $GOPATH
# 設定されていたらディレクトリ削除、GOPATH設定されていない場合は不要
$ rm -rf $GOPATH
```

環境変数削除

```bash
# GOROOT,GOPATHの設定箇所削除
$ vi ~/.zshrc
```

## ローカルのMacにGoをインストール

[Go公式のLinuxのインストール方法](https://go.dev/doc/install)と同じやり方で行います。
ローカルにGoをインストール(go1.24.3の場合)

すでにこの記事を最後まで実行し、Goのバージョン更新だけしたい方はこの部分だけ実行でOKです。

### intel Mac

```bash
wget https://go.dev/dl/go1.24.3.darwin-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.24.3.darwin-amd64.tar.gz
```

### M1, M2 Mac

```bash
wget https://go.dev/dl/go1.24.3.darwin-arm64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.24.3.darwin-arm64.tar.gz
```

### Linux

```bash
wget https://go.dev/dl/go1.24.3.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.24.3.linux-amd64.tar.gz
```

`.zshenv`or`.bashrc`or`.profile`にPATHを追記する。

```bash
vi ~/.zshenv
```

追記内容

```bash
# GOROOT, GOPATHはgoコマンド内部でしか使用しないので設定不要
# go envで見るとシェルに設定しなくても、goコマンド内部で設定されていることが確認できる
export PATH=$PATH:/usr/local/go/bin
# go installを実行した際のGo製バイナリのインストール先
export PATH=$PATH:$HOME/go/bin
```

インストール確認

```bash
$ go version
go version go1.24.3 darwin/amd64
```

## Goの複数バージョン管理(おまけ)

以下を参考に公式のやり方でできます。
<https://go.dev/doc/manage-install>

インストール後vscodeの左下の`Go 1.xx.x`をクリックし、`Choose Go Environment`からバージョン切り替えできます。
![vscode上でのGoを切り替える画像](/images/go-setup-for-mac/vscode.png)
