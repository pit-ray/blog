---
layout: post
title: WindowsをVimのキーバインディングで操作するツールを作った ー win-vind 3.1
tags:
  - win-vind
  - Vim
  - C++
  - UIA
  - hatenablog
---


<div align="center">
<img src="https://github.com/pit-ray/pit-ray.github.io/blob/master/win-vind/imgs/win-vind-icon.png?raw=true" width="256" height="256">
</div>
[:contents]

* はじめに
　Vimは非常に高性能なエディタである一方、使いこなすために多くの時間を要します。Vimを使い始めると、他のエディタでとてつも無くストレスを感じ、文字を打てば文章内に<b>:w</b>などが散らばったりします。これは、Vimのキーバインディングを施すソフトが、多数存在する理由の一つでもあります。

　Windowsにおける統合的なキーバインディングツールと言えば、AutoHotKeyという有名なものがあります。AutoHotKeyは、独自のスクリプトであるAHK Scriptを利用してWindowsのAPIを呼び出し、キーイベントを起こすことでバインドを行います。しかし、AHK Scriptをイチから学習するコストは高く、Vimのコマンドラインやモード管理を再現しきれているものは多くありません。そこで、今回は導入コストやパフォーマンスの面を考え、Vimのキーバインディングを施す専用のツールを作りました。


* win-vind
　このツールは、Windowsに<b>GUI操作用のモードを追加</b>し、Vimを利用しているときのちょっとしたウィンドウ操作をキーボードに手を置いたまま行うことを目的としています。主な機能として、GUI操作、プロセス制御、Vimエミュレート、Vimライクなモード管理、コマンドラインからの利用などがあります。そして、設定用のGUIは英語と日本語、ドキュメントは英語に対応しています。GitHubとホームページは以下を参照してください。
[https://github.com/pit-ray/win-vind:embed:cite]

[https://pit-ray.github.io/win-vind/:embed:cite]

また、次のリンクからダウンロードできます。
[https://pit-ray.github.io/win-vind/downloads/:embed:cite]

現在はWindows10のみをサポートしており、MITライセンスを採用しています。


** GUI操作
　ウィンドウのリサイズや選択、マウスの移動やクリック、仮想デスクトップの切替、UIオブジェクトの検出など、とりわけ多くのファンクションがあります。デフォルトの設定では、マウスを<b>hjkl</b>で操作できたり、VimiumやEasyMotionのような方式でクリックを行えます。
<div align="center"><figure><img src="https://github.com/pit-ray/pit-ray.github.io/raw/master/win-vind/imgs/EasyClickDemo.gif?raw=true" />
<figcaption>EasyClick Demo</figcaption></figure></div>
因みに、この機能はUI Automationをサポートしていれば動作するため、Microsoft Edgeなどのリンクも認識できます。  


** Vimのエミュレーション  
　この機能は、メモ帳やOffice Word、WebのフォームなどでVimのキーバインディングを利用するためのものです。
<figure><div align="center"><img src="https://github.com/pit-ray/pit-ray.github.io/raw/master/win-vind/imgs/edi-mode-demo.gif?raw=true" />
<figcaption>Vim Emulation Demo</figcaption></div> </figure>

** モード管理
　デフォルトでは、GUIを操作するためのモードと、Vimをエミュレートするための2層のモードから構成されています。内部的には、それぞれのモードは独立したものであるため、設定から好きなモードだけを利用したり、キーコンフィグを行うことも可能です。
<div align="center"><figure><img src="https://github.com/pit-ray/pit-ray.github.io/blob/master/win-vind/imgs/GUIandEditor.jpg?raw=true">
<figcaption>GUI Mode and Editor Mode</figcaption></figure></div>

<div align="center"><figure><img src="https://github.com/pit-ray/pit-ray.github.io/raw/master/win-vind/imgs/mode_overview_2.jpg?raw=true" />
<figcaption>Mode Overview</figcaption></figure></div>


** プロセス制御
　プロセス制御と題していますが、大層なものではなく、現在はプロセスの起動しか行えません。例えば、メモ帳を<b>notepad</b>コマンドとして登録しておくと、仮想コマンドラインを用いて、次のように起動できます。
<div align="center"><figure><img src="https://github.com/pit-ray/pit-ray.github.io/raw/master/win-vind/imgs/cmd-demo.gif?raw=true"/>
<figcaption>Start External Application Demo</figcaption></figure></div>

実行可能なファイルであればどんなものでも登録でき、数に限りはありません。


** Vimとの連携  
　win-vindは複数起動ができないようになっており、起動中のwin-vindに対して、二つ目のwin-vindから一意のファンクションIDを渡すことで、そのファンクションをワンショットで呼び出すことができます。
>|sh|
$ ./win-vind.exe --func change_to_normal
||<
したがって、Vimの<b>:!</b>コマンドと組み合わせることで、VimからGUIを操作したり、win-vindへスムーズに切り替えることができます。実装されている機能は、ホームページの[https://pit-ray.github.io/win-vind/cheat_sheet/:title=チートシート]から一覧することができます。


* 最後に
　以上のように、win-vindを利用することで、Vimのキーバインディング機能を一括して導入することができます。しかしながら、AutoHotKeyを利用するメリットも多くあるため、一つの選択肢としていただけると幸いです。

