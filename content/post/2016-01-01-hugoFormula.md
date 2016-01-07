+++
date = "2016-01-01T23:41:35+09:00"
draft = false
title = "HugoでGitHub markdown形式でコードブロックを記述する"

+++

ふとブログにコードを載せようとしてmarkdown形式で書いてみたら反映されなかった。
Hugoに設定が要るらしい。

Hugoをbrewからインストールして使っていたが、versionが古く0.14がインストールされていた。
最新のHugoをビルドして使う必要がある。

まずgo を入れていなかったのでインストールする。
```bash
$ brew install go
```

mercurial も入れていなかったのでインストール。
```bash
$ brew install mercurial
```

最新のHugoをビルドして使う。
```bash
$ export GOPATH=$HOME/go
$ go get -v github.com/spf13/hugo
```

$HOME/go/bin にHugoのバイナリが生成されるので、ここをPATHに追加しておく
```bash
$ export PATH=$PATH:$HOME/go/bin
```

Hugoがちゃんと入ったか動作確認する。
```bash
$ hugo version
Hugo Static Site Generator v0.16-DEV BuildDate: 2016-01-01T23:34:08+09:00
```

次にコードのハイライトを解析するPygmentsのインストールをする。
```bash
$ pip install Pygments
```

config.tomlにPygmentsを使うことを設定する。
Pygmentsのスタイルのプレビューは公式サイトで見ることができる。[Pygments](http://pygments.org/)
```cpp
pygmentscodefences = true
pygmentsstyle = "paraiso-light"
```

ただし、Qiitaのmarkdown記法で`bash:filename`とか書いてもファイル名は表示されない
