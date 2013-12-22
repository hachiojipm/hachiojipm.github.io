tags: Perl Advent-Calendar-2013
pubdate: 2013-12-19 23:59
---
# PlackのMiddlewareを条件付きでロードする

papixです. ここまでこの｢Hachioji.pm 日めくり__テックトーク__｣を, まったくテックに関係ない話題で埋め尽くしてきました(cf. 旅行情報, 飯情報...)が, 今日は久々にテックな話題をお送りします.  
  
PlackのMiddlewareを, ｢ある条件を満たした場合のみ適用したい｣という場合があると思います.  
例えば, Basic認証をかける[Plack::Middleware::Auth::Basic](https://metacpan.org/pod/Plack::Middleware::Auth::Basic)を, 一部のパスの場合のみ有効にして, Basic認証をかけたい! ...という場合とかです.  
  
Plack::Middleware::Auth::Basicを`enable`で有効にした場合, 全てのパスに対してBasic認証をかけてしまいます.  
このような場合, `enable`はなく`enable_if`を使うことで実現することができます.  
  
例えば, パスの先頭が`/login`である場合のみ, Basic認証をかけたいという場合は,   
  
    enable_if { $_[0]->{PATH_INFO} =~ m!^/login! }
        'Plack::Middleware::Auth::Basic',
        authenticator => sub {
            my ($username, $password, $env) = @_;
            return 'hoge' eq $username && 'fuga' eq $password;
        };
  
のように書けばOKです.  
詳しくはこちらの記事をどうぞ: [Plack::Middlewareの使い所](http://perl-users.jp/articles/advent-calendar/2011/casual/8)
