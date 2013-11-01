tags: Perl
---
# Minilla（Module::Build）でproveオプションを渡したい

最近はやりのモジュールセットアップ＆管理ツールの[Minilla](http://search.cpan.org/~tokuhirom/Minilla/)で使うminil testでprove --jobs 10的な並列処理をかけたいなーと思って、いろいろ調べて教えてもらったのでその話です。

## proveのオプションって？
proveはPerlの動作確認テストを一括実行するコマンドです。proveのオプションとか知らない方は、こちらのxaicronさんの記事を参照→[prove についてのおさらい](http://perl-users.jp/articles/advent-calendar/2011/test/21)

テストの並列処理とかカラーリングとか時間出したりできます。モジュールのテストでも並列処理させたいことはしばしばあると思いますので、結構ニーズあるような設定だと思いますね。

## Makefile.PLの場合
[テストのためにデーモンを自動的に起動するやりかた2011年版](http://perl-users.jp/articles/advent-calendar/2011/test/18)にあるように、.provercとかもりもりかいて、以下のようにMakefile.PLに書けばいいのです。

    sub MY::test_via_harness { "\tprove --rc=t/proverc t\n" }

ただ、Minillaは最近はやりのBuild.PL形式で[Module::Build](http://search.cpan.org/~leont/Module-Build-0.4007/lib/Module/Build.pm)使っているんですが、ドキュメントがモリモリでよくわからん！という感じなりました。ググってもあまり出てこないし。。。

## MinillaのModule::Buildでproveの並列オプションの渡し方
さてはて、困ったなーと思っていたんですが、こんなときこそはちぴー仲間の集うYanchaだ！！ということで聞いてみたら、[gfxさんの記事](http://d.hatena.ne.jp/gfx/20110313/1300027796)をさらっと[tsuttchiさんにおしえてもらった](http://yancha.hachiojipm.org/quot?id=172960,172953,172951,172949)。マジ感謝です！！！で以下のようにproveのオプションは環境変数で設定できるということです。

    HARNESS_OPTIONS=j19 minil test

で、毎回環境変数渡すのだ類のでMinillaにいれたいんですが、Build.PLを自動生成するMinillaなので、どうするかというと、[Minillaのminil.tomlの設定のドキュメント](http://search.cpan.org/~tokuhirom/Minilla/lib/Minilla.pm#CONFIGURATION)とtokuhiromさんの[Module::Buildでテストの前とかあとにフックする方法](http://blog.64p.org/entry/20111202/1322803079)を参考に以下のようになります。

### minil.tomlで以下のようにオレオレbuild_classを設定

    [build]
    build_class = "builder::MyBuilder"

### オレオレbuildクラスで以下のようにテストをフック
./buidler/ディレクトリ作って、vi /builder/MyBuilder.pmに書き込みます

    package builder::MyBuilder;
    use strict;
    use warnings;
    use parent qw(Module::Build);

    sub ACTION_test {
        my $self = shift;

        local $ENV{HARNESS_OPTIONS} = ($ENV{HARNESS_OPTIONS} ||'') . 'j19';
        $self->SUPER::ACTION_test(@_);
    }

    1;


### いろいろ渡す方法
[HARNESS_OPTIONSのドキュメント](http://search.cpan.org/~ovid/Test-Harness-3.29/lib/Test/Harness.pm#ENVIRONMENT_VARIABLES_THAT_AFFECT_TEST::HARNESS)をみると、たくさんオプションを渡すときは、「:」区切りで渡すようです。

    HARNESS_OPTIONS=j9:c make test

### 余談、do_system('prove --jobs 19 -r t')には注意
最初ACTION_testの中で、$self->SUPER::ACTION_test(@_)呼ばないで、$self->do_system('prove --jobs 19 -r t')でやればよくね？って思ってやったんですが、Minillaがいい感じに裏でやっていると思われるxt系のテストがよばれなくなっちゃうので、ちとうれしくないと思いやめました。xt系のテストあまりわかっていないので、とりあえず、tokurhiromオススメを不都合が出るまで使っておきましょうという感じです。

