---
layout: post
title: GitHubリポジトリにSynopsys Coverity Scan Static Analysisのバッジをつける
tags:
  - diary
  - C++
  - hatenablog
---

今回はGitHubのREADMEでよく見かけるCoverity Scanのバッジをつける流れのメモです。

[https://camo.githubusercontent.com/c5ac830f3fb7dfc432f31df74c22978336950978c40ba6ae205a1cec827b31a0/68747470733a2f2f7363616e2e636f7665726974792e636f6d2f70726f6a656374732f32323431372f62616467652e737667:image=https://camo.githubusercontent.com/c5ac830f3fb7dfc432f31df74c22978336950978c40ba6ae205a1cec827b31a0/68747470733a2f2f7363616e2e636f7665726974792e636f6d2f70726f6a656374732f32323431372f62616467652e737667]  ← コレ

[:contents]

* Synopsys Coverity Scan とは <hr>  
　Coverityは、Synopsys社が提供する静的コード解析ソフトウェアで、Coverity Static Analysisではビルドのプロセスから生成したグラフから網羅的な解析を行います。Coverityではこのような方式をとっていることから、低レベルなエラーなども含めた高精度な解析結果を得ることができます。対応している言語は、C/C++, C#, Java, JavaScript, PHP, Python, .NET Core, ASP.NET, Objective-C, Go, JSP, Ruby, Swift, Fortran, Scala, VB.NET, iOS, TypeScriptなどです。

詳細情報は、以下のリンクから静的解析 (Coverity)のデータシートから見ることができます。
[https://www.synopsys.com/ja-jp/software-integrity/resources/datasheets.html:embed:cite]

　Coverityは、GitHubで公開されているオープンソースのソフトウェアでは、Coverity Scanというサービスにより無償で利用できます。

* 今回利用するプロジェクト <hr>  
今回はCMakeを利用した次のC++プロジェクトに対して解析を行います。特にwxWidgetsという外部ライブラリをリンクしているため、コード解析の例として参考になると思います。
[https://github.com/pit-ray/win-vind:embed:cite]

このソフトウェアは、WindowsをVimのキーバインディングで操作するツールとして公開しているもので、興味のある方は次の記事をご覧ください。
[https://www.pit-ray.com/entry/win-vind-des:embed:cite]


* 手順  <hr>  
Coverity Scanは次のサイトから利用します。
[https://scan.coverity.com/:embed:cite]

** GitHubとの連携 <hr>  
まずは、GitHubとのリンクを行うために、右上のSign upからSign in with GitHubを押します。

[f:id:pit-ray:20210112173841j:plain]

[f:id:pit-ray:20210112174038j:plain]

すると、Coverity ScanによるGitHubへのOAuthの認証が要求されるので、許可します。
[f:id:pit-ray:20210112174406j:plain]

** プロジェクトの追加 <hr> 
アカウントを連携したら次のリンクから自分のプロジェクトを選択します。
[https://scan.coverity.com/projects/new?tab=github_index]

そうしたら、ライセンスやホームページURLなどの必要な事項を記入し、Submitします。今回の場合は、自分のリポジトリであるので、RoleをMaintainer/Ownerとしていますが、適宜適切なものを選択します。
[f:id:pit-ray:20210112174919j:plain]

すると、Project Settingsの設定画面が表示されると思います。ここで、Quick Start GuideからSubmit Buildを選択します。
[f:id:pit-ray:20210112175817j:plain]

** 解析 <hr>
解析のためのファイルはUpload Buildのページからアップロードします。ここでバージョンや説明などの必要な事項を入力します。

[f:id:pit-ray:20210112175711j:plain]

初めに述べたように、Coverity Static Analysisはビルド時のプロセスを監視して解析用のグラフを生成します。そこで、Coverityビルドツールをダウンロードし、オフラインで解析用のファイルを生成したうえで、そのファイルをアップロードします。このような手順を踏むことで、外部ライブラリなどのプロジェクト固有の環境の変化に対応することができます。

まず、以下のページからビルドツールをダウンロードします。
<span style="font-size: 150%">[https://scan.coverity.com/download:title=Coverity Scan - Download the Coverity Scan Build Tool: C/C++]</span>


必要な言語のページから、適切なOSを選択します。
[f:id:pit-ray:20210112180941j:plain]

すると、826 MBのデカい圧縮ファイルがダウンロードできるので、解凍し、そのうちの<span style="color: #ff5252">binディレクトリのパスを通します</span>。

次に、GitHubから自分のリポジトリを落とします。
>|sh|
$ git clone https://github.com/pit-ray/win-vind.git
$ cd win-vind
||<

MinGW + CMakeの場合は、次のように解析グラフを生成します。
>|sh|
$ mkdir debug_s
$ cd debug_s
$ cmake -DCMAKE_BUILD_TYPE=Debug -G "MinGW Makefiles" -DCMAKE_SH="CMAKE_SH-NOTFOUND" ..
$ cov-build --dir cov-int cmake --build .
||<

最後のcov-buildで、死ぬほど時間がかかりますが気長に待ちます。成功すれば、Compilation units (100%) are ready for analysysと表示され、cov-intディレクトリが生成されます。

このディレクトリをtarコマンドやzipコマンドで圧縮するか、Windowsの標準UIで圧縮します。
[f:id:pit-ray:20210112182646j:plain]

そうしたら、先ほどの Upload a Project Buiildページのファイル選択から圧縮したcov-int.zipを選択します。アップロードと解析にかなりの時間を必要とするはずなので、また待ちます。

解析が終わると、次のように解析結果を見ることができます。ライブラリも含んで41万行のバカでかいプロジェクトのように表示されています。

[f:id:pit-ray:20210112183111j:plain]


** バッジを貼り付ける <hr>  
ダッシュボードのProject Settingsを見ると、下部にバッジのHTMLがあるので、README.mdに貼り付けます。
[f:id:pit-ray:20210112183554j:plain]


以上で終わりです。参考になれば幸いです。
