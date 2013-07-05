## 概要

 * Mac ではない Linux 環境を仕事で使う際の小技をシチュエーション別に整理した
 * 一部は Mac でも使えるかもしれない 

***
  
## デュアルディスプレイ

### xrandr

`xrandr`  コマンドを使うことでデュアルディスプレイを設定する。`Ubuntu` ではプラグアンドプレイで宜しくやってくれるが、ノート PC だけの利用にすると自動的に切り替わってくれないので、`xrandr`  を叩くシェルするクリプトを用意している。

```
xrandr --output LVDS1 --mode 1366x768
xrandr --output HDMI1 --mode 1920x1200 --left-of LVDS1
```

`LVDS1` がノート PC  のモニタで `HDMI1` が外付けのモニタを示す。これを実行することでデュアルディスプレイ環境が設定される。ちなみに `--left-of` で `HDML1` は `LVDS1` の左に設置となる。

***

## 作図

なんやかんやで簡単なシステム構成図やネットワーク図を書かなきゃいけない時  Mac の場合には `Cacoo` を選ぶ。また、 Linux 環境では `Flash` がちゃんと動けば問答無用に `Cacoo` を選ぶことになるが…自分の環境では残念ながら思ったように `Cacoo` が使えないので、以下のツールを適材適所て使うことにしている。

### [blockdiag](http://blockdiag.com/ja/blockdiag/)

Sphinx というドキュメントビルダーの為に作られた作図ツール。もちろん Python で書かれている。作者は [@tk0miya](https://twitter.com/tk0miya) さん。

#### セットアップ

セットアップは以下のように `python-pip` をインストール後、`pip` コマンドでインストールする。

```
sudo apt-get install python-pip
sudo pip install blockdiag
```

#### 使ってみる

以下のようなテキストファイルを作成する。

```
blockdiag {
  orientation = portrait
  //
  A[label = "redis router"];
  B[label = "redis 6379"];
  C[label = "redis 6380"];
  //
  A -> B,C
  //
  group {
    label = "redis server";
    B; C;
  }
}
```

以下のようにテキストファイルを引数として `blockdiag` コマンドを実行する。

```
blockdiag sample.diag
```

正常に終了すると以下のような図が作成される。

<p><span itemscope itemtype="http://schema.org/Photograph"><img src="http://cdn-ak.f.st-hatena.com/images/fotolife/i/inokara/20130630/20130630115106.png" alt="f:id:inokara:20130630115106p:plain" title="f:id:inokara:20130630115106p:plain" class="hatena-fotolife" itemprop="image"></span></p>

様々なオプションを引数として渡すことでフォントのサイズ等の変更を行うことが出来る。また、[ドキュメント](http://blockdiag.com/ja/blockdiag/)が充実しているのでイザという時にも安心！

### [graph-easy](http://search.cpan.org/~tels/Graph-Easy/)

Perl で書かれた作図ツール。アスキーアート以外にも HTML 等でも書き出せる。

#### セットアップ

Perl のパッケージは `cpanm` でインストールするのがナウいのかな...

```
cpanm install Graph::Easy
```

インストールしたらパスを通してあげるのがナウいのかな...

```
export PATH="$HOME/perl5/bin:$PATH"
export PERL5LIB="$HOME/perl5/lib/perl5"
```

#### こんな感じで使う

簡単な図であれワンライナーでなくてもいける。

```
echo "[ user ] --> [ proxy ] --> [ web1 ], [ web2 ]" | graph-easy
```

以下のようにアスキーアートで図が出来上がる。これはイイ。

```
+------+     +-------+     +------+
| user | --> | proxy | --> | web1 |
+------+     +-------+     +------+
               |
               |
               v
             +-------+
             | web2  |
             +-------+

```

## 情報収集

情報収集は主に twitter ということでコマンドラインから twitter にアクセスするツールを二つほど。

### [knife-twitter](https://github.com/higanworks/knife-twitter)

[@sawanoboly](https://twitter.com/sawanoboly) さんが書かれた knife コマンドのプラグイン。

#### インストール

漢は黙って gem でインストール。

```
sudo gem knife-twitter --no-ri --no-rdoc -V
```

最近は `-V` を付けるのが個人的なブーム。

#### こんな感じで使う。

タイムラインの取得。

```
knife twitter tl
```

ポストは以下のように...。

```
knife twitter post -m "hogehoge"
```

### [termtter](https://github.com/jugyo/termtter)

ターミナルから利用出来る Twitter クライアント。

#### インストール

knife-twitter と同様に漢は黙って gem でインストール。

```
sudo gem termtter --no-ri --no-rdoc -V
```

いやー、ほんま便利になりましたなー。

#### 使ってみる

コマンドプロンプトから `termtter` を実行すると、初回の場合にはブラウザが起動してアクセストークンの取得を行う。ブラウザ上に現れたトークンをプロンプトで待ち構える `termtter` に登録して利用を開始する。

現在はタイムラインを追っかけることでしか利用していないが、仕事中でもイチイチ GUI なクライアントを立ち上げる手間が省けるのと Twitter 見てても仕事している感がある（笑）ので良い...かな。

***

## ということで…

`Windows` や `Mac` と比較すると、どうしても一手間増えてしまうけど、その手間もスキルアップの一つとポジに捉えて暫くは頑張ってみようと思う。ちなみに、仕事はサボってはいない。
