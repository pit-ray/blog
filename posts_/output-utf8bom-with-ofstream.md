---
layout: post
title: 【C++】ofstreamでUTF-8 with BOMを出力する方法
tags:
  - C++
  - hatenablog
---

　今回はC++の標準ライブラリのfstreamのwrite関数を用いた、<b>バイトオーダーマーク(BOM)</b>の付け方をご紹介します。

　以前は標準ライブラリのcodecvtを利用することで変換できましたが、非常に分かりにくいうえ、C++17よりcodecvtは非推奨となっています。そこで、今回はBOMを付与する場合に限定して独自に関数を作ってみようという試みです。

** BOMとは <hr>
　Unicodeにはいくつかの種類があります。有名なものだと、UTF-16やUTF-8などがあります。詳しい仕様などの説明は省きますが、それぞれに完全な互換性はありません。国際的な共通化を図るための文字コード同士は、似た状況で使用される可能性が多くなります。したがって、その方式を示すようなマークがファイルの先頭についていると便利です。
　
　UTF-8 with BOMの<b>BOM(バイトオーダーマーク, Byte Order Mark)</b>は、「Unicodeというのは分かったからどんな形式をしているんだい？」という問いに答えるべく、冗長な判別用データのことを指します。

　よって、UTF-8からUTF-8 with BOMへの変換はそこまで難しくはありません。プログラマは、出力するファイルがUTF-8と分かっているので、決められたデータを付与するだけです。

BOMの一覧は次の通りです。
|*符号化形式|*エンディアン|*BOM|
|UTF-8||0xEF 0xBB 0xBF|
|UTF-16|BE|0xEF 0xFF|
||LE|0xFF 0xFE|
|UTF-32|BE|0x00 0x00 0xFE 0xFF|
||LE|0xFF 0xFE 0x00 0x00|
(出典: <a href="https://ja.wikipedia.org/wiki/%E3%83%90%E3%82%A4%E3%83%88%E3%82%AA%E3%83%BC%E3%83%80%E3%83%BC%E3%83%9E%E3%83%BC%E3%82%AF">バイトオーダーマーク</a>)

** 実装 <hr>
以上より、テキストの先頭に付けるデータは
UTF-8の場合、
<span style="font-size: 150%"><span style="color: #ff5252">0xEF 0xBB 0xBF</span></span>
です。

バイナリで書き込むには以下のようにwrite関数を用いるのが直感的かと思います。ただし、システムコールやストリームのキャッシュを考慮していくと効率的に良いのかは分かりません。
>|cpp|
#include <iostream>
#include <fstream>

int main()
{
  auto create_utf8bom_stream = [](auto path) {
      std::ofstream ofs(path) ;
      const unsigned char bom[] = {0xEF, 0xBB, 0xBF} ;
      ofs.write(reinterpret_cast<const char*>(bom), sizeof(bom)) ;
      return ofs ; //apply RVO
  } ;

  //UTF-8 with BOM
  auto ofs_bom = create_utf8bom_stream("u8b") ;
  ofs_bom << "UTF-8 with BOM\n" ;

  //UTF-8
  std::ofstream ofs("u8") ;
  ofs << "UTF-8\n" ;

   return 0 ;
}
||<

　絶対パスを用いた場合、一般に長い文字列となり、SSO(Small String Optimization)が適用されない可能性があります。したがって、絶対パスを引数として渡すことが多いときは、次のように完全転送を用いているほうが設計的には良いのかもしれません。

>|cpp|
auto create_utf8bom_stream = [](auto&& path) {
  std::ofstream ofs(std::forward<decltype(path)>(path)) ;
  const unsigned char bom[] = {0xEF, 0xBB, 0xBF} ;
  ofs.write(reinterpret_cast<const char*>(bom), sizeof(bom)) ;
  return ofs ; //apply RVO
} ;
||<

** 結果 <hr>
<figure class="figure-image figure-image-fotolife" title="バイナリエディタ(Stirling)による確認：BOMあり">[f:id:pit-ray:20181211021852j:plain]<figcaption>バイナリエディタ(Stirling)による確認：BOMあり(2行目先頭に注目)</figcaption></figure>
<figure class="figure-image figure-image-fotolife" title="バイナリエディタ(Stirling)による確認：BOMなし">[f:id:pit-ray:20181211021923j:plain]<figcaption>バイナリエディタ(Stirling)による確認：BOMなし</figcaption></figure>

このようにBOMありには、書き込んだ先頭データがあるのが分かります。

当然、VSCodeなどのエディタで開くと以下のようになります。

<figure class="figure-image figure-image-fotolife" title="VSCodeによる判定：BOMあり">[f:id:pit-ray:20181211021532j:plain]<figcaption>VSCodeによる判定：BOMあり</figcaption></figure>
<figure class="figure-image figure-image-fotolife" title="VSCodeによる判定：BOMなし">[f:id:pit-ray:20181211021529j:plain]<figcaption>VSCodeによる判定：BOMなし</figcaption></figure>

以上です。先ほど、示した表のバイト列を書き込めば、UTF-16やUTF-32も同様にBOMを付与できます。
