tags: hachioji-tech
pubdate: 2013-09-18 23:57

---

※この記事は, [@Spring_MT](https://twitter.com/Spring_MT)さんから寄稿頂いた記事をpapixが代理投稿したものです.

# perlでTravisCI、Coverallsを使ってみる
最近東京に戻ってきた[Spring_MT](https://twitter.com/Spring_MT)です。

perlで自分でモジュールを書く準備をと思って、TravisCI、Coverallsを使う方法を調べてみました。
だいぶperlからは離れていたので、perlを入れるとことから調べてます。

## plenvを入れる
今回はplenvを使ってperlを入れました。

[plenv](https://github.com/tokuhirom/plenv)

plenvでperlを入れて、cpanmも入れていきます。

    % git clone git://github.com/tokuhirom/plenv.git ~/.plenv
    % echo 'export PATH="$HOME/.plenv/bin:$PATH"' >> % ~/.zshrc
    % echo 'eval "$(plenv init -)"' >> ~/.zshrc
    % exec $SHELL -l
    % plenv install 5.18.0
    % plenv local 5.18.0
    % plenv rehash
    % plenv install-cpanm
    % plenv rehash

## Minillaを使ってひな形を作る
最初はModule::Installを使いながら試していたのですが、TravisCIでこけて先に進めず。。。

で、[この会話](http://lingr.com/room/perl_jp/archives/2013/08/29#message-16407055)で勧められた通り、Minillaを使って作ってみました。

[Minilla](https://github.com/tokuhirom/Minilla)
[Minilla を用いた Perl モジュールの作り方](http://blog.64p.org/entry/2013/05/14/080423)

    % cpanm install Minilla
    % plenv rehash
    % minil new Sample
    % cd Sample
    % git add .
    % git commit

Minillaはこれだけで新しいモジュールのひな形ができちゃいます！
ここまでできたら、githubでレポジトリを作って、TravisCIのサイトでリポジトリをsyncし、TravisCIの設定をonにしましょう。
TravisCI設定はこちらがめっちゃ参考になります。

[GithubにあるリポジトリをTravis CI連携する手順 #junitbook](http://sue445.hatenablog.com/entry/2013/06/01/170607)

Minillaで作ったモジュールのテストを手元で走らせる場合は、minil testを実行すると思うのですが、TravisCIではデフォルトで下記コマンドでテストが実行されます。

    perl Build.PL && ./Build test 

[Building a Perl Project](http://about.travis-ci.org/docs/user/languages/perl/#Module%3A%3ABuild)

Minilaで作ったデフォルトのBuild.PLを使うと、TravisCIではxt/配下のテストは走らないのです。

ここまでできたら、pushします。
Minillaで作ったモジュールには既に .travis.yml が用意されているので、何もしなくてもtravisが動きます。

![image](<: '/static/image/perl_minilla_1.jpeg' | uri_for :>)
[TravisCI SpringMT/Sample](https://travis-ci.org/SpringMT/Sample)

うまくいきました！

バッジも貼りましょう。
TravisCIのビルド結果ページ右上のbuild passingの画像をクリックすると、そのまま貼り付けられる状態のテキストが表示されるので、それをコピーしてREADME.mdに貼り付けます。

ついでに、Coverallsも追加しちゃいましょう。

[Coveralls](https://coveralls.io)

まずは、TravisCIと同様に、Coverallsにリポジトリを登録してください。
perlの場合は、Devel::Cover::Report::Coverallsを使えば、Coverallsに簡単にカバレッジの情報を登録できます。

[coverallsをperlなprojectで使えるようにしてみた](http://memo.fushihara.net/post/48086316824/coveralls-perl-project)

Coverallsに情報を投げるために .travis.ymlに下記を追記してください。

    before_install:                                                                                                                                 
      cpanm -n Devel::Cover::Report::Coveralls
    script:
      perl Build.PL && ./Build build && cover -test -report coveralls

pushすると、こんな感じになると思います。

![image](<: '/static/image/perl_minilla_2.jpeg' | uri_for :>)

[Coveralls SpringMT / Sample](https://coveralls.io/r/SpringMT/Sample)

## 最後に
最終的に少しテストを追加しました。

[GitHub SpringMT/Sample](https://github.com/SpringMT/Sample)

githubにページはこんな感じでバッジも出ています。

![image](<: '/static/image/perl_minilla_3.jpeg' | uri_for :>)

これでモジュールを書き始める準備が出来上がったので、なんか書きます。

## ちょっとハマったところ
新しく作ったファイルにテストを書いてminil testを実行したけど、テストがされなくてなんでだろうと思ってたら、git管理下じゃないからだった。。。。

