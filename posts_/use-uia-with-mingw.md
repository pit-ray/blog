---
layout: post
title: MinGWでUI Automationを使う (CMake, MinGW-w64, uiautomationclient.h)
tags:
  - C++
  - MinGW
  - UIA
  - hatenablog
---


　C++のコンパイラを自由に選べる環境ならば、GNUのg++を利用したいと考えている人は少なくないかと思います。Windowsでは、MinGWやCygwinなどを利用することになると思いますが、付属されているWindows SDKが完全なモノとは限りません。今回は頻繁に利用されないWindows APIを利用し、不完全ゆえのエラーが出力された時の対策を考えます。

* Microsoft UI Automation <hr>
　<b>Microsoft UI Automation (UIA)</b> は、プログラムからWindowsのGUIのコンポーネントを操作するためのAPIです。前身である<b>Microsoft Active Accessibility (MSAA)</b>は、身体的または認知的困難を持つ障碍者やアプリケーションの自動化のためにWindows 95から導入されています。身近なものだと、Windowsのナレーター機能などがあります。ナレーションをするには、UIを認識する必要があるわけです。UIAとMSAAはどちらも<b>Component Object Model(COM)</b>に基づいています。

参考
[https://docs.microsoft.com/en-us/windows/win32/winauto/entry-uiauto-win32:embed:cite]


* 問題 <hr>
　Mingw-w64にて、次のように、単にインクルードしたところ、宣言・定義されていないとのエラーを大量に吐かれました。
>|cpp|
#include <uiautomationclient.h>
||<

　そこで、MinGWの<b>uiautomationclient.h (MinGW/x86_64-w64-mingw32/include)</b>を参照したところ、マクロの羅列ばかりで実装が不完全でした。

2015年にも同様の報告があります。
[https://sourceforge.net/p/mingw-w64/feature-requests/74/:embed:cite]
最新のMinGWとMinGW-w64のソースコードを確認しましたが、未だ実装されていません。

* 対策 <hr>
　公式で提供されているWindows SDKを利用するのが手っ取り早いので、CMakeでWindows SDKを検出し、MinGWと使い分けるようにインクルードします。

Windows SDKは次のリンクからインストールできます。
[https://developer.microsoft.com/en-us/windows/downloads/windows-10-sdk/:embed:cite]

CMakeでは、WindowsのレジストリからWindows SDKのパスを読み出して、最新のWindows SDKのパスの検出を行うようにします。
>|cmake|
# CMake
set(WINDOWS_SDK_INCLUDE_DIR "")
if("${WINDOWS_SDK_INCLUDE_DIR}" STREQUAL "")
    get_filename_component(WINDOWS_SDK_ROOT_DIR "[HKEY_LOCAL_MACHINE\\SOFTWARE\\WOW6432Node\\Microsoft\\Windows Kits\\Installed Roots;KitsRoot10]" REALPATH)
    file(GLOB WINDOWS_SDK_INCS LIST_DIRECTORIES true "${WINDOWS_SDK_ROOT_DIR}/Include/10.0.*.*")
    list(SORT WINDOWS_SDK_INCS ORDER DESCENDING)
    list(GET WINDOWS_SDK_INCS 0 WINDOWS_SDK_INCLUDE_DIR)
endif()
if(NOT EXISTS ${WINDOWS_SDK_INCLUDE_DIR})
    message(FATAL_ERROR "Could not find the directory; ${WINDOWS_SDK_INCLUDE_DIR}. You can fix by the cmake flag; -DWINDOWS_SDK_INCLUDE_DIR=<Path>")
endif()
message(STATUS "Detected Windows SDK: ${WINDOWS_SDK_INCLUDE_DIR}")
||<

以上で、<b>${WINDOWS_SDK_INCLUDE_DIR}</b>に次のような値が得られます。
>|cmake|
C:/Program Files (x86)/Windows Kits/10/Include/10.0.19041.0
||<

したがって、<b>${WINDOWS_SDK_INCLUDE_DIR}</b>を<b>include_directories</b>で読み込むことで、次のようにMinGWのものと区別して<b>uiautomationclient.h</b>を読み込むことができます。
>|cpp|
#include <um/uiautomationclient.h>
||<


GitHubにも置いておきます。  
[https://github.com/pit-ray/using-winsdk-with-mingw:embed:cite]

　
* UI AutomationのGUID <hr>
　先ほどの手順で<b>uiautomationclient.h</b>を読み込むことはできますが、COMオブジェクトが必要とするGUIDが定義されていないため、<b>CLSID_CUIAutomation</b>や<b>IID_CUIAutomation</b>などでリンクエラーが生じると思います。

** 対策1: Windows SDKのGUIDをまとめたライブラリをリンクする
　先ほどインクルードしたWindows SDKのlibディレクトリの<b>libuuid.lib</b>というライブラリをリンクします。ただ、プロジェクトによってはその他のライブラリと競合し、multiple definitionとしてリンクエラーを吐かれることがあります。私は、wxWidgetsと併用してuuidライブラリをリンクしたところ、定義の重複が生じました。その際、リンカに<b>--allow-multiple-definition</b>オプションを渡すことでリンクすることができます。

** 対策2: GUIDを定義する
　GUIDは、基本的に一意な128 bitの定数なので、自前で定義することも可能です。WindowsのGUIDは以下のデータベースで検索することができます。
[https://www.magnumdb.com/:embed:cite]

これにより、次のように定義が可能です。
>|cpp|
#include <initguid.h>

DEFINE_GUID(IID_IUIAutomation,   0x30cbe57d, 0xd9d0, 0x452a, 0xab, 0x13, 0x7a, 0xc5, 0xac, 0x48, 0x25, 0xee) ;
DEFINE_GUID(CLSID_CUIAutomation, 0xff48dba4, 0x60ef, 0x4201, 0xaa, 0x87, 0x54, 0x10, 0x3e, 0xef, 0x59, 0x4e) ;
||<


参考になれば幸いです。
