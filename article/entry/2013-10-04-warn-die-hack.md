tags: Perl

---

Perlの警告と例外を両方握りつぶすtypesterハック
=================================

先日[独立](http://unknownplace.org/archives/start-new-company.html)された[@typester](https://twitter.com/typester)さんのモジュール[GitDDL](http://search.cpan.org/~typester/GitDDL-0.02/lib/GitDDL.pm)のコードを見ていたらよくわかんところがありまして、はちぴー御用達のChatサービス[yancha](http://yancha.hachiojipm.org/)でいろいろと教えてもらったので、そのご報告です。ちょっとマニアックな話題なので、はてぶとかでどしどし間違いだったりの突っ込みよろしくですー！


hackishな問題のコード
-----
        my $version = try { ## ここのコードがhackishでよくわからん！！！
            open my $fh, '>', \my $stderr;
            local *STDERR = $fh;
            $self->database_version;
            close $fh;
        };

さてこれが問題のコードです。ここだけパッと見ると、

- 謎にtryが始まって
- スカラーリファレンスのファイルハンドル開いて
- STDERRにぶち込んで

なんじゃこりゃ？と思ったわけです。

全体をみるとのコード
-----
一部だけに注目しているとわけがわからんことはしばしばで、yanchaで聞いたらよくよく全体を見たら少しわかります。

    sub deploy {
        my ($self) = @_;

        my $version = try { ## ここのコードがhackishでよくわからん！！！
            open my $fh, '>', \my $stderr;
            local *STDERR = $fh;
            $self->database_version;
            close $fh;
        };

        if ($version) {
            croak "database already deployed, use upgrade_database instead";
        }

        croak sprintf 'invalid version_table: %s', $self->version_table
            unless $self->version_table =~ /^[a-zA-Z_]+$/;

        $self->_do_sql($self->_slurp(File::Spec->catfile($self->work_tree, $self->ddl_file)));

        $self->_do_sql(<<"__SQL__");
    CREATE TABLE @{[ $self->version_table ]} (
        version VARCHAR(40) NOT NULL
    );
    __SQL__

        $self->_dbh->do(
            "INSERT INTO @{[ $self->version_table ]} (version) VALUES (?)", {}, $self->ddl_version
        ) or croak $self->_dbh->errstr;
    }


    sub database_version {
        my ($self) = @_;

        croak sprintf 'invalid version_table: %s', $self->version_table
            unless $self->version_table =~ /^[a-zA-Z_]+$/;

        my ($version) =
            $self->_dbh->selectrow_array('SELECT version FROM ' . $self->version_table);

        if (defined $version) {
            return $version;
        }
        else {
            croak "Failed to get database version, please deploy first";
        }
    }


問題を抽象化したコード
-----
一部だけに注目しているとわけがわからんことはしばしばで、yanchaで聞いたらよくよく全体を見たらよくわかりました。deployメソッドの中では、database_versionのリターンが偽が期待されているんだけど、その場合「$self->_dbh->selectrow_array('SELECT version FROM ' . $self->version_table);」でDBIの設定によって警告が出るので、それを/dev/nullに入れる的なことをしたいということなんですね。要するに、以下のコードがその状況を抽象化したもののようです。

    #!/usr/bin/env perl
    use strict;
    use warnings;
    use utf8;
    binmode STDOUT, ":utf8";
    use Try::Tiny;
    use Test::More;

    sub warn_and_die {
        warn "警告だよ！！";
        die "例外だよ！！";
    }

    my $res = try {
        open my $fh, '>', \my $stderr;
        local *STDERR = $fh;
        warn_and_die();
        close $fh;
    };

    ok !$res;

    done_testing;

考察と疑問
-----
まぁそれでもよくわからないことはいくつかあるのでそれについてつらつらと。

### スカラーリファレンスを対象にファイルハンドル開けるんだ

        open my $fh, '>', \my $stderr;

ここですが、ほとんど見慣れないですよね。


### STDERRにファイルハンドルを突っ込む

    local *STDERR = $fh;

これですね。よく

    local $SIG{TERM} = sub { $cnt = 0 };

とかやってforkマネージャーみたいなやつで、シグナルハンドラをいじるのはみかけますが、STDERRをオーバーライドするのはぼくは初めてお目にかかりました。
    
    local *STDERR = undef;

こんな感じに$fhのファイルハンドルわざわざ開かずにundef突っ込んでもよいのでは？という話もありましたがSTDERRはファイルハンドルなのでファイルハンドルをつっこまなければならないようです。

### close $fhのリターンバリューは何か
tryブロックのリターンバリュー（GitDDLのコードでいう$version、先ほどの簡易コードでいう$resに入るデータ）は、close $fhの返り値になるわけですが（なるよね？）、そんなもの普段意識しないのでちょっと疑問に思いますね。数年後にはtypesterさん並みのハッカーになっているだろう[まこぴー](https://twitter.com/mackee_w)さんが、颯爽と[closeのドキュメント](http://perldoc.jp/func/close)付で、

> 操作が成功し、PerlIO 層からエラーが報告されなかった場合に真を返します。 引数が省略された場合、現在選択されているファイルハンドルをクローズします。

と教えてくれました。

しかし、ファイルハンドルのopen,closeはよく or dieとかつけて例外処理するようなもので、close $fhて失敗したら、どうするのって思ったりもしますが、開いているファイルハンドルがスカラーリファレンスなので失敗することはないという風に解釈しました（がどうなんでしょうか？認識あっているでしょうか？）

### そいういえばlocal $SIG{__WARN__} = sub {};
よく考えれば、$SIG{__WARN__}をつかえばいい気もしますが、まぁ息を吸うように書いた時の気分ってやつでしょうか。むしろ、$SIG{__WARN__}で代替できない問題があれば是非教えて頂きたいところです。。。

CPANモジュールのコードを読んで勉強しよう
--------
必ずしも参考になるコードばかりではないですし、見たコードを実践すべきかどうかは別として、なんかわからないコード見つければいろいろドキュメントみたりあれこれ議論したり調べたりすると、大変Perlや広い意味でのシステム基礎知識全般への理解が深まり大変うれしいですね！

[typeさんのコード、hackishだから読んでたら新しい発見いろいろある](http://yancha.hachiojipm.org/quot?id=141199)とまこぴーのモヒカン発言に激しく同意しつつ、いくつかのhackishなモジュールで大変お世話になっているtypesterさんの独立を激励する記事でございました！
