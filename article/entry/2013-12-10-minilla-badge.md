tags: Travel Advent-Calendar-2013 Minilla Perl
pubdate: 2013-12-10 23:59
---
# MinillaでTravis CIなどのバッヂをREADME.mdに貼り付ける方法

Hachioji.pm Advent Calendarもついに10日目. そんな2桁記念日に颯爽と登場したるはpapixです. 今日も宜しくお願い致します.  
今日は時間がないので, 若干あっさり目で行きたいと思います.  
  
...さて. 最近, [Travis CI](https://travis-ci.org/)や[Coveralls](https://coveralls.io/)を使っている人が増えてきているような気がします.  
これらのサービスは, GitHubのリポジトリを結びつけておくと, 自動的にテストを回してくれたり, テストのカバー率を表示したりしてくれるのでとても便利です.  
更に, GitHubのリポジトリ内のREADME.mdにこれらのバッヂを貼り付けておけば, README.mdの内容はリポジトリのトップページ(?)に表示されるのでこれらの状況をとても確認しやすくなります.  
  
ただ, Minillaでモジュールを管理している場合, README.mdは`minil build`や`minil test`をすると自動的に再度生成されるので, その都度バッヂを貼る必要があります.  
かといって, 元のPODにバッヂを貼るのはよろしくないです.  
  
...というのを悩んでいたところ, 実は`minil.toml`で設定すれば, README.mdに対して自動的にバッヂを生成できる, ということを昨日知りました.  
この設定をしておけば, `minil.toml`にこういう感じで書けば, `minil build`や`minil test`などで再度生成されるREADME.mdに, 自動的にバッヂが貼り付けられます! 便利!!!  
  
```  
badges = ['travis', 'coveralls']  
```  
  
badgesを含む`minil.toml`の設定等々については, [Minillaのドキュメント](https://github.com/tokuhirom/Minilla#configuration)に書かれていますので一度読んでみるといいかもしれません.  
  
<blockquote class="twitter-tweet" lang="ja"><p>PerlモジュールのREADME.mdにTravis CIのバッジ貼りたいけど、README.mdがminilaが生成するPODベースの文章を使ってるのでPOD内に含める必要がありそうだけど、POD内だとCPAN上に表示するときダメそう??</p>&mdash; ıɐɯɐu ıɥsoʇɐs (@ainame) <a href="https://twitter.com/ainame/statuses/410325755181031425">2013, 12月 10</a></blockquote>  
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>  
  
今日, こんなやりとりもあったので, 俺得備忘録を兼ねて, Hachioji.pm Advent Calendarでメモさせていただきたいと思います.  
  
# 次回予告  
  
明日は再び@hirobanexさんが登場の予定です! お楽しみに.  
