---
layout: post
title: 【Qt】CDBをデバッガとして利用する方法
tags:
  - C++
  - hatenablog
---

今回は、QtにCDBをデバッガとして設定する方法を解説します。

Qtでデバッグ開始時に以下のようなエラーを吐かれてデバッグができないケースがあります。
 [f:id:pit-ray:20180828212259j:plain]

>>
The selected debugger may be inappropriate for the inferior.
Examining symbols and setting breakpoints by file name and line number may fail.

The inferior is in the Portable Executable format.
Selecting CDB as debugger would improve the debugging experience for this binary format.
<<

エラーの通り、<b>CDBをデバッガにする</b>と改善が見込めるとあります。
Qtの初期設定では、<u>CDBが既にインストールされている場合</u>には自動検出されるため、CDBがPC上にないことが考えられます。
よって、CDBをインストールすればOKです。

まず、CDBのインストーラーを入手する必要があります。

以下のWebサイトから得られるインストーラーは、<b>Windows Driver Kit</b>といい、様々なツールが含まれています。
このWDKの中にCDBがあります。
[f:id:pit-ray:20180828220226j:plain]
[https://docs.microsoft.com/ja-jp/windows-hardware/drivers/download-the-wdk:title]





次に、WDKインストーラーを入手したら、この中にあるCDBインストーラーのみを実行します。
CDBインストーラーのみ実行するには以下の手順を踏みます。

①WDKインストーラーを実行し、
<span style="color: #F5A2A2"><b>Download the Windows Driver Kit - Windows 10.0.17134.1 for installation on a separate computer</b></span>
からインストール。
[f:id:pit-ray:20180828221043j:plain]

②次にダウンロード先のディレクトリ<span style="color: #F5A2A2"><b>Windows Kits/10/WDK/Installers</b></span>から
<span style="color: #F5A2A2"><b>X64 Debuggers And Tools-x64_en-us.msi</b></span>または
<span style="color: #F5A2A2"><b>X86 Debuggers And Tools-x86_en-us.msi</b></span>を実行します。
[f:id:pit-ray:20180828222044j:plain]
<b>Windows 64bit：x64
Windows 32bit：x86</b>


このインストーラーがCDBインストーラーになります。


CDBインストーラーは流れにそってそのまま進めるとインストールが完了し、自動でQtに適用されます。
[f:id:pit-ray:20180828223005j:plain]
自動で適用されない場合、手動でcdb.exeのパスを入れます。

以上です。
なにかありましたらコメントください。
