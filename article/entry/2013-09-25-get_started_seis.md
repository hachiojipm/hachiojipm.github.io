tags: hachioji-tech Perl Perl6
---
# Perl6をかいま見るためにSeisを使ってみる

[自分の感想ブログ](http://mackee.hatenablog.com/entry/2013/09/22/211305)でFuture Perlのトーク聞いて思ったこととか書いてたんですが、[Seis](https://metacpan.org/module/Seis)の話はそういえば全然していなかったなーって思って、じゃあ実際にどうなのかって思って、おもむろに(metacpan)[http://metacpan.org/)でSeisって打ったら上がっていたので勢いでcpanmしてみました。

で、どうなったかというと。

    [- o -] $ cpanm Seis
    --> Working on Seis
    Fetching http://www.cpan.org/authors/id/T/TO/TOKUHIROM/Seis-0.12.tar.gz ... OK
    Configuring Seis-0.12 ... OK
    Needs perl 5.018000, you have 5.016003
    ! Installing the dependencies failed: Installed version (5.016003) of perl is not in range '5.018000'
    ! Bailing out the installation for Seis-0.12

マジかよ、Perl 5.18以上じゃないとダメ！　とはとんがってる。
というわけでおもむろにPerl5.18.1を入れる

    [- o -] $ plenv install 5.18.1
    ...
    [- o -] $ plenv local 5.18.1
    [- o -] $ plenv install-cpanm
    ...

で、cpanm Seis

    [- o -] $ cpanm Seis
    --> Working on Seis
    Fetching http://www.cpan.org/authors/id/T/TO/TOKUHIROM/Seis-0.12.tar.gz ... OK
    Configuring Seis-0.12 ... OK
    Building and testing Seis-0.12 ... FAIL
    ! Installing Seis failed. See /Users/mackee/.cpanm/work/1379948870.12803/build.log for details. Retry with --force to force install it.

oops、build.logを見てみる

    [- o -] $ less /Users/mackee/.cpanm/work/1379948870.12803/build.log
    ...
    t/spec/roast/S32-io/file-tests.t ................................ ok
    Can't locate Path/Tiny.pm in @INC (you may need to install the Path::Tiny module) (@INC contains: CODE(0x7ff96a8386c0) /Users/mackee/.cpanm/work/1379948870.12803/Seis-0.12/blib/lib /Users/mackee/.cpanm/work/1379948870.12803/Seis-0.12/blib/arch /Users/mackee/.cpanm/work/1379948870.12803/Seis-0.12/_build/lib /Users/mackee/.plenv/versions/5.18.1/lib/perl5/site_perl/5.18.1/darwin-2level /Users/mackee/.plenv/versions/5.18.1/lib/perl5/site_perl/5.18.1 /Users/mackee/.plenv/versions/5.18.1/lib/perl5/5.18.1/darwin-2level /Users/mackee/.plenv/versions/5.18.1/lib/perl5/5.18.1 .) at /Users/mackee/.cpanm/work/1379948870.12803/Seis-0.12/blib/lib/Seis/IO/Path.pm line 50.
    t/spec/roast/S32-io/io-path.t ...................................
    Dubious, test returned 2 (wstat 512, 0x200)
    Failed 8/13 subtests
    t/spec/roast/S32-io/path.t ...................................... ok
    ...

見てみると何故かPath::Tinyがないよとテストで言われていてコケているみたい。他の人も一緒なのかなと思って[CPANTesters](http://cpanstesters.org/)見に行ったら他の人も同じ理由でコケているのでとりあえず入れる！

    [- o -] $ cpanm Path::Tiny
    ...

ちゃんと入ったのでSeisもう一度

    [- o -] $ cpanm Seis
    ...
    Successfully installed Seis-0.12
    1 distribution installed

よし、OK！
はい、ここまででcpanmで一発で入らなくてもくじけない編(裏タイトル)は以上です。

じゃあSeis使ってみる。トークではseisコマンドでREPLに入っていたので

    [- o -] $ seis
    seis>

なんか入れたっぽい。[僕のブログで前やった](http://mackee.hatenablog.com/entry/2013/09/22/211305)Reduction operatorを使ってみる。

[Reduction Operators](http://perlcabal.org/syn/S03.html#Reduction_operators)

    seis> [+] 1..100
    $VAR1 = 5050;

やばい、ていうかめちゃくちゃ早い。

    [- o -] $ time perl6 -e 'say [+] 1..100'
    5050
    perl6 -e 'say [+] 1..100'  0.61s user 0.14s system 99% cpu 0.750 total
    [- o -] $ time perl -E '$a += $_ for 1..100; say $a'
    5050
    perl -E '$a += $_ for 1..100; say $a'  0.02s user 0.03s system 51% cpu 0.111 total
    [- o -] $ time seis -e '$a = [+] 1..100; say $a'
    5050
    seis -e '$a = [+] 1..100; say $a'  0.15s user 0.04s system 97% cpu 0.196 total

上からRakudo Star + Parrot、Perl 5.16.3、Seis 0.12 + Perl 5.18.1でございます。
十分実用範囲内の速度な気がします。

Perl6だとこんなかんじで仮引数の型を指定してやることもOK

[Parameter Types](http://perlcabal.org/syn/S02.html#Parameter_types)

    [- o -] $ seis -e 'sub id(Int $n) { return $n }; my $a = id("aaa"); say $a'
    Can't locate object method "throw" via package "Seis::Exception::ArgumentType" (perhaps you forgot to load "Seis::Exception::ArgumentType"?) at -e line 3.

ちゃんとコケてくれてますね。

でもまだPerl6のすべての仕様が実装されているわけではなさそうで、このように返り値の型指定をするとコケたりします。

[Of types](http://perlcabal.org/syn/S02.html#Of_types)

    [- o -] $ seis -e 'sub id(Int $n) returns Int { return $n }; my $a = id(1); say $a'
    Can't parse -e:
    Syntax error! Around:
    sub id(Int $n) r
       at /Users/mackee/.plenv/versions/5.18.1/lib/perl5/site_perl/5.18.1/darwin-2level/Seis/Compiler.pm line 56.

でもXSやC、そしてPerl5で書かれているので既存の開発者でもPerl6を理解するための窓口にはなりやすそうです。Authorの[@tokuhirom](https://twitter.com/tokuhirom/)さんが言われているようにPerl6の機能をPerl5にフィードバックしてPerl5を進化させるのにはもってこいの実験室だと思われます。
