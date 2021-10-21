---
layout: post
title: 【meekrosoft/fff】 MSVCでWindows APIのフェイク関数を作ると多重定義となる (LNK2005)
tags:
  - C++
  - C
  - hatenablog
---


　先日からWindows APIを多く含んだC++プロジェクトのテストを書いています。その際、Windows APIのスタブ/モック用のフレームワークとしてfffを利用しています。

[https://github.com/meekrosoft/fff:embed:cite]

しかし、READMEに沿ってWindows APIのフェイク関数を定義したところ、MinGW-w64ではうまくコンパイルできるものの、MSVCでは多重定義(LNK2005)となりました。

例えば次のようなコードです。

<script src="https://gist.github.com/pit-ray/93605447158d3de1ad3433e266ac5677.js"></script>
<s>
そこで、次のように無名名前空間で囲むことで内部リンケージを作り、多重定義を回避できました。
<script src="https://gist.github.com/pit-ray/5f7670d6c6865c3e8332768fc7f6ba57.js"></script>

少しハックな方法ですが、参考になれば幸いです。他の方法があればコメントください。
</s>


**追記**

MSVCのリンカに対して、多重定義が検出されてもリンクされるようにオプションを渡せばいいです。

詳細は次を参照してください。


[https://docs.microsoft.com/en-us/cpp/build/reference/force-force-file-output?view=msvc-160:embed:cite]


CMakeならば次のようにして、リンカにフラグを渡します。

```cmake
    add_link_options(/FORCE:MULTIPLE)
```
