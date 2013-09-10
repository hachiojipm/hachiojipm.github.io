## Rijiのインストール

    % cpanm Riji

## 記事の作成

artcle/entry 以下に適当に、markdownファイルを拡張子.mdで配置して編集してコミットして下さい。

面倒臭かったら、`% riji new-entry hoge` とかやって下さい。

riji publishする前に一旦mdファイルはgit commitしておいてください。

## 構築と配信

    % riji publish && git commit -am "publish" && git push

とか。
