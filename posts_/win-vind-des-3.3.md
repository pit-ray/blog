---
layout: post
title: VimiumやEasyMotionのようにウィンドウを操作する ー win-vind 3.3
tags:
  - win-vind
  - Vim
  - C++
  - UIA
  - hatenablog
---

win-vindの新しいバージョンを公開したので、その更新内容を書きます。

前回記事

[https://www.pit-ray.com/entry/win-vind-des-2:embed:cite]

GitHub

[https://github.com/pit-ray/win-vind:embed:cite]

ホームページ

[https://pit-ray.github.io/win-vind/:embed:cite]



[:contents]



# バージョン3.3
## ダウンロード <hr>  

[https://github.com/pit-ray/win-vind/releases/tag/v3.3.0:embed:cite]


## 変更点 <hr>  
### EasyClick <hr>  
<img src="https://github.com/pit-ray/pit-ray.github.io/raw/master/win-vind/imgs/EasyClickDemo.gif?raw=true">  

　EasyClickをマルチスレッドで処理するように変更しました。これまでのEasyClickでは、キーマッチングとヒントの描画をシングルスレッドで行っていたため、ヒンティングがチラついたり、マッチング開始までに多くの時間を必要としていました。3.3からはそれらのオーバーヘッドが解消されました。ただし、C++のstd::threadのマルチスレッドということであり、実際に物理的なマルチスレッドかどうかは環境や状況によります。

### ウィンドウ操作 <hr>  
<figure class="figure-image figure-image-fotolife" title="ウィンドウの選択 (&#x60;&lt;C-w&gt;H&#x60;)">[f:id:pit-ray:20210523165640j:plain]<figcaption>ウィンドウの選択 (&#x60;&lt;C-w&gt;H&#x60;)</figcaption></figure>
　ウィンドウの選択などの距離計測方法を変更しました。ユークリッド距離による計量は変わりませんが、計測点がウィンドウ端からウィンドウの中心に変わりました。これにより、以前のバージョンの不自然なウィンドウ操作が解消されました。また、殆どのウィンドウ操作機能がマルチモニタに対応したため、ストレスフリーにウィンドウ操作ができます。

## 追加点 <hr>  
### Vimのエミュレーション <hr>  
　まず、`2d2w`のように中間にリピート数を入れるような構文に対応しました。Vimと同様に`2d4w`の場合では8回実行されます。加えて、`1000000h`のような莫大なリピートをした際に、`<Esc>`でキャンセルできるようにしました。本家Vimの移動系の繰り返しは、列と行からO(1)で移動できますが、win-vindではGUIを間接的に操作する性質上、単純な繰り返しO(n)となってしまいます。そのため、大きな負荷が生じ、フリーズの原因となっていました。今回は、`<Esc>`によるキャンセル導入により、安心して利用できます。

## 開発面 <hr>  
　3.3は、内部のリファクタリングを大規模に実施し、多くのコードが整理されました。特に、入力解析系のクラスなどはオートマトンベースのものにし、管理しやすい設計に変更しました。例えば、キーボードからの入力解析器であるNTypeLoggerでは次のような状態遷移図となっています。

<figure class="figure-image figure-image-fotolife" title="NTypeLoggerの状態遷移図">[f:id:pit-ray:20210518155025p:plain]<figcaption>NTypeLoggerの状態遷移図</figcaption></figure>

バインディングのマッチングもオートマトンベースにすることで、入力によって候補が絞られていくようにし、不要なマッチングをしないようにしています。状態遷移図は次のようになっています。
<figure class="figure-image figure-image-fotolife" title="LoggerParserの状態遷移図">[f:id:pit-ray:20210518155436p:plain]<figcaption>LoggerParserの状態遷移図</figcaption></figure>

また、CIを整備し、テスト環境を整えました。しかし、現在は主要な機能だけのテストケースしかないため、カバレッジはすこぶる低いですが、今後増やしていきます。ちなみに、ブランチカバレッジを基準としています。


以前、Twitterで「1万行以上のプロジェクトはアーキテクチャを説明するドキュメントを整備せよ」というものを見かけました。その影響もあり、開発用のキュメントを用意しました。今後、時間があるときに追記していきたいと思います。

[https://github.com/pit-ray/win-vind/tree/v3.3.0/devdocs:title]

# 今後 <hr>  
　今のwin-vindは使いにくいです。バインディングを変更する際に、2000行のjsonを変更するか、操作性がクソなGUIをいじらなくてはいけません。そこで、次のバージョンでは`.vimrc`のような**Run Commands**形式の設定方法を採用します。ちなみに、あらかた実装が終わっており、現在のmasterをビルドすれば利用できます。コマンドは`map`, `noremap`, `unmap`, `mapclear`, `command`, `delcommand`, `comclear`, `set`, `source`が対応しており、`.vindrc`ファイルに記述することでVimと同じような記法で設定できます。細かい仕様は、バージョン4.0を公開する際にまた書きますが、`map`コマンドでは低レベルなホットキーをサポートしています。例えば、`F`キーを`G`キーとして認識させることができます。これは、win-vindのキー認識の範囲ではなく、Windowsとしてレジスタ不要のキーマッピングを行えます。理想としては、`<CapsLock>`を`<Ctrl>`としてマッピングできればと思います。

現在の`.vindrc`の仕様は次のようになっています。

[https://github.com/pit-ray/win-vind/blob/master/devdocs/prototypes/vindrc_syntax.md:title]


このような**rc**形式の設定を採用するとGUIは不要な気もしますが、しばらくは補助機能としてサポートすることにします。ただ、win-vindとは完全に分離させ、別のアプリケーションとして提供する予定です。

バージョン4.0は、6月中に公開できればと思いますが、近頃忙しく、もう少し先になるかもしれません。

ちなみにプルリクエストは大歓迎です。手順としては、Discussions等で提案してからプルリクエストを出す形になります。日本語でも大丈夫です。これは、私及び他のContributerと作業が重複しないための対策です。

現在のToDoリストは、Projectsで参照できます。

[https://github.com/pit-ray/win-vind/projects/2:title]


今後もよろしくお願いします。
