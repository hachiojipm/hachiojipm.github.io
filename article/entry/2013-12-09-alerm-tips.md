tags: Perl
pubdate: 2013-12-09 23:59
---
# x秒後に消滅するpsgiプロセスをPerlで実装したい

plackupして放置しているプロセスとかそういうのいい感じに消えてほしいのですが、そいうやつの実装についてのアイデアと疑問をば。

## 前置き：要件的なやつ
例えばですが、自分のローカルPCの中にたくさんのプロジェクトが入っていて、それらのプロジェクトを個別に詳しく機能追加とかするわけじゃないんだけど、それぞれの管理画面だけどんなんだったかな？って眺めてみたいとして、一枚の一覧HTMLだけたちあげておいて、そここからリンクをクリックすれば、適当に空いているportゲットして個別のプロジェクトの管理画面用のpsgiファイルをplackupしてそのURLにリダイレクトするみたいなのがほしいとします。

ただ、このとき、いろんなportをばかばかと使ってプロセスが立ち上がりっぱなしになると痛いので、立ち上げたpsgiのプロセスがいい感じに消えてほしいわけですね。

つまり、以下のようなイメージですね。

- http://project_admin:8080/luancher/my_app_1にアクセスする
- /luancherのエンドポイントで、いい感じにport取得し、system("plackup ./somepath/my_app_1.psgi -p $port -L Shotgun")的なことして、http://project_admin:$port/にリダイレクト
- その後、my_app_1.psgiはいい感じに消滅

さて、具体的なイメージわきましたでしょうか（笑）まぁ意味不明ですね。。。

とりあえず、いい感じにportを取得するのは、テストでもないですが、Test::TCPに任せるか、内部で使っているportとってくるモジュールでも使えばいいとして、問題は、「psgiがいい感じに消滅」です。

## いい感じに消滅　≒　起動にalarmで指定した秒後に消滅

こんなURL「/luancher?project=HogeProject&psgi_path=script/app.psgi」にアクセスすると、以下のようなコントローラーにとおって

        ### MyAdmin::Web::C::Luancher
        use Test::TCP;
        use IO::Interface::Simple;

        sub run {
            my ($class,$c) = @_;

            my $project   = $c->req->param('project');
            my $psgi_path = $c->req->param('psgi_path');


            my $server = Test::TCP->new(
                code => sub {
                    my $port = shift;
                    system("perl ./script/psgi/proclet.pl $project $psgi_path $port");
                },
            );

            $c->redirect("http://".IO::Interface::Simple->new('eth0')->address.":".$server->port());
        }

以下のような「assets/script/psgi/proclet.pl」をたたくって感じですね。ちょっとそれますが、「IO::Interface::Simple」とか何気覚えておきたい便利モジュールですね。あと、なんかもうTest::TCPを直使いしてますが、昔のやっつけコードなので記念に残しておきますw

        ### assets/script/psgi/proclet.pl

        use strict;
        use warnings;
        use Proclet;

        my $project   = $ARGV[0];
        my $psgi_path = $ARGV[1];
        my $port      = $ARGV[2];

        local $SIG{ALRM} = sub { kill 'TERM',$$; };

        alarm $timeout;

        my $proclet = Proclet->new(color => 1);

        $proclet->service(
            tag  => 'psgi',
            code => "cd /home/hirobanex/project/$project && plackup $psgi_path -L Shotgun -p $port",
        );

        $proclet->run;

ポイントは、あえてProcletでラップしたスクリプト内でalermの設定をしてをたたくということですね。

立ち上げるPSGIプロセスは、なんとなく-L Shotgunとか-Rとかつけたいのですが、これって内部でフォークしているので、素のpsgiファイルにalerm仕込んでも、子プロセスが死ぬだけで親プロセスが残っちゃうんですねー。

Procletでラップしてあげることで、親プロセス毎まるっと、alermで死ぬという感じです。

## いい感じに消滅　≒　アクセスがあった時から指定した秒後に消滅
上記の方法でもいいんですが、そうすると30分とか1時間とかある程度まとまった時間をいれておかないと使いずらい気がするのですが、そうすると細かくプロセスを立ち上げまくっていくとportが埋まってしまいそうな恐怖感（あくまで恐怖感）に陥ります。

で、アクセスがあるたびに毎回消滅時間が延長されるというか、リセットされたほうが筋が良い実装な気がするんですね。

ただ、どうやっていいのか思いつかず。。。。苦笑

AnyEventとかでイベント実装すればいい気がするんですが、AnyEventって全く使っていなくてよくわかんす。。。

今回のケースではここまで頑張らなくていい気もしますが、なんかこういうことがいい感じにできると、何か応用できそうな・・・・という気がしたりです。

## 結び
結局、「alermの紹介じゃないっすか」って感じの記事になってしまいましたが、まぁ「select」とか「fork」とかUnixネイティブなオーラの漂う関数ってCとかやっていないとヒヤっとするので、そういうのPerlから入門していく電子書籍とかあるとうれしいなーと思いますので、誰か記事書いてくださいーw
