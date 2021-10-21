---
layout: post
title: WindowsでVimライクにウィンドウを操作 ー win-vind 3.2
tags:
  - win-vind
  - Vim
  - C++
  - UIA
  - hatenablog
---

以前公開したソフトをアップデートをしました。その更新内容と補足内容です。

前の記事
[https://www.pit-ray.com/entry/win-vind-des:embed:cite]

GitHub
[https://github.com/pit-ray/win-vind:embed:cite]

ホームぺージ
[https://pit-ray.github.io/win-vind/:embed:cite]


目次

[:contents]


## バージョン3.2 <hr> 
今回のアップデートではウィンドウの操作に関連した機能を強化しました。ここでは網羅的に書きます。

### 変更点 <hr> 
　まず、GUIオブジェクトをVimiumのようにヒンティングする<b>EasyClick</b>という機能の無駄な処理を省いたため、スキャン時間が3分の1程度まで短縮され、実用的なものになりました。さらに、今現在のmasterブランチのビルドでは、EasyClickのマルチスレッド稼働に対応しており、より高速で安定しています。  
　また、恥ずかしながら、以前のバージョンまで文字コードを雑に扱っていた節があり、GUIからパス名を設定すると例外が発生し、強制終了するというバグがありました。17035インサイダービルド以降のWindows10からは、ASCII版の関数においても、UTF-8でエンコーディングされたファイル名を処理してくれるようですが、現在のwin-vindでは外部のやり取りにUTF-16を利用し、内部でUTF-8を利用することにしました。ネイティブエンコーディングがUTF-8にならない限り、この実装で行きたいと思います。どちらにせよ、今回のバージョンにより日本語環境でも安定した動作が期待できます。


### 追加点 <hr> 

#### ウィンドウの整列 <hr>
　この機能は、現在開いている（最小化されていない）ウィンドウを全て検出し、メモリの使用率順でタイル状に自動で並べます。Vimの`<C-w>=`に相当する機能で、複数のディスプレイがある環境でも全て整列させます。現在は、メモリの使用率で順に並べていますが、使用頻度にするか検討中です。

<img src="https://github.com/pit-ray/pit-ray.github.io/blob/master/win-vind/imgs/arrange_window_min.gif?raw=true" >

3.2の更新内容をredditのVimコミュニティにポストしたところ、「お前はWindows用のi3wmを作った！」など絶賛されていましたが、ウィンドウマネージャという観点からはMicrosoftとそのコミュニティがオープンソースで開発しているPowerToysの[FancyZones](https://docs.microsoft.com/en-us/windows/powertoys/fancyzones)などのほうが美しく優れていると思います。他にもフリーウェアの[Plumb](http://palatialsoftware.com/plumb/)や[LGUG2Z / yatta](https://github.com/LGUG2Z/yatta)や[tzbob / python-winodws-tiler](https://github.com/Tzbob/python-windows-tiler)などがあり、それぞれ非常に豊富な機能を持っています。このように類似したツールが多くあるうえで、再実装するのは馬鹿らしいように思えますが、Vimのシーケンシャルなコマンド入力やコマンドラインを利用したバイディングに乗っ取っていることに多くの意味があります。

ちなみに、win-vindはワンショットの呼び出しに対応しており、win-vindのバインディング機能を利用せず、AHKのバインディングの下でユーティリティだけを使うことができます。

```ahk
#a::Run, win-vind -f arrange_windows
```

<b>--func/-f</b>オプションに渡す文字列は、[Cheat Sheet](https://pit-ray.github.io/win-vind/cheat_sheet/)のIDです。


#### ウィンドウの回転<hr>
　この機能は、選択しているウィンドウが表示されているモニタ内で、整列済みのウィンドウの回転を行います。Vimの`<C-w>r`や`<C-w>R`に相当する機能です。ただ、release版では`<C-w>=`で整列し、ウィンドウの順番を生成してからでないと利用できませんが、masterビルドではこの限りではありません。因みに、rが反時計周り、Rが時計回りとしています。

<img src="https://github.com/pit-ray/pit-ray.github.io/blob/master/win-vind/imgs/rotate_min.gif?raw=true" >  


#### ウィドウの交換<hr>
　この機能は、選択しているウィンドウから最も近いウィンドウと位置とサイズを交換します。Vimの`<C-w>x`に相当する機能です。距離の指標には、ウィンドウの中心からのL2ノルムで計算しています。

<figure class="figure-image figure-image-fotolife" title="Exchange overview">[f:id:pit-ray:20210227053639j:plain]<figcaption>Exchange overview</figcaption></figure>


#### ウィンドウのリサイズ<hr>
　ウィンドウのリサイズをVimライクに行う機能です。私は、[simeji / winresizer](https://github.com/simeji/winresizer)を愛用しているため、標準の機能はあまり利用しないのですが、一連の実装として追加しました。Vimのコマンドでは、Verticalが列に対する用語であるため、動作とあべこべなコマンドとなってしまいました。

<b>高さ</b>  
`:resize <num>`で高さをピクセル単位で指定することができます。`<num>`はwin-vind独自のキーワードで、数字が入力され数字以外が入力されるまでの文字列を表しています。例えば、`:resize 500`で高さが500ピクセルとなります。`:resize 512abcd123`のような入力の場合、512ピクセルと解釈されます。また、`<C-w>+`または`:resize +<num>`で高さを増やし、`<C-w>-`または`:resize -<num>`で高さを減らすことができます。ちなみに、`resize`は`res`と短縮できます。

<b>幅</b>  
`:vertical resize <num>`で幅をピクセル単位で指定できます。同様に、`<C-w><gt>`または`:vertical resize +<num>`で幅を増やし、`<C-w><lt>`または`:vertical resize -<num>`で幅を減らすことができます。この点、横方向のリサイズにも関わらず、verticalとなっています。ちなみに、`vertical resize`は`vert res`と短縮できます。


#### ウィンドウのスナップ<hr>
　この機能は、選択しているウィンドウをある方向の画面半分にします。Vimの`<C-w>H`, `<C-w>J`, `<C-w>K`, `<C-w>L`に相当する機能です。Windows標準のショートカットである`<win-left>`や`<win-right>`とは違い、ウィンドウ自体のリサイズと移動を伴います。


#### ウィンドウの選択<hr>
　この機能は、Vimの`<C-w>h`, `<C-w>j`, `<C-w>k`, `<C-w>l`に対応する機能です。選択したい方向の端の中心から最も近いウィンドウを選択します。距離には、L2ノルムを用いています。この機能は、複数のモニタ間でも動作します。

<figure class="figure-image figure-image-fotolife" title="Select a left window Overview">[f:id:pit-ray:20210227064354j:plain]<figcaption>Select a left window Overview</figcaption></figure>


#### ウィンドウの複製<hr>
　現在選択しているウィンドウを半分にし、空いた部分に同じプログラムを起動します。Vimの`:split`や`:vsplit`に相当する機能です。エクスプローラやウェブサイトを分割して開きたいときに便利です。多重起動を許さないなどの理由によって新しいウィンドウを作れなかった場合には、ウィンドウのリサイズだけ行います。

[https://github.com/pit-ray/pit-ray.github.io/blob/master/win-vind/imgs/horizontal_split_min.gif?raw=true:image=https://github.com/pit-ray/pit-ray.github.io/blob/master/win-vind/imgs/horizontal_split_min.gif?raw=true]
  
## このソフトウェアについて <hr> 
　このツールは、gVimを利用しているときに他のソフトを触りたいときや、Wordの使用を余儀なくされているときにVimと同じように操作したいという気持ちから作りました(Vimのエミュレート機能についてはオモチャみたいなものですが...)。しかし、READMEにVimのエミュレート機能を強くアピールしていたせいか、メモ帳をVimに変える面白ソフトにとして紹介されたことがありました。

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Windows users can have a lot of fun turning Notepad.exe into Vim, among other things: <a href="https://t.co/SrR4de4OzV">https://t.co/SrR4de4OzV</a> <a href="https://t.co/5qqIk38nyS">pic.twitter.com/5qqIk38nyS</a></p>&mdash; Vim Links (@VimLinks) <a href="https://twitter.com/VimLinks/status/1345989539551186945?ref_src=twsrc%5Etfw">January 4, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

　概念的に言えば、Vimというアプリケーションよりも一つ上のレイヤから、Vimバインディングを行うような機能に相当します。Vimプラグインの[skywind3000 / asyncrun.vim](https://github.com/skywind3000/asyncrun.vim)を利用すれば、VimにGUI操作用のモードを追加するツールとしても使えます。

```Vim
Plug 'skywind3000/asyncrun.vim'

command! GUINormal :AsyncRun win-vind -f change_to_normal
```
これにより、`:GUINormal`でGUI操作用モードに移行できます。

　ただ、WindowsはLinuxのようにウィンドウマネージャを切り替える手段を提供していないため、Windows APIを呼び出すだけの疑似的なものでしかありません。また、Vimのようなシーケンシャルなコマンド入力は、[rcmdnk / vim_ahk](https://github.com/rcmdnk/vim_ahk)のようにAutoHotKeyでも再現することができます。したがって、AHKスクリプトのRunなどを利用すれば、単体の機能を有するツールを呼び出し、同様にVimバインディングのユーティリティツールを再現できます。(例えば、EasyClickの機能には[zsims / hunt-and-peck](https://github.com/zsims/hunt-and-peck)などが利用できます)

　AHKは高度なカスタマイズを行えるソフトウェアである一方、win-vindは事前定義された関数を呼ぶため、処理の内容まで変更できません。しかし、プログラミングを主として、バインディングをその支援とするユーザにとっては、行き過ぎた自由度は導入のコストを増やします。特に、上記のようなヒンティング機能であったり、Vimバインディング導入のためには、さまざまなリソースに依存することになります。対して
win-vindは、Vimライクなバインディング機能とユーティリティを一括で導入でき、カスタマイズの自由度も必要最低限です。

## 今後 <hr> 
　現在のwin-vindは、初期値をカスタマイズするようにして利用しますが、これでは導入コストが高いように思います。それに、網羅的な設定が一つのファイルに収まっているため、カスタマイズが難解であることは否めません。そこで、ゲーミングデバイスようなプリセット機能であったり、.vimrcライクな記法を導入しようかと検討中です。とはいっても自由度を高めすぎると、学習コストが高まったり、最終的にAHKに収束してしまうような気もします。また、UWP系のモダンなUIに憧れがあるので、GUIを印新するかもしれません。WinUI3の動向や今流行りのFlutter2を触ってみて決めたいと思います。
