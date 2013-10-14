tags: Perl LLVM goccy
---
# Compiler::CodeGenerator::LLVMで遊ぼう!

今日は, [@goccy54](https://twitter.com/goccy54)さんが開発された[Compiler::CodeGenerator::LLVM](https://github.com/goccy/p5-Compiler-CodeGenerator-LLVM)を導入して, ひと通り動かすまでのアレコレをご紹介したいと思います.

## LLVMのインストール

Compiler::CodeGenerator::LLVMはLLVM 3.3に依存しているので, まず最初にLLVM 3.3を導入します.  
ついでにLLVMを基盤とするコンパイラのClangと, ランタイムライブラリのCompiler RTも入れてしまいましょう.

まず, [こちらのページ](http://www.llvm.org/releases/download.html#3.3)から, `Clang source code`, `LLVM source code`, `Compiler RT source code`をクリックして, 必要なソースコードをダウンロードします.  

    $ wget http://llvm.org/releases/3.3/cfe-3.3.src.tar.gz
    $ wget http://llvm.org/releases/3.3/compiler-rt-3.3.src.tar.gz
    $ wget http://llvm.org/releases/3.3/llvm-3.3.src.tar.gz

それぞれ, `cfe-3.3.src.tar.gz`, `llvm-3.3.src.tar.gz`, `compiler-rt-3.3.src.tar.gz`というファイルがダウンロードできるので, さっさと解凍してしまいます.

    $ tar -xf cfe-3.3.src.tar.gz
    $ tar -xf llvm-3.3.src.tar.gz
    $ tar -xf compiler-rt-3.3.src.tar.gz

解凍してできた`llvm-3.3.src`の中の適切な位置に, `cfe-3.3.src`と`compiler-rt-3.3.src`を配置していきます.

    $ mv cfe-3.3.src llvm-3.3.src/tools/clang
    $ mv compiler-rt-3.3.src llvm-3.3.src/projects/compiler-rt

続いて, `llvm-3.3.src`ディレクトリに移動して, `configure`で設定を行います.  

    $ cd llvm-3.3.src
    $ ./configure

問題なく`configure`が終われば, `make`, `make check`, `make install`でビルドして, インストールまで済ませます.  
ちなみに`configure`を実行する際, `--prefix`オプションで任意の場所にLLVMやClangをインストールすることができますが, 手元の環境では何故かうまく動かなかったのでデフォルトでインストールした方が良さそうです.  
｢sudoで入れるの嫌だなあ...｣という方はVagrantとかVirtualBoxとか, そういうのを作ってCompiler::CodeGenerator::LLVM用のVMを建てましょう!

    $ make

ちなみに, `make`コマンドは`make -jN`とすることで, Nコアを使って処理を進めることができます.  
ビルドには時間がかかるので, `make -j2`とか`make -j4`とかで高速化しましょう.  

    $ make check
    $ sudo make install

`make check`は任意ですが, とりあえずやっておきましょう. 手元の環境では100件ほどFailしますが, おそらく大丈夫... だと思います.  
あとは`sudo make install`を実行すれば, LLVMとClangがインストールされます.

    $ which llvm-config
    /home/username/local/llvm/bin/llvm-config
    $ llvm-config --version
    3.3

といった感じで, LLVMとClangが正常にインストールできていることが確認できれば, 準備完了です.

## Compiler::CodeGenerator::LLVMのインストール

それではCompiler::CodeGenerator::LLVMをインストールしましょう.  

まず依存モジュールの準備です.  
Compiler::CodeGenerator::LLVMは, こちらもまたgoccy氏が開発した[Compiler::Lexer](https://github.com/goccy/p5-Compiler-Lexer)と[Compiler::Parser](https://github.com/goccy/p5-Compiler-Parser)に依存しているので, これを入れる必要があります.

    $ cpanm Compiler::Lexer Compiler::Parser

どちらもCPANizeされているので, 素直に`cpanm`でインストールしてもいいですし, GitHubからcloneしてインストールしてもよいでしょう.  
ちなみにどちらのモジュールも, Minillaで開発されているので`minil install`でインストールできます.

依存モジュールの準備が終われば, いよいよCompiler::CodeGenerator::LLVMのインストールです.  
こちらのモジュールは, まだCPANizeされていないので,

    $ git clone https://github.com/goccy/p5-Compiler-CodeGenerator-LLVM.git

でcloneしてきてから,

    $ cd p5-Compiler-CodeGenerator-LLVM
    $ perl Makefile.PL
    $ make
    $ make test
    $ make install

の手順でインストールしていきます.

## 早速, 試す!

それでは早速, Compiler::CodeGenerator::LLVMで遊んでみましょう.  
まずはSynopsisのコードを動かしてみます.

    use Compiler::Lexer;
    use Compiler::Parser;
    use Compiler::CodeGenerator::LLVM;
    
    my $filename = $ARGV[0];
    open(my $fh, "<", $filename) or die("$filename is not found.");
    my $script = do { local $/; <$fh> };
    my $lexer = Compiler::Lexer->new($filename);
    my $tokens = $lexer->tokenize($script);
    my $parser = Compiler::Parser->new();
    my $ast = $parser->parse($tokens);
    my $generator = Compiler::CodeGenerator::LLVM->new();    
    my $llvm_ir = $generator->generate($ast); # generate LLVM-IR
    $generator->debug_run($ast); # execute LLVM-IR with JIT

Synopsisのコードは, 標準入力に与えられたファイル名に書かれているコードからLLVM IRを生成して, それをJITで実行するというスゴイやつです.  
テストコードを`synopsis.pl`みたいな名前で保存して, 

    use strict;
    use warnings;
    
    my @arg = (0, 1, 2, 3, 4);
    foreach my $i (@arg) {
        say $i;
    }

こんな感じのファイルを`sample.pl`で保存して, `$ perl synopsis.pl sample.pl`てな感じで実行すると...

    $ perl synopsis.pl sample.pl
    0
    1
    2
    3
    4

Wow! 実行できましたね.  
個人的には, 内部で生成されたLLVM IRをダンプできれば, いろいろと遊べそうな感じがしますが, 現状そのようなメソッドはないっぽいので, 今後に期待したいと思います.  
(時間があれば実装とかチャレンジしてみたい所ですが... XSだしなあ...)

## まとめ

YAPC::Asiaにおいて, 注目の的の1つでもあったPerlMotionを支えるgoccy wareの, Compiler::CodeGenerator::LLVMの導入方法をまとめてみました.  
このモジュールが完成すれば, PerlでiPhoneアプリが開発できるようになるだけでなく, PerlのコードをLLVM IR経由で各種バイナリに変換したり, C言語に変換したり, CUDAに変換したり... と, いろいろな可能性が期待できるようになります.

今後も, バク報告だけでなく, 出来ればコードをコミットしていく... という方法で, Compiler::CodeGenerator::LLVMやCompiler::Lexer, Compiler::Parserといったgoccy wareをサポートして行けたらいいなあ, と思っています.  
これからのCompiler::CodeGenerator::LLVM, 要チェックですね!

なお, 今回この記事を書くにあたり, 作者の[@goccy54](https://twitter.com/goccy54)さんに大変, ものっすごっく, アドバイスとご協力を頂きました.  
この場をお借りして御礼申し上げます. 本当にありがとうございました!
