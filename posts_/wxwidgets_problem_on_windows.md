---
layout: post
title: Windows環境でwxWidgetsのインクルードやリンクが失敗する問題 （3.0.x, 3.1.x, Visual Studio 2019, MinGW）
tags:
  - C++
  - MinGW
  - hatenablog
---


今回はVisual Studio 2019でwxWidgets 3.0.5のビルドが失敗する問題と、MinGW-w64でwxWidgets 3.1.4のビルドが失敗する問題の対策です。

[:contents]


* wxWidgets 3.0.5 + Visual Studio 2019<hr>  

** 問題
wx/wxcrt.hの中のwxStroll()に関してテンプレートのエラーが吐かれます。wx/wxcrt.hにて<b>wxNEEDS_DECL_BEFORE_TEMPLATE</b>が定義されてないのが問題です。

** 対策
次のようなヘッダファイルを作り、エラーが生じているソースの先頭でインクルードすればよいです。
>|cpp|
#if defined(_MSC_VER) && _MSC_VER >= 1500
#ifndef wxNEEDS_DECL_BEFORE_TEMPLATE
#define wxNEEDS_DECL_BEFORE_TEMPLATE
#endif
#endif
||<

<br>  

* wxWidgets 3.1.4 + MinGW-w64 <hr>  
** 問題
libwxmsw31u_core.aとのリンクが失敗し、大量のundefined referenceが出力されます。wxWidgetsがMinGWの<b>libuxtheme.a</b>と<b>liboleacc.a</b>とのリンクに失敗しているのが原因です。

** 対策
MinGWをインストールしたディレクトリから<b>libuxtheme.a</b>と<b>liboleacc.a</b>をコピーし、<b>wxWidgets/lib/gcc_lib</b>の中に置きます。MinGW-w64のデフォルトのディレクトリ構成ならば、<b>mingw64/x86_64-w64-mingw32/lib/</b>の中にあります。バッチファイルでコピーする場合には、次のようなスクリプトを書けばよいです。

>|dosbatch|
for /f "usebackq" %%A in (`where.exe mingw32-make`) do set MINGW_MAKE_PATH=%%A

cd libs
@if not exist tmp (
    mkdir tmp
)
powershell cp "%MINGW_MAKE_PATH:~0,-21%\\*-mingw*\\lib\\liboleacc.a" "tmp\\liboleacc.a"
powershell cp "%MINGW_MAKE_PATH:~0,-21%\\*-mingw*\\lib\\libuxtheme.a" "tmp\\libuxtheme.a"

copy "tmp\\liboleacc.a" "wxWidgets\\lib\\gcc_lib\\liboleacc.a"
copy "tmp\\libuxtheme.a" "wxWidgets\\lib\\gcc_lib\\libuxtheme.a"

cd ..
||<

** 追記
wxWidgetsの<a href="https://github.com/wxWidgets/wxWidgets/releases/tag/v3.1.5">v3.1.5</a>ではこの問題が修正されているようです。


<br>  

* 素直にVisual Studioを使おう <hr>  
CMakeのwxWidgetsのモジュールでは、<b>wxWidgets/lib/vc_lib</b>と<b>wxWidgets/lib/vc_x64_lib</b>と<b>wxWidgets/lib/gcc_lib</b>しか検索を行わないため、MinGWを利用したときに32 bitと64 bitを自前で使い分けないといけません。また、今現在のMinGWでビルドを行うと、古いスタイルのリソースを参照するため、wxWidgetsとVisual StudioのwxWidgetsでは見た目が異なります。

><figure class="figure-image figure-image-fotolife" title="wxWidgets (MinGW)">[f:id:pit-ray:20210117103246p:plain]<figcaption>wxWidgets (MinGW)</figcaption></figure><
><figure class="figure-image figure-image-fotolife" title="wxWidgets (Visual Studio 2019)">[f:id:pit-ray:20210117103625p:plain]<figcaption>wxWidgets (Visual Studio 2019)</figcaption></figure><

<br>  

* 参考
- [https://stackoverflow.com/questions/62827741/vs2019-wxwidgets-sample-hello-world-wxstrcoll-not-found:title]
- [https://forums.wxwidgets.org/viewtopic.php?t=44355:title]
