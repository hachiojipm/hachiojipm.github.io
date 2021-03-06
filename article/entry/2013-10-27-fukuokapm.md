tags: Perl FukuokaPM
---
# Fukuoka.pmに乱入してきた!

皆さんこんにちは, 提督業と修論業で慌ただしい日々を送っている｢地域PMエバンジェリスト(自称)｣ことpapixです.  
最近は駆逐艦の良さがわかってきた気がしています. ...あ, そういう意味ではないですヨ?

...さて今日は, 紆余曲折を経て10月26日(土)に開催された｢Fukuoka.pm #24｣に行ってきたので, その感想レポートをお送りしたいと思います. 

## スポンサー紹介

今回, Fukuoka.pmへ参加するにあたり, 往路の交通費を以下の方々にスポンサードして頂きました.  
この場をお借りして御礼申し上げます. 本当にありがとうございました!

- [@yusukebe](https://twitter.com/yusukebe)さん
- [@uzulla](https://twitter.com/uzulla)さん
- [@xtetsuji](https://twitter.com/xtetsuji)さん

## Fukuoka.pm #24

Fukuoka.pmの雰囲気等々についてはゆーすけべーさんの感想ブログ記事, ｢[福岡のPerl Workshop「Fukuoka.pm#24」に行ってきました！](http://yusuke.be/post/65193797413)｣がよくまとまっているので, こちらを読めばいいのではないでしょうか.  

ゆーすけべーさんもブログ記事で書いていますが, 参加人数が少ない分(ATNDの参加申し込み人数で14人), 発表者と参加者の距離が近いので, 参加者の反応を感じやすかったですし, 質疑応答やツッコミといったコミュニケーションが生まれやすい雰囲気が出来ていたので, とても良かったです.

## トーク

今回のスピーカーは, JPAの講師派遣制度で来ていたゆーすけべーさん, 東京から参戦のcharsbarさん, Fukuoka.pmのヒラタさん([@debility](https://twitter.com/debility)), そして私の4人でした.

ゆーすけべーさんのトークは, 最近チャレンジしているボケての構造化について.
ざっくりまとめると, ｢フロントエンド(テンプレートエンジンとかを使って, 実際にブラウザに表示する部分を担う)｣と, ｢APIフレームワーク(フロントエンドのAPIリクエストに応じて, DBにアクセスしたりして, 適切な(フロントエンドで表示したい)データを返す)｣の2つに分離して, 分業化するというアイデアを実践している... という話でした.  

社内プロダクトならともかく, ボケてのような社の垣根を越えた少人数のチームで開発している場合, このように(開発者の/Webサービスの)役割を明確に区切る, というアプローチは有効そうな気がします.  
今後, この方針がどのように進展していくかが, とても気になる発表でした!

charsbarさんは, CPANTSやCPANモジュールのKwalitee, MinilやMinillaを使ったモジュール開発等々についてのお話でした.  
何故か自分がKwaliteeランキングで10位に入っていましたが, あれは｢よしKwaliteeランキングで上位に入ってやろう!｣と思って頑張った訳ではなくて, モジュール開発の為に漬かっているMinillaがよしなにやってくれただけだと思います.
そういう意味を含めて, Minillaはとても便利なツールなので, これからモジュールを作る! という方はMinillaを採用することをおすすめしますし, 既存モジュールも, 積極的にMinillaに切り替えていく価値があると思っています.

ヒラタさんの発表は, 社内の余興で某クイズ番組をPHPやNodeを駆使して再現した, というお話.
弊社も割とそういう空気がありますが, ああいう余興(?)に全力を尽くせるノリっていいなあ... と感じました. 
というか, あのシステムはすごい出来が良いので, 某局とコラボするとか, ロゴ等々のマズい部分をなんとかしてOSSのような形で公開したりとか, そういう展開があってもいいですよね. 今後どうなるか期待です.

## 自分のトーク

で, 自分ですが｢GPGPUとXSとわたし｣というタイトルで, PerlにおけるGPGPUの現状と, XSについての話をして来ました.  
資料はこちらです(後でアップします...)

前半は割と学術っぽいネタで, 後半はXSという人を選ぶ中級者向けなネタだったので, ビギナーな方には｢あー, こういう世界があるんだなー｣というのを感じて頂けるように努力... したつもりなんですが如何でしたでしょうか.  
人を選ぶネタなので, お昼寝兼内職タイムになっちゃうのでは? と懸念していたのですが, 予想以上に興味津々な感じで聞いて頂けましたし, 質問もいくつか飛んできたので非常に嬉しかったです.

あと, PerCUDAの成果として完成した｢PerCUDA::Backend::GPU｣ですが, これ普通にPerlからPTXをデバイスAPIを使って呼び出すツールとして使える気がするので, ｢CUDA::DeviceAPI｣みたいな名前で独立したモジュールとして公開してもいいんじゃないかなー, と思ったりしました.  
人柱よろしくです! (To GPGPUで研究をしている八王子在住のPerl Monger)

## 福岡最高!

常日頃から言っていますが, 福岡は自分にとって最高の都市の1つです.  
まず都市の規模が程よくコンパクトで, 移動がとても楽. 東京で言えば新宿と渋谷と銀座が徒歩圏内にある, みたいなイメージでしょうか.
あと空港から中心部までの距離が異常に近いのも良いですね. 博多駅から福岡空港まで, 地下鉄で2駅ですし.

あと食いしん坊バンザイな自分やゆーすけべーさんからすると, 屋台だったり居酒屋だったり, 飲食店のレベルが高いのも高印象でした. 要するに, 適当な店に行っても大抵メシがウマイ.
そして焼酎がとても安いですね! 黒霧島とか, 多分大阪や東京の100〜200円引きくらいの値段で飲めます.

そしてあれです. Fukuoka.pmとかに行くと, 地元の方が自信を持っておすすめするお店とかに行く事になるので, まあマズいはずがないですよね, ええ.  

![モツ鍋](<: '/static/image/nabe.jpg' | uri_for :>)

こちらのモツ鍋は, Fukuoka.pmの懇親会(1次回)で行ったモツ鍋屋のモツ.  
ちなみにこの後, 2次会でラーソーメンを食べ, 3次会で屋台に行って鶏の炭火焼や餃子を食べたりしてました.  
最終的な解散時間は午前1時...! そして地元の方はずっとビールなり焼酎なりを飲み続けていました. お酒に強すぎィ!

![明太子御飯](<: '/static/image/mentaiko.jpg' | uri_for :>)

こちらの明太子御飯は, Fukuoka.pmの翌日(つまり今日), ｢ここ超よさそうじゃね!?｣とゆーすけべーさんが見つけたお店で食べたもの.  
多分, これ東京で食べたら間違いなく1.5〜2倍くらいの値段がするんじゃないかなあ...

## さいごに

Fukuoka.pm最高! 呼ばれなくてもまた来ますし, 呼ばれたら絶対に行きます!!!  
ちなみに次のFukuoka.pmには[｢ツールチェインギャング見習い｣として有名なmoznion氏も参加](http://yancha.hachiojipm.org/quot?id=168623,168617)するみたいなので, とても楽しみですね!

Fukuoka.pm #24に参加された皆様, そして交通費をスポンサードして頂いたゆーすけべーさん, uzullaさん, てつじさん, 本当に本当に, ありがとうございました!  

<center>Fukuoka.pm最高〜〜〜〜〜〜〜〜〜〜〜ッ!!!!!</center>


