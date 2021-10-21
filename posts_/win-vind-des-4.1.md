---
layout: post
title: EasyClickの高速化 - win-vind4.1
tags:
  - win-vind
  - Vim
  - C++
  - UIA
  - hatenablog
---

win-vind 4.1の更新内容です。

[:contents]

前回記事 (4.0)
[https://www.pit-ray.com/entry/win-vind-des-4:embed:cite]

GitHub
[https://github.com/pit-ray/win-vind:embed:cite]

ホームページ
[https://pit-ray.github.io/win-vind/:embed:cite]


### ウィンドウの整列の例外リスト <hr>
　Twitterでご指摘がありましたが、rainmeterなどの常駐ウィジェットはウィンドウハンドルを所持しているため、`<C-w>=`によるウィンドウ整列でレイアウトが崩れてしまうそうです。そこで、整列対象から外すための`arrangewin_ignore`オプションを追加しました。

``` vim
set arrangewin_ignore = notepad
set arrangewin_ignore = notepad, rainmeter
```

　このようにプロセスの実行ファイル名(拡張子なし)を指定します。複数指定する場合にはカンマで区切ります。現在は<b>各プロセスが異なる実行ファイル名である</b>と仮定しているため、同じ実行ファイル名のプロセスがあると、同様に例外リストに追加されます。

### テキスト領域の自動選択 <hr>
　Editor Normalモードに移行するためにEscキーを押すと、アプリケーションによってはテキストフィールドのフォーカスが外れることがあるようです。そこで、Editor Normalモードに移行する際に、ウィンドウ内のテキストフィールドを全て検出し、マウスカーソルから最も近いものにフォーカスする`autofocus_textarea`オプションを追加しました。

```vim
set autofocus_textarea " 有効化
```


### UIスキャンの高速化 <hr>
　デフォルトのEasyClickや`autofocus_textarea`は、コマンドを実行するたびにスキャンを行うため、処理時間がネックとなっていました。そこで、別スレッドで非同期的にUIをスキャンし、そのキャッシュを利用する`uiacachebuild`というオプションを追加しました。このオプションはいくつかのパラメータがあります。
```vim
set uiacachebuild " 有効化
set uiacachebuild_lifetime = 1500 " 1.5秒
set uiacachebuild_staybegin = 500 " 0.5秒
set uiacachebuild_stayend = 2000 " 2秒
```

　`uiacachebuild`は、EasyClickなどを使っていない時でも、定期的なスキャンを行います。頻繁なスキャンを行うと負荷が増大するため、`uiacachebuild_lifetime`という処理時間とキャッシュの信頼度のトレードオフを決定するパラメータがあります。値には、キャッシュの有効期間をミリ秒を指定します。この値が小さいと精度と負荷が上昇し、大きいと精度と負荷が減少します。

　さらに、`uiacachebuild_staybegin`と`uiacachebuild_stayend`というパラメータがあり、これらは有効期間よりも優先されます。先ほど述べたように、`uiacachebuild`は定期的なキャッシュ生成を行うため、その回数を必要最低限にすることが望ましいと言えます。このパラメータは、<b>マウスカーソルが同じ位置にいるとき</b>を<b>操作していない</b>とみなし、停止してからの指定した期間内にスキャンします。どちらもミリ秒の値を取ります。


[https://pit-ray.github.io/win-vind/imgs/uiacachebuild_stay.png:image=https://pit-ray.github.io/win-vind/imgs/uiacachebuild_stay.png]


### デフォルトのバインディングを変更 (※注意) <hr>
　Residentモードへの移行を`<Esc-Down>`に、ResidentモードからInsertモードへの移行を`<Esc-Up>`に変更しました。以前の`<C-i>`は、MS Officeの斜体にするショートカットと重複しているため、このような変更となりました。


[https://pit-ray.github.io/win-vind/imgs/mode_overview.png:image=https://pit-ray.github.io/win-vind/imgs/mode_overview.png]


方向キーは、図中の位置関係に対応しています。


### 最後に <hr>
　Windows 11には対応する予定です。
