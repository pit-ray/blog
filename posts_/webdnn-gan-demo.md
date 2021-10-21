---
layout: post
title: WebDNNでブラウザで動くGANを実装した話
tags:
  - ML
  - GAN
  - JavaScript
  - Python
  - hatenablog
---

本日は近年研究が盛んなGAN（Generative Adversarial Networks）をWebアプリにしてみようというだけです。

[:contents]
<br>

* GANとは <hr/>
Generative Adversarial Networksは、GANやGANsと略され、日本語では敵対的生成ネットワークといいます。このアルゴリズムは、Goodfellow氏の論文<a href="#[1]">[1]</a>にて発表されたものです。名前の通り、敵対するネットワークを用いて学習を進めます。具体的に言うと、生成ネットワーク（Generator）と識別ネットワーク（Discriminator）です。

Goodfellow氏は、これらのネットワークを偽札における偽造者と警察の攻防になぞらえて次のように説明しています。<span style="font-size: 80%">（GANに関係する論文を読んだ人ならば、腐るほど聞いているハズです）</span>
<b>偽造者（生成ネットワーク）は、本物にそっくりな偽札を作り出そうとします。それに対し、警察（識別ネットワーク）はある紙幣を調べ、本物か偽物か判別しようとします。</b>

これが意味するのは、<b>学習後に得られるのは、データセットに似たデータを生成できるネットワークと、データセットを分類できるネットワーク</b>ということになります。今回は、MNISTデータセットを生成してくれるようなアプリケーションを想定していますので、前者が得られることが目標となります。

まず、それぞれのネットワークの概略図を見ていきます。

<figure class="figure-image figure-image-fotolife" title="GANのネットワーク構成">[f:id:pit-ray:20190904224739j:plain]<figcaption>GANのネットワーク構成</figcaption></figure>

Generatorはガウス分布から生成した乱数を種に、偽の画像を生成します。Discriminatorは従来の分類器のような構造で、読んだ情報を本物か偽物か二値分類します。

近年は、GeneratorやDiscriminatorの中身や構成を変化させ、様々な派生GANが登場していますが、基本的なアルゴリズムは変わりません。今回は、CNNを用いたDCGAN（Deep Convolutional GAN <a href="#[2]">[2]</a>）を用います。また、指定できたほうがおもしろいので、Conditional GAN <a href="#[3]">[3]</a>で組みました。
<span style="color: #999999"><span style="font-size: 80%">当初はWGAN-gpで組んでいましたが（特に、記事を書いているときに学習をさせていた）、集束が非常に遅く、生成結果もクオリティが低いことから、記事を大きく変更するﾊﾒになりました... orz</span></span>

今回のソースはGithubにて確認できます。Chainer 5.3.0で実装しました。

