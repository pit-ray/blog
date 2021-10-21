---
layout: post
title: カラースキーム・テーマを他のテキストエディタ用に変換するツール【TeraPad, さくらエディタ対応】
tags:
  - C++
  - hatenablog
---

今回は、テキストエディタの色設定を相互変換できるツールの紹介です。

C++の勉強の副産物として生まれたツールですので、TeraPadとさくらエディタにのみ対応しています。

ソースコードはこちらです。
[https://github.com/pit-ray/TxEditorCCTool:embed:cite]

GitHubのREADME.mdに実行方法がありますので、ご確認ください。

ビルド済みのバイナリはこちらです。
[https://github.com/pit-ray/TxEditorCCTool/tree/master/bin/release:embed:cite]

Windows 10のみですが、ソースコードをリビルドすれば、Linux等でも使えます。
CMakeでビルド可能です。

当初はQtでGUIを作っていましたが、モチベーションが保てず、断念しました。

不明な点がありましたら、コメントやGitHubのissueにお寄せください。
