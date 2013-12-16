tags: Perl
pubdate: 2013-12-14 23:59
---
# Perlにおいて配列を逆順にするいくつかの方法

よくあるアルゴリズム問題ですね。たぶんいろんなところで既出のような気がしますが、昔某氏に教えてもらったことを思い出しながら書きます。

## 問題

    my @array = (1..2014);

という配列があったとして、この並びを逆にするにはどんな方法があるでしょうか？？

## reverseに決まっとるがじゃ！！
基本的に、マニアでなければPerlの基本的な関数であるreverseを使うと思います。

        my @reversed = reverse @array;

## プログラム始めました的な人とか

### popとの合わせ技

    my @reversed;
    for  (1..scalar @array) {
        push @reversed,pop @array;
    }

### unshiftとの合わせ技

    my @reversed;
    for  my $var (@array) {
        unshift (@reversed,$var);
    }

と、まぁいろいろあるわけですね。


## なんか速そうな技
で、だいたい最後に出てくるのがこれですね。

    my @reversed = @array;
    for  my $i (1..int(scalar @array/2)) {
        $reversed[-$i]  = $array[$i-1];
        $reversed[$i-1] = $array[-$i];
    }

なんか、もっときれいに掛けた気がするけど忘れちゃった。。。

## で、ベンチは？
どれが早いかは、是非、ベンチをとってみてくださいという感じで、古き良きネタのどぶさらいははおしまいー。

