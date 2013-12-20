tags: Perl Advent-Calendar-2013 Compiler-Lexer
pubdate: 2013-12-17 23:59
---
# Compiler::Lexerを使ったモジュール達  
  
Hachioji.pm Advent Calendarの17日目です. 担当はもう何度記事を書いたかわからなくなってきたpapixです.  
今日は, 最近注目を集めている@goccyさん製のPerl用トーカナイザ, [Compiler::Lexer](http://search.cpan.org/~goccy/Compiler-Lexer/)を活用したモジュール達を紹介したいと思います.  
  
## Perl::MinimumVersion::Fast by @tokuhirom  
  
[Compiler::Lexer をつかって Perl::MinimumVersion::Fast をかいてみた](http://blog.64p.org/entry/2013/05/02/185530)  
  
tokuhiromさんが作った, ｢あるPerlのコードが, どのPerlのバージョンを必要とするか｣を確認できるモジュール.  
本家のPerl::MinimumVersionはPPIで実装されており, これをtokuhiromさんがCompiler::Lexer化して完成したものです.  
このモジュールを使えば, Perl 5.8を対象としたライブラリで`//`や`~~`を使っていないかどうかを確認したりできます.  
  
## Test::Synopsis::Expectation by @moznion  
  
[SYNOPSISのコメントを使ってテストするTest::Synopsis::Expectation](http://mackee.hatenablog.com/entry/2013/12/16/234919)  
[Test::Synopsis::Expectationというモジュールをリリースしました](http://moznion.hatenadiary.com/entry/2013/12/17/113458)  
  
[@t_wada](https://twitter.com/t_wada)さんから大絶賛を受けていた, moznionさんのモジュールです.  
  
<blockquote class="twitter-tweet" lang="ja"><p>“SYNOPSIS自体に期待する値を埋め込んでおいて，それをもとにSYNOPSISのコードをテストする事で，SYNOPSISがSyntax的にもLogic的にもValidであることをチェック” すばらしい! jsにもREADME.m… <a href="http://t.co/xaVBM2C534">http://t.co/xaVBM2C534</a></p>&mdash; Takuto Wada (@t_wada) <a href="https://twitter.com/t_wada/statuses/412849220853317632">2013, 12月 17</a></blockquote>  
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>  
  
Synopsisのコメントを使ってテストをするモジュールです.  
｢モジュールの挙動がよくわからなかったら, とりあえずSynopsisを動かしてみる｣という人にとって, 肝心のSynopsisが間違っている(Syntaxが間違ってるとか, 過去の仕様に準拠したSynopsisだったりとか...)と結構困ります.  
モジュール開発者がこのモジュールでテストをしておけば, Synopsisが本当に正しいかどうかを確認することができるので, このようなトラブルを回避できるようになります.  
  
このモジュールをつくる為に, moznionさんはCompiler::Lexerにいくつかのパッチを送っていました.  
ただ, ｢時折Compiler::Lexerが原因? でテストがコケる...｣という症状があるらしく, 現在PPI版の実装に取り組んでいるとのことです.  
将来的には, 後述のTest::LocalFunctionsのように, バックエンドをPPIとCompiler::Lexerから選べるようになるのではないでしょうか.  
  
## Test::LocalFunctions::Fast by @moznion / @papix  
  
[Test::LocalFunctions というモジュールを書いてました](http://moznion.hatenadiary.com/entry/20130412/1365795045)  
[Test::LocalFunctions::Fastというモジュールを作った.](http://blog.papix.net/entry/2013/05/02/223803)  
[Test::LocalFunctions::FastをTest::LocalFunctionsにマージしてもらいました.](http://blog.papix.net/entry/2013/05/27/224100)  
  
Test::LocalFunctionsは, モジュールの中で使われていないローカル関数がないかテストする為のモジュールです.  
元々はmoznionさんがPPIを使って実装していて, それをpapix(私)がCompiler::Lexerを使って高速化し, Test::LocalFunctions::Fastの名前でGitHubに公開しました.  
最終的には, Test::LocalFuctionsにマージしてもらってmoznionさんが管理･リリースしています.  
  
## Test::UsedModules::Fast by @papix  
  
[Test::UsedModules::Fastに心が折れたので, Test::UsedModulesの紹介をします](http://blog.papix.net/entry/2013/07/09/211600)  
  
[Test::UsedModules](https://github.com/moznion/Test-UsedModules)は, こちらもまたまたmoznionさんが開発された, 必要のないモジュールを`use`/`require`していないか確認してくれるモジュールです.  
このモジュールもPPIで実装されていたので, papix(私)がCompiler::Lexerを使って高速化してみました.  
PPIとCompiler::Lexerの差で随分苦労したのですが, 最終的には何とか完成までこぎつけることができました.  
こちらも, 本家(Test::UsedModules)にマージしてもらうつもりだったのですが... いろいろ忙しくて, その辺りの作業を放置して今に至っています.  
  
とりあえず暫定版が[こちら](https://github.com/papix/Test-UsedModules/tree/fast)にありますので, 試してみたい方はどうぞ.  
  
## まとめ  
  
Compiler::Lexerを活用したモジュールを紹介してみました.  
ここ最近, Compiler::Lexerはかなり安定してきた(以前は時折SEGVったりすることがありました...)と印象を持っているので, モジュールを開発する中でPerlのトーカナイザが必要であれば, 使ってみる価値があると思います.  
  
Compiler::Lexerを含むgoccyさんのプロダクトに要注目ですね!  
