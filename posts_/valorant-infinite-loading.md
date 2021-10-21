---
layout: post
title: 【VALORANT】無限ローディングの対処法
tags:
  - VALORANT
  - hatenablog
---

突然黒い画面になり、再起動後に次のようにロビー画面でフリーズしたので、その対処法です。
[f:id:pit-ray:20200602234132j:plain]

VALORANTは、チート防止のために多くの認証システムを導入しています。
[https://technology.riotgames.com/news/demolishing-wallhacks-valorants-fog-war:title]

そのせいか分かりませんが、プレイ中のままサーバに取り残されたような形になったのかと思います。
これは既存のバグのようで、海外のサイトに多くの報告がありました。

次の手順で回復できました。自己責任でお願いいたします。
<hr>
<b>1.</b> タスクトレイから
Riot Vanguard>More>Uninstall Vanguard

<b>2.</b> <a href="https://playvalorant.com/ja-jp/download/">VALORANTインストーラ</a>を起動し、Vanguardを再インストール

<b>3.</b> PCを再起動

<b>4.</b> LEAGUE OF LEGENDSをダウンロード
(インストール済みなら不要)

<b>5.</b> LOLにVALORANTと同じアカウントでログイン
Riotのサーバ上のプレイ情報をリセットするためかと思います。

<b>6.</b> 以下のファイル削除
|C:/Riot Games/VALORANT/live/Manifest_DebugFiles_Win64.txt|
|C:/Riot Games/VALORANT/live/Manifest_NonUFSFiles_Win64.txt|

<b>7.</b> VALORANTを管理者権限で起動 
プロパティ>互換性>管理者としてこのプログラムを実行する
にチェック
<hr>

LOLにログインすることで、
「違うゲームを始めたから、VALORANTは終了していいよ」
というメッセージをサーバに送る仕組みならば、実質4だけで解決するのかもしれません。
同じバグを再現することができないため、確証はありません。

また、極端にping値が高くなった場合には、画面が黒くなるようですが、待つと元に戻るそうです。


*** 参考
・[https://www.gamerevolution.com/guides/644819-valorant-stuck-on-loading-screen-bug-fix:title]
・[https://gamexguide.com/endless-loading-and-freezing-on-screen-download-in-valorant-how-to-fix-the-bug/:title]
・[https://www.reddit.com/r/VALORANT/comments/g41v83/valorant_stuck_in_loading_screen/:title=Reddit thread Valorant Stuck In Loading Screen]

*** 追記
メンテナンスが入ったようなので、修正されるかもしれません。
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">メンテナンスのお知らせ：マッチメイキング<a href="https://t.co/O1WBpvrVU2">https://t.co/O1WBpvrVU2</a> <a href="https://t.co/FJoavlCz2W">pic.twitter.com/FJoavlCz2W</a></p>&mdash; VALORANT (@VALORANTjp) <a href="https://twitter.com/VALORANTjp/status/1267841626296086528?ref_src=twsrc%5Etfw">June 2, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