[https://github.com/pit-ray/webdnn-mnist-dcgan:embed:cite]
<br>
* WebDNNとは <hr/>
日本の研究の最先端ともいえる、東京大学大学院 原田研究室にて開発されたフレームワークです。このフレームワークを用いることで、Deep Neural Network（DNN）をWebで高速に実行できるような形に変換できます。

従来、DNNを用いたWebサービスを提供するには、大規模な計算機を用意する必要があり、現実的ではありませんでした。また、クライアント側でその処理を行うアイデアもありましたが、処理速度が非常に遅いという欠点がありました。

WebDNNは次の二点を行うことで、クライアント側で高速に計算を行います。

|計算量の削減|2×3を6と置き換えるような変換ルールを多数用いる|
|Web規格の利用|WebGPU、WebAssemblyなどを用いる|


実行速度や対応ブラウザなどは、以下のページで確認できます。
[https://mil-tokyo.github.io/webdnn/ja/:embed:cite]

今回は、WebAssemblyをメインに実装します。
<br>
* WebDNNでConditional DCGANを実装 <hr/>
WebDNNは、DNNモデルをPythonで組み、表面的な実装はJavaScriptで行います。必要なものは、学習済みのパラメータファイル<span style="font-size: 80%">（Chainerではsnapshotに相当）</span>とそれに対応するネットワークです。ここで、パラメータファイルとの整合性をとるのが非常に面倒であるため、学習時のネットワーク構成を変更しないようにします。

WebDNNのセットアップは、[https://mil-tokyo.github.io/webdnn/docs/:title]を見ればスグ分かると思います。
<br>
** Python側の実装 <hr/>

<span style="font-size: 80%">[https://github.com/pit-ray/webdnn-mnist-dcgan/blob/master/2pack.py:title]</span>
2pack.py
>|python|
#coding: utf-8
import numpy as np
import chainer

from webdnn.frontend.chainer import ChainerConverter
from webdnn.backend import generate_descriptor

from networks import Generator

def main():
    z_dim = 100
    device = -1 #CPU
    batch_size = 1
    model = Generator(z_dim)

    model.to_gpu()
    chainer.serializers.load_npz('result-dcgan/gen_snapshot_epoch-200.npz', model)
    model.to_cpu()

    x, _ = model.generate_noise(device, batch_size)
    y = model(x)

    graph = ChainerConverter().convert([x], [y])
    exec_info = generate_descriptor("webassembly", graph)
    exec_info.save("./model")

if __name__ == '__main__':
    main()
||<

WebDNNでは、計算グラフを用いて最適化を行います。<b>Chainerにおいては一度実行しないと計算グラフが生成されない</b>ため、仮の入力データを用意しています。

ここで注意したいのは、<b>cupyをnumpyに統一する必要がある</b>ということです。WebDNNは内部でnumpyを想定してコーディングされています。したがって、学習時にGPUを使っている場合は、to_gpuメソッドでcupyに統一したうえで読み込み、to_cpuでパラメータ類もnumpyに統一します。

また、<b>Generatorのネットワークが非常に大きい場合</b>、WebDNNで生成される最適化済みファイルの容量が膨れ上がります。この場合、レンタルサーバーなどで一度にアップロードできず、一筋縄ではいきません。対策としては、ある程度の箇所でネットワークを切断し、分割するか、パラメータを少なくするような実装が考えられます。
<br>
** JavaScript側の実装 <hr/>

<span style="font-size: 80%">[https://github.com/pit-ray/webdnn-mnist-dcgan/blob/master/app.js:title]</span>
app.js
>|javascript|
var webdnn = require('webdnn');
var nj = require('numjs');
var runner = null;

async function predict(num) {
    runner = await webdnn.load('model');

    let x = runner.inputs[0];
    let y = runner.outputs[0];

    //generate random normal distribution (z-noise)
    let array = nj.random(100);

    //conditional
    let label = nj.zeros(10);
    label.set(num, 1);
    init_x = nj.concatenate([array, label]);

    x.set(init_x.tolist());
    await runner.run();

    //draw
    var canvas = document.getElementById('output');
    webdnn.Image.setImageArrayToCanvas(y.toActual(), 28, 28, document.getElementById('output'), {
            dstW: canvas.getAttribute('width'),
            dstH: canvas.getAttribute('height'),
            scale: [255, 255, 255],
            bias: [0, 0, 0],
            color: webdnn.Image.Color.GREY,
            order: webdnn.Image.Order.CHW
        });

}

window.onload = function() {
    document.getElementById('button').onclick = function() {
        let num = document.getElementById('number').value;
        //console.log(num);
        predict(num);
    }
}
||<

JavaScript側の実装では、npmなどでパッケージとしてWebDNNを読み込みます。今回は、WebPackで実装しました。

コードの解説は必要ないと思いますが、簡単に説明いたします。

まず、GANでは入力がノイズであるため、ガウス分布の乱数を生成できる数学ライブラリnumjsを利用しました。そこに、conditional GANとしてのラベルを与えています。

WebDNNで結果として画像を表示する際には、HTML側でcanvasを用意して、そのidに書き込むように指定します。その際のオプションとして、チャンネル、高さ、幅の順番や、RGBAかRGBかGray Scaleかの選択などが可能です。（[https://mil-tokyo.github.io/webdnn/docs/api_reference/descriptor-runner/interfaces/webdnn_image.imagearrayoption.html:title]）

また、入力データのセットには、setメソッドを用います。
<br>
* 実装結果 <hr/>
[https://pit-ray.github.io/webdnn-mnist-generator/index.html:embed:cite]

処理は、クライアント側で行っています。
スタイルシートすら作っていない手抜きです(;´д｀)
<br>
* まとめ <hr/>
今回は、GANをWebDNNにてWebアプリに応用しました。WebDNNを使うことで計算用のサーバを用意する必要がないため、個人でもサービス展開が容易になると思います。

<b>ご質問等ありましたら、コメントいただけると幸いです。</b>
<br>
*参考文献 <hr/>
<span id="[1]">[1]</span> Ian J. Goodfellow, Jean Pouget-Abadie, Mehdi Mirza, Bing Xu, David Warde-Farley, Sherjil Ozair, Aaron Courville, Yoshua Bengio. Generative Adversarial Networks. <i>arXiv preprint  [https://arxiv.org/abs/1406.2661:title=arXiv:1406.2661, 2014]</i>

<span id="[2]">[2]</span> Alec Radford, Luke Metz, Soumith Chintala. Unsupervised Representation Learning with Deep Convolutional Generative Adversarial Networks. <i>arXiv preprint [https://arxiv.org/abs/1511.06434:title=arXiv:1511.06434, 2016]</i>

<span id="[3]">[3]</span> [https://qiita.com/triwave33/items/f6352a40bcfbfdea0476:title]


