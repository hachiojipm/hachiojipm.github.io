tags: Mac
pubdate: 2013-12-24 23:59
---
こんばんわ、uzullaです。正直きつい、死にたい、そんな気分のクリスマス・イヴです。


# 先日…

先日、某所にてHappyHacking Keyboard Proが超激安価格で発売されたのを覚えていらっしゃいますでしょうか。
あの場においては私は買いそびれたのですが、  運良くツテで入手することができました！！！これでおいらもハッカーだ！！


# しかし…

手にいれたものはいいものの、正直あまり自覚していなかったのですが、私はかなりの変態キーボード使いだったのです。


# 自己紹介
私は基本的にUS配列派なのですが（日本語配列のキーボードをつかうと＠すら入力が困難）、なんと物理的にはJISキーボードしか受け付けないという実に変態な人間なのです！！！！

皆がかっこいいUS配列のKBDを使う中、私はJISキーのMacをつかっているのです！！！


おかしいな…割と賛同者を他に見た事が無い…


# どうしてそうなった、または具体的にJISをUS配列で使う事のメリット

* 「JIS配列はボタンが多い」
* 「USキーは（渡され｜売って）ないことが（ある｜った）」

コレで大体の説明が出来るのでは無いかなと思っているのですが、どうでしょうか…無理か。


簡単にいえば、昔Windowsをつかっていたころ、Thinkpadを使っていたのですが、USキーボードモデルは高かったので買えなかった。

なので、101キーボードドライバをいれることで106を101としてつかっていたのです。
今でこそMacはUSキーボード選び放題ではありますが、昔は割とそういう文化が無かった。USキーボードは秋葉原で買ってくる物だった。


さらに、窓使いの憂鬱というソフトが当時はありまして、ソレを使う事でキーマップいじりたいほうだいモードになり、兎に角好き勝手にキーボードに機能を追加する事ができました、楽しい。たとえばCtrlとJKLIあたりでカーソルとかマジクール。
JIS->USの切り替えも一発でできるので((ドライバを入れ換えればいいが、アレはアレで微妙な事になることも多かった))最高でしたね


# 時は流れ…

いまわたしは立派なマカーになりました！！！


MacとWindowsの大きな違いは何でしょうか？そうです、CmdとCtrlの違いです！！！


私は当然のようにCapsをCtrlに置換していたのですが、まあMacはCmdですからね、最初からの出直しでした。
とはいえしばらくすれば慣れる物で、大して問題にはならなかった。

ただ、ちょっとMacのCmdって左右によりすぎじゃね？どう考えても親指が折れ曲がりすぎじゃね？とおもって英数キーをCmdに変換しておりました。英数、かなキーなんてつかわねーや、トグルはCtrl+;だ！！そんな感じでした。


しかし、これが今の悲劇につながるとはその当時おもってもみませんでした…


# で、どういうこっちゃ

英数をCmdにするとめっちゃ指が楽なので超お勧めなんですけど、逆に、本来のCmdをまったく押せなくなるんですよね、指が鍛えられていない。結果として、HHKBに切り換えた途端、Cmd系の操作を意識しないとできなくなりました！！つらい！！！

二日くらい頑張ってみたんですけど、やっぱり究極的に微妙にタイプ間違えるんですよね…忙しいのもあいまって、諦めて一度前のキーボードにモドしたくらい無理でした…


# リトライしたい

しかし、HHKBにそろそろリトライしたい！！

しかしどうしてもやっぱりCmdとSpaceを押し間違える…
もうSpaceの左半分がCmdならいいのに…


…あれ、SpaceをCmdにしたらいいのでは？？？（天才現る）


# 本題、Keyremap4MacBookで自由なキーマップをする

Keyremap4Macbookは多分多くの人が入れているキーマップ変更ソフトウェアだと思います。
こいつの持ってる設定は超豊富なんですけど、微妙にかゆいところに手が届かない事があるのは皆様ご存じの通りです。

バージョンがドンドンがあって、Real Viモードとかそれもはやキーリマップソフトといえないのでは…という次元にまで進化したこいつ、実際ちゃんとしらべたら独自キーマップができたんですねーーーしらなかった！！！

Keyremap4Macbookを起動して、Misc & Uninstallタブから「Open Private.xml」を押す事で自前設定を追加することができます！！

そして、以下が今回ワイが追加した設定や！！！

```
<?xml version="1.0"?>
<root>
    <item>
      <name>Space to Cmmand_L</name>
      <appendix>(+ When you type Space only, send Space)</appendix>
      <identifier>remap.space2commandL_space</identifier>
      <autogen>__KeyOverlaidModifier__ KeyCode::SPACE, KeyCode::COMMAND_L, KeyCode::SPACE</autogen>
    </item>
    <item>
      <name>Space to Command_L</name>
      <appendix>(+ When you type Space only, send Space) + [KeyRepeat]</appendix>
      <identifier>remap.space2commandL_space_keyrepeat</identifier>
      <autogen>__KeyOverlaidModifierWithRepeat__ KeyCode::SPACE, KeyCode::COMMAND_L, KeyCode::SPACE</autogen>
    </item>
</root>
```

これはスペースキーを単体でおしたらスペースが出るが、何かのキーと一緒におしたらCmdになる！
すごい！Cmdが最高にでっかくなっちゃった！！便利！！

 これで「スペースをCmdに差し替えたい（なぜなら普段JISをUS配列でつかっているから、Cmdをタイプするのが大変）」という頭がわりとおかしめなひとも大満足です。Keyremap4Macbook最高！！便利！！


（一応かいておきますけど、XMLに記述したらChangeKeyのチェックボックス群にこの設定が追加されるだけなので、オンオフは自由自在です）


# 参考情報

書き方は以下のURLにありますが、

[https://pqrs.org/macosx/keyremap4macbook/xml.html.ja](https://pqrs.org/macosx/keyremap4macbook/xml.html.ja)

結局既存の設定ファイルをみたほうが勉強になります

[https://github.com/tekezo/KeyRemap4MacBook/blob/master/src/core/server/Resources/include/checkbox/standards/option.xml](https://github.com/tekezo/KeyRemap4MacBook/blob/master/src/core/server/Resources/include/checkbox/standards/option.xml)


非常に、本当に非常に色々なサンプルがあるので、殆どの場合コピペ＆修正でどうにかなりますので、PHPerの私でも安心でした！！


# みんなも変態キーマップでがんばろう！！

イエス！変態キーマップ！！


# 欠点
でもね、なんかこれ(KeyOverlaid)やるとちょっとキーボードの判定が遅くなった気がするの、多分気のせいじゃ無いの。
日本語入力の時に結構顕著で、スペースで変換するとちょっと遅れてやってくる。微妙にそれが気にくわない感じはあるの。
ウーーンあとちょっとなんだけどなあ…。

Keyrepeatの設定から、KeyOverlaid ModifierのWait周りをいろいろいじると何となくそれっぽくはなるんだけど、うーーん…。

まあ、しばらく頑張ってどうしてもどうしても駄目ならあきらめるかなーー…HHKB…ああ、あこがれのハッカーーー。  
（まあもっともっともっとがんばって定義書けば解決するのかも知れない…）



といった感じで微妙な残尿感をのこしつつ、明日は感動のフィナーレの予定です！！！
