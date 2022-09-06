---
title: "Goにコントリビュートするための環境をVS Codeで設定する"
emoji: "🐦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["go","vscode"]
published: true
published_at: 2022-09-06 17:19
---
VS Codeの設定は下記記事の最後の方に説明がありますが、swigのインストールなど書かれていなかったため補足記事として書かせていただきます。
# CLA の同意とアカウント作成
下き記事に従って進める。
https://ema-hiro.hatenablog.com/entry/2021/12/07/000040
または公式のガイドに従う。
https://go.dev/doc/contribute

Contributionはせずに手元でソースからGoを動かしたいだけであれば、[GitHub](https://github.com/golang/go)からクローンしてくるでも良いです。

# Goの最新ソースをクローンしてくる
CLA の同意とアカウント作成を行なった場合
```bash
$ git clone https://go.googlesource.com/go
```
GitHubからクローンする場合
```bash
# SSH
$ git clone git@github.com:golang/go.git
# HTTPS
$ git clone https://github.com/golang/go.git
```
# Goのテストとビルド実行
[公式ガイドライン](https://go.dev/doc/contribute#sending_a_change_gerrit)に従って行います。
※環境によりますが15分くらいかかります。
```bash
$ cd go/src
$ ./all.bash
```
go/bin以下にGoのバイナリが生成されます。

# VS Codeで最新Goの開発環境を整える
Goのrepositoryをvscodeで開きます。
```bash
$ cd go
$ code .
```

go/.vscode/settings.jsonを作成し、ソースからビルドしたGoを使用するように指定する。
```json
{
	"go.goroot": "Goのrepositoryの絶対path",
	"gopls": {
		"build.experimentalWorkspaceModule": true // go.modがプロジェクトルートにない場合にはtrueにする必要がある
	}
}
```

## swigのエラーが出る場合
私の場合はローカルにswigを入れていないためVS Codeでエラーが出ていました。
![](/images/go-contribute-vscode-setting/swig-error.png)
Macなので下記コマンドでswigを入れましたが、Windowsなどの場合でもインストール後PATHを通せば問題ないはずです。
```bash
brew install swig
```
swig入れた後はmisc以下のみがエラーとなっておりましたが、`misc/swig/stdio/file_test.go`をVS Codeでデバッグ実行できたので問題なさそうです。
![](/images/go-contribute-vscode-setting/misc-error.png)

## テストをdebug実行してみる
`src/errors/errors_test.go`を開き、TestNewEqualにブレイクポイントを貼った上で、`debug test`を実行します。
![](/images/go-contribute-vscode-setting/debug-test.png)
テストがdebug実行されブレイクポイントが止まれば環境構築完了です。


