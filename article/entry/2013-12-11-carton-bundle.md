tags: Perl
pubdate: 2013-12-11 23:59
---

# carton bundle && carton install --cachedの使いどころ

carton bundleとか「*.tar.gzでモジュールをインストールできて何が嬉しいの？」と疑問に思っていたんですが、この度、carton bundleの使いどころが分かったので、経緯をばしたためておきたいと思います。

## ゆるふわにCarton使うと、installとexecしか使わない
なんか、いろいろ調べるといろいろ書いてあったりするけど（[Carton & cpanm―Perlモジュール管理最新事情（3）| Perl Hackers Hub](http://gihyo.jp/dev/serial/01/perl-hackers-hub/002103?page=2),[1.3 Cartonによるアプリケーションの実行環境の構築](http://blog.livedoor.jp/lestrrat-practical_modern_perl/archives/25002846.html)）
結局のところ、開発環境だろうと本番だろうと以下のようにすれば、Cartonを使ったモジュール管理ができて楽ちんよって感じが普通だと思います。

        cpanm Carton
        #プロジェクトの依存モジュールcpanfileに書く
        carton install #モジュールのインストール
        carton exec -- plackup ./app.psgi

なので、cartonっていろいろコマンドあるっぽいけど、結局

- carton install
- carton exec

の二つを使っておしまいで、他のコマンドの使いどころが意味不明ーって長らく思っていました、すみませんw

## GitHubにあったりのモジュールをDarkPANで入れたプロジェクトで危機に遭遇
最近、[OrePAN2::Server](https://metacpan.org/pod/OrePAN2::Server)っていうOrePAN2のサーバー版をCPANにアップすることが出来まして、意気揚々とこれを使って、社内モジュールをがしがしmyDarkPANにアップしていまして、Carton使う時なんかも

        PERL_CARTON_MIRROR=http://oreprepanan/ carton install

って感じで、ゴリゴリ入れたんですが、この度初めて社外のVPSにデプロイして運用する機会が生まれまして、「そういや、VPS的な本番環境だと、このオレオレなOrePANサーバーにアクセスできなくね？」って思ったのです。

なんと！！！これは大変な事件だ！！！もう、後数時間後から稼働させるってみんなに言っちゃったのに！！！

で、まず解決策として思いついたのが、cartonが作るlocalごとrsyncで飛ばせばいいやーという案でした。

開発環境と本番環境が同じならこれで済んだとおもんですが、しかし、実はぼくのローカルPCのMySQLのバージョンが5.1で、本番は「最近はせめて5.5くらいにしておきたいよねー」とか頑張っちゃって5.5にしていてたので、実際に、「carton exec -- plackup ./app.psgi」とかして動かしたら、localディレクトリに入っているDBD::mysqlだかなんだかか、libmysqlclient16がないよとかエラーがでてきて、ツンだ感じになりました。。。

「やっぱり本番環境でcarton install掛けなきゃいけないのかーーーつらいなー」と思いつつ、そういやlocalのディレクトリの中見ていたら、cacheって中にモジュールの「*.tar.gz」ファイルがたくさんあって、とりあえず、もうcartonとかわすれて、これをシェルスクリプトで、cpanmでグルグルまわして、とりあえず、ユーザーのPerlディレクトリにつっこんじゃおーって思いつつ、ふと、「そもそもcartonになんかそういう機能あるんじゃね？」って思い、ikasm_aさんに記事を見直したらやっとcarton bundleというコマンドにたどり着きました。

## carton bundle && carton install --cachedを使ったモジュールのインストールとデプロイ
とりあえず、リアリティを出すために実際にディレクトリ構成を用いて説明します。

こんな感じのAmon2なディレクトリ構成になっているとして、localにはガッシリ、オレオレなCPANにアップされていないモジュールが入っています。

        Build.PL
        Changes
        META.json
        builder/
        config/
        cpanfile
        cpanfile.snapshot
        daemontools/
        db/
        deploy.process
        lib/
        local/ #carton installで入れたモジュールが入っている
        log/
        minil.toml
        script/
        sql/
        static/
        t/
        tmpl/
        xt/

で、まず、開発環境でcarton bundleとうちます。

        Build.PL
        Changes
        META.json
        builder/
        config/
        cpanfile
        cpanfile.snapshot
        daemontools/
        db/
        deploy.process
        lib/
        local/ #carton installで入れたモジュールが入っている
        log/
        minil.toml
        script/
        sql/
        static/
        t/
        tmpl/
        vendor/ #carton bundleコマンドでできたディレクトリ
        xt/

すると、vendorというディレクトリができます。そんで、rsyncを打ちます。

        rsync -avz  --delete --exclude="*.swo" --exclude="*.swp" --exclude='log/*.log.*' --exclude='daemontools/*/log/main' --exclude='daemontools/*/log/supervise' --exclude='daemontools/*/supervise' --exclude=local --exclude=META.yml --exclude=MYMETA.json --exclude=MYMETA.yml --exclude=.carton /home/hirobanex/project/MyApp myapp-server:/home/production/project/

これで、意味の分からんファイルは同期されれずに、プロジェクトのディレクトリがvendorも含め同期されます。

そんで、以下のように作業します。

        ssh myapp-server #本番サーバーに移動して
        cd /home/production/project/MyApp #プロジェクトTOPに移動
        carton install --deployment --cached ##ここに注目

「caton install」のオプションとして、「 --cached」っていうのを使うとvendorディレクト内の*.tar.gz圧縮ファイルからモジュールをインストールしてくれるのです。とりあえず、高速かつOrePAN的なモジュールもわざわざOrePANサーバーにアクセスする必要ないので気が楽です。

ちなみに、「--deployment」を付けると、「cpanfile.snapshot」が更新されないので、本番でキモいことにならずに済みます。

蛇足ですが、実際は、以下のようにスクリプトにまとめておけば、小規模環境のプチデプロイスクリプトになり便利。別にサーバー何個もあるわけじゃないから、とりあえず、こんなシェルスクリプトで十分。

        #/bin/sh
        carton bundle
        rsync -avz  --delete .....(上述のため省略)
        ssh myapp-server "cd /home/production/project/MyApp && /home/production/.plenv/versions/5.x.x/bin/carton install --deployment --cached"
        ssh myapp-server "sudo svc -h /etc/service/myapp-web-server"
        ssh myapp-server "sudo svc -h /etc/service/myapp-admin-server"
        ssh myapp-server "sudo svc -h /etc/service/myapp-worker"

## carton install --cachedvendorのキャッシュから入れるからマジ楽
「OrePANとかで運用しているモジュールもないし、おいらには関係のない話だー」と思う人もいるかもですが、新しいプロジェクトに入った時とか、ローカルにcarton installするだけでもマジ時間かかって何していたか忘れるくらいだと思うので、「vendor」ディレクトリもgitでコミットしてしまうか、あるいは、別のサブモジュールとかに切り出すかとかして管理して、新しく入った人も開発環境で、carton install --cachedで依存モジュール落とせばマジで好感度アップ間違いなしだと思う次第です。突然誰かがいじりたいって言っても、vendorディレクトリを適当にどっかにあげて、rsyncでひっぱってもらえば、さくっと入れられるし、なんかtar.gzファイル便利w

## まとめ～carton bundleとcarton install --cachedのつかいどころ～

- OrePAN運用モジュールのアクセスの拡張を気にせず、積極的に使える
- 2番目の人から依存モジュールのインストールが楽ちんに

ということで、have nice carton life!!
