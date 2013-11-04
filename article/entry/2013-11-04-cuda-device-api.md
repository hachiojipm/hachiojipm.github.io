tags: Perl CUDA GPGPU
---
# CUDA::DeviceAPIでPerlからお手軽GPGPUライフ!

おはようございます. 三連休で生活リズムが崩れに崩れ, 朝6時に寝て昼15時に起きるという生活をしているpapixです.  
今日は, 艦これのイベントのE-5で詰んでしまったので, 修論の一環で作成した拙作モジュール, [CUDA::DeviceAPI](https://github.com/papix/CUDA-DeviceAPI)についていろいろ書いて行きたいと思います.

## GPGPUとは?

[GPGPU](http://ja.wikipedia.org/wiki/GPGPU)は, GPU(グラフィックボード)の演算資源を画像処理以外に応用する技術のことです.  
複数の計算を, GPUが持つ多数のコア(64個とか128個とか...)で並列に計算することで, シミュレーションや暗号解読などを高速に処理することができます.

ここ最近, いわゆるLL言語でのGPGPUという分野が多少活発になりつつあります. 有名どころだと, PythonのPyCUDAとかはその一例です.  
学術研究レベルでも, 東大が開発した[Ikra](http://www.graco.c.u-tokyo.ac.jp/ppp/index.php?Projects%2FRubyGPU)や, 同志社大学が開発した[ParaRuby](http://mikilab.doshisha.ac.jp/research/graduate_thesis/2011/nakamura/rnakamura.pdf)など, いくつかのフレームワークが研究･実装されています.

Perlではどうなのか? というと, CPANにはあがっていませんが, [CUDA::Minimal](https://github.com/run4flat/perl-CUDA-Minimal)というモジュールが存在します.  
これは, CUDAのランタイムAPIを利用して, Inline::CのようにCUDAのコード(CUDA C言語)を書いて, GPUで実行することができる... というものです.

今回紹介する｢CUDA::DeviceAPI｣は, CUDAが持つもう1つのAPI, デバイスAPIを使って, PerlからGPGPUを利用するモジュールです.

## CUDA::DeviceAPIのインストール

CUDA::DeviceAPIはCUDA 5.0を要求します. 最新版のCUDA 5.5だと動かない... んじゃないでしょうか. 試してませんが.  
CUDAのインストールは基本的にめんどくさいのですが, Ubuntu 13.04以降だと, 

    $ sudo apt-get install nvidia-common nvidia-cuda-dev nvidia-cuda-toolkit

でインストールできるようになっています. いやはや, 便利な時代になったものです...  
なお, 言うまでもありませんがNVIDIA製のGPUが搭載されていることが前提条件ですので今すぐ日本橋か秋葉原にでも行って買ってきましょう. GPUを使えばPerCUDAの開発者になれる上, Minecraftとかもサクサク動くようになるので一石二鳥です.

CUDAのインストールが終われば, 

    $ git clone https://github.com/papix/CUDA-DeviceAPI.git
    $ cd CUDA-DeviceAPI
    $ minil install

とかすれば, インストールが出来るはずです.

## CUDA::DeviceAPIを使ってみる

さて, 早速CUDA::DeviceAPIを使って, PerlからCUDAを使ってみましょう.
まず, CUDAで実行するコードを用意します.

    extern "C" __global__ void kernel(float *a, float *b, float *c, int n)
    {
        int col = blockIdx.x * blockDim.x + threadIdx.x;
        int row = blockIdx.y * blockDim.y + threadIdx.y;
    
        c[col + row * n] = 0;
        if(col < n && row < n) {
            float tmp = 0.0f;
            for (int i = 0; i < n; i++) {
                tmp = tmp + a[row * n + i] * a[col + n * i];
            }
            c[col + row * n] = tmp;
        }
    }

この`kernel`関数は, n行n列のfloat型配列を3つ受け取り, 1つ目のfloat型配列と2つ目のfloat型配列の乗算を計算して, 3つ目のfloat型配列に格納する, というコードです. 多分.  
今回は, このコードを`sample.cu`という名前で保存します.

このコードを, CUDA::DeviceAPIから実行できるように, CUDAのアセンブリ表現であるPTXに変換します.  
CUDAのインストールが終わっていれば, `nvcc`というCUDA用コンパイラが使えるようになっているはずなので,

    $ nvcc -ptx sample.cu

すると, `sample.ptx`というファイルが生成されるはずです.

最後に, 生成したPTXを呼び出すPerlのコードを書きます.

    use strict;
    use warnings;
    
    use CUDA::DeviceAPI;
    use CUDA::DeviceAPI::Array;
    use Data::Dumper;
    
    my $LENGTH = 4;
    my $block_size = 32;
    
    my $matrix = [
        [ 1, 2, 3, 4 ],
        [ 1, 2, 3, 4 ],
        [ 1, 2, 3, 4 ],
        [ 1, 2, 3, 4 ],
    ];
    
    my $path = 'sample.ptx';
    my $host_data = CUDA::DeviceAPI::Array->new($matrix);
    
    my $ctx = CUDA::DeviceAPI->new();
    
    my $dev_ptr_1 = $ctx->malloc_from($host_data => 'f');
    my $dev_ptr_2 = $ctx->malloc_from($host_data => 'f');
    my $dev_ptr_3 = $ctx->malloc($host_data->size() => 'f');
    
    my $size = $ctx->ceil($LENGTH / $block_size);
    
    $ctx->run($path, 'kernel_sum', [
        $dev_ptr_1 => 'p',
        $dev_ptr_2 => 'p',
        $dev_ptr_3 => 'p',
        $LENGTH    => 'i',
    ], [
        $size, $size, 1, $block_size, $block_size, 1
    ]
    );
    
    my $result = $ctx->return($dev_ptr_3);
    
    print Dumper $result;

今回は, 4行4列の配列の乗算を計算してみました.  

計算する行列は`my $matrix`に入っていて, これを`CUDA::DeviceAPI::Array->new($matrix)`とすることで, CUDA::DeviceAPIでよしなに扱える形式にしています.

`malloc_from`は`CUDA::DeviceAPI::Array`型のオブジェクトを第1引数に受け取って, 第2引数の型(今回は`f`なのでfloat型)でGPUのメモリに領域を確保し, 値を配置するメソッドです.  
`malloc`は, 第1引数に`{ x => 3, y => 2, z => 1 }`のようなハッシュリファレンスを受け取ると, 第2引数の型に応じて, 第1引数で指定したサイズの配列を格納する為に必要な領域を, GPUのメモリに確保してくれます.

あとは, ここまで生成した値を利用してPTXを呼び出し, 実行するだけです.  
実行は`run`メソッドから行います.  
このメソッドの第1引数は実行するPTXへのパス, 第2引数は実行する関数の名前を指定します.  

第3引数は関数に渡す引数を配列リファレンスで指定しています.
配列リファレンスの2n番目には, GPUで実行する関数のn番目の引数となるポインタ(`malloc_from`や`malloc`が返すもの)ないし数値を指定します.   
配列リファレンスの2n + 1番目には, 2n番目で指定した引数の構造を指定します. 配列のポインタであれば`p`, 整数であれば`i`, float型の数値であれば`f`といった形です.

第4引数はCUDAのグリッドサイズやブロックサイズを指定する部分になります.

最後に, GPUで計算した結果は, `return`で取得する事ができます.  
引数に`malloc_from`や`malloc`で生成したポインタを指定すれば, そのポインタが指す値をGPUから取得し, 適切な配列リファレンス構造に変換した上で返します.

## まとめ, そしてCUDA::DeviceAPIからPerCUDAへ...

...というわけで, Perlから, CUDAのデバイスAPIを利用してGPGPUを実現するCUDA::DeviceAPIというモジュールを作りました.

CUDA::DeviceAPIやCUDA::Minimalによって, PerlからCUDAを利用する環境というのは整いつつありますが, まだまだ使い勝手が万全, という訳ではありません.  
そこで現在, このモジュールを利用して, PerCUDAというPerl向けGPGPUフレームワークを作ろうとしています.  

PerCUDAでは, CUDA::DeviceAPIの分かりにくい部分を極力隠蔽するのはもちろん, Perlのコードから自動的にPTXを生成し, ｢Perlのコーディングのみ｣でGPGPUを実現できるようにする予定です.  

これによって｢GPUがない場合はPerlのみで実行｣, ｢GPUがある場合はPerl + GPUで実行｣という2つの手法を簡単に使い分けることができるようになるので, ｢GPUがないPC(例えばMBAやノートPC)でコードを書いて小規模の計算で検証｣, ｢細かな調整と大規模な計算はGPUがあるPCで高速に実行｣といった開発手法ができるようになる, ...はずです.

近日中にPerCUDAの方もコードを公開したいと思っておりますので, CUDA::DeviceAPIともども, patchやissue, バグ報告等々をお待ちしております!
