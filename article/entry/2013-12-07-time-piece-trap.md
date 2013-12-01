tags: Perl
---
# Time::PieceのParseで%zを使うのは無理ゲー

「タイムゾーンマタギはおいておいて、そうでないケースにおいて、Date::Timeはオワコンで、Time::Pieceが主流」的なノリになってからだいぶ長いですが、最近やっとTime::Pieceを使い始めたのですが、メールのヘッダーのDateのところをぴっぱろうとしたらはまったんで、そのあたりをまとめておきます。あまり、直接的なタイトルの日本語記事がなかったので。

## ダメのコード
さて、問題となる具体的なコードは以下です。

    #!/usr/bin/env perl
    use strict;
    use warnings;
    use utf8;
    use Time::Piece;

    {
        my $datetime = "Thu, 28 Nov 2013 18:32:29 +0900 (JST)";
        my $t = localtime(Time::Piece->strptime($datetime,'%a, %d %h %Y %H:%M:%S %z (%Z)'));
        warn $t->datetime; # 2013-11-28T09:32:29
    }

    {
        my $datetime = "Thu, 28 Nov 2013 18:32:29";
        my $t = localtime(Time::Piece->strptime($datetime,'%a, %d %h %Y %H:%M:%S'));
        warn $t->datetime; # 2013-11-28T18:32:29
    }

[日付の書式指定](http://www2u.biglobe.ne.jp/MAS/perl/waza/strftime.html)にあるように、日付をいろいろパースするときの記号はいろいろあるわけですが、「%z」という「+0900」という「タイムゾーンのGMTへのオフセット時間」ってやつが、Time::Pieceでうまくいかないのはご存知でしょうか。

下の方ではしっかりパースできるのに、上の「+0900 (JST)」のコードのほうが、見事に9時間ズレてしまっています。

## 既存文献と対応方法
重要な問題だからこそでしょうが[Time::Piece::strftime and time zone issue](http://blog.64p.org/entry/2013/03/01/112510)という英語の記事でtokuhiromさんが、問題と対応方法を書いているんですが、なんかややっこしいので、どうしたものかと思ったら、sonmguさんが、[HTTP::Date](http://search.cpan.org/~gaas/HTTP-Date-6.02/lib/HTTP/Date.pm)を使うのがいいかもねって教えてもらったので、ぼくはそうしました。

    use HTTP::Date;

    my $datetime = "Thu, 28 Nov 2013 18:32:29 +0900 (JST)";
    my $t = localtime(HTTP::Date::str2time(HTTP::Date::parse_date($datetime)));
    warn $t->datetime; # 2013-11-28T18:32:29 


まぁ便利でわかりやすい。Date::Time使うのとCPUとかメモリのコストがどれほど違うのかよくわかりませんが、わかりやすさ間違いないでしょう。毎度、sonmgu++という感じでいつか恩返ししなきゃいけないところです。

## 終わりに
日付周りのモジュールは何が何だったかすっかり忘れてことが多いので、しっかりメモっておきたい今日この頃です。

こういう軽いTipsをこちこちやっていきたいところです。年末はたくさんアドベントカレンダーあるんで、ちょっと胃もたれがちですからねー。

