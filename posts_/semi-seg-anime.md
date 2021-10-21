---
layout: post
title: GANを使ってデータセットを増やしたい【Semi-Supervised Anime Semantic Segmentation】 
tags:
  - GAN
  - ML
  - Semantic Segmentation
  - Python
  - hatenablog
---

[:contents]


* はじめに<hr>
　今回はAdversarial Learning for Semi-Supervised Semantic Segmentation[7]という論文を参考に、小規模のデータセットでSemantic Segmentationを行いました。経緯としては、pix2pixでラベルマップからイラスト生成ができたら面白いだろうというのがあります。しかしながら、手作業のアノテーションは、60枚ほどにして心が折れましたので、自動化してやろうという魂胆です。

次のような256 x 256のラベルマップを用意しました。
<span style="color: #dd830c">背景</span>、<span style="color: #ff0000">目</span>、<span style="color: #00cc00">顔と首</span>、<span style="color: #2196f3">髪</span>、<span style="color: #cc00cc">その他</span>
といった具合です。
<figure class="figure-image figure-image-fotolife" title="レム">[f:id:pit-ray:20191115211101j:plain]<figcaption>レム</figcaption></figure>

アノテーションをきれいに行えば、一枚のイラストからレイヤ情報を自動抽出するようなアプリケーションに利用できるかもしれません。

結果から言うと、ラベル無しの10000枚のデータセットに対して、ある程度のクオリティのラベルが半数ほど得られました。しかしながら、そのクオリティを判別することが容易でないため、正直にアノテーションしたほうが良かった気もします。

記事中に何度もリンクを載せていますが、ここにもGitHubへのリンクを置いておきます。
今は亡きChainerでの実装となります。
[https://github.com/pit-ray/Anime-Semantic-Segmentation-GAN:embed:cite]



* 事前知識<hr>
** Semantic Segmentation<hr>
　ニューラルネットワークを用いた最も基本的なタスクは、<b>分類</b>といえます。人間もそうですが、何かを認識しなくては何も進みません。まず、次の図をご覧ください。
<figure class="figure-image figure-image-fotolife" title="手書数字認識">[f:id:pit-ray:20191120202727j:plain:w500]<figcaption>手書数字認識</figcaption></figure>

　この図は、数字の画像がどのクラスに属するか判断するニューラルネットワークを概念的に表しています。一般的には一次元のデータのスコアをもとにソフトマックスで分類を行います。

　対して、Semantic Segmentationは一枚の画像を1ピクセルに置き換えようという試みです。つまり、ピクセル単位でどのクラスに属するのかを判別する<b>検出</b>です。

<figure class="figure-image figure-image-fotolife" title="Semantic Segmentation">[f:id:pit-ray:20191120204618j:plain:w400]<figcaption>Semantic Segmentation</figcaption></figure>

　上の図で示すように、出力層は<b>高さ(H) x 横(W) x クラス(C)</b>のマップとして出力します。そして先ほど述べた通り、クラス方向にソフトマックスを適用することで、<b>そのピクセルが何を表しているのか</b>を判別させます。

それでは、ネットワークのアーキテクチャを見ていきます。
<figure class="figure-image figure-image-fotolife" title="Different types of networks for semantic segmentation（出典[1]）">[f:id:pit-ray:20191120211044j:plain]<figcaption>Different types of networks for semantic segmentation（出典[1]）</figcaption></figure>
　最も基本的なSemantic Segmentationのアーキテクチャは、FCN(Fully Convolutional Networks)[3]です。畳み込み層を連ね、クラス分けを行います。

　そのFCNの派生形として、U-NetやSegNetなどのEncoder-Decoder構成のネットワークがあります。これらは、位置情報を保持するためにスキップ構造を追加しています。上の図(b)でいう矢印のことです。

　そして現在のSOTAは、Dilated Convolutionを用いた構成で、DeepLabやFastFCN[1]などがあります。<b>今回はDeepLab(Dilated FCN)を用いたネットワークを主として扱います</b>。



** DeepLab <hr>
 　DeepLabは2016年にLiang-Chieh Chen氏らが考案したものですが、その後v2[5], v3[4]ときてv3+と進化を続けてきました。特にFastFCN[1]はDeepLabの動作を近似することで、高速でより高い精度を出しています。今回は、DeepLab v3をベースとしました。

*** Atrous Convolution
　DeepLabを形成する根幹のメソッドは、<b>Atrous Convolution (Dilated Convolution)</b>です。これは、CNNなどで扱われる畳み込み演算を拡張したものとされています。元のフィルターの大きさを[tex:k × k]、dilated rateを[tex:r]として

<span style="font-size: 120%">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[tex: k^{\prime}=k+(k-1)(r-1)]</span>

となる[tex: k^{\prime}×k^{\prime}]のフィルターを用いた畳み込み演算です。

次の図は、[tex: k=3, r=2]の具体例です。
<figure class="figure-image figure-image-fotolife" title="Atrous Convolution Filter ([tex:k=3, r=2])">[f:id:pit-ray:20191121003544j:plain:w200]<figcaption>Atrous Convolution Filter ([tex:k=3, r=2])</figcaption></figure>

したがって、[tex: r=1]のときは通常の畳み込みと同じになります。

　Atrous Convolutionのメリットは、<b>パラメータを増やすことなく、大きな視野をもつことができる</b>点です。従来のFCNではダウンサンプリングをする代わりにチャンネル方向に深くすることで、ピクセルにRGB以上の情報を持たせるように畳み込みを行いました。しかしながら、ダウンサンプリングで解像度を落とすと位置的な情報が失われ、Semantic Segmentationにおいてはあまり嬉しくありません。そこで、Atrousな大きなフィルターで見渡すことにより、高解像度でも従来と同等の畳み込みができるようになったのです。

　DeepLabでは、従来通りにダウンサンプリングした後、出力層側でAtrous Convolutionを用いてある程度の解像度を保持します。その後は、v2では双線形補間(bilinear interpolation)で教師データ側の解像度を落とし、低解像度で損失を算出しますが、v3ではGeneratorの出力側をアップサンプリングして、教師データラベルの解像度に合わせます。

　ちなみに多くの機械学習フレームワークでは、Atrous ConvolutionはDilated Convolutionであったり、[tex:r]を指定するDilationパラメータとして実装されています。


*** Atrous Spatial Pyramid Pooling (ASPP)
　前節ではAtrous Convolutionが[tex:r]によって視野を広げられることを確認しました。ここで解説するAtrous Spatial Pyramid Pooling (以下ASPP)は、複数のAtrous Convolutionを用意し、それぞれの[tex:r]を変えて重ねることで<b>ミクロからマクロのスケール(ピラミッド状)で情報を得よう</b>という試みです。

<figure class="figure-image figure-image-fotolife" title="Atrous Spatial Pyramid Pooling">[f:id:pit-ray:20191210042944j:plain]<figcaption>Atrous Spatial Pyramid Pooling</figcaption></figure>

それぞれのスケールで畳み込み(atrous-BN-Relu)を行った後、concatenateして畳み込みでまとめます。詳しい実装はGitHubをご覧ください。

[https://github.com/pit-ray/Anime-Semantic-Segmentation-GAN/blob/master/architecture.py#L10-L60:embed:cite]


以上がDeepLab v3のメインメソッドです。
<span style="font-size: 80%">v2以前ではCRFsを導入していますが、v3以降はそうではないので割愛させていただきます。
</span>

それでは、今回の根幹となる手法について見ていきます。


* 手法<hr>
　まず、GANを用いた半(弱)教師ありSemantic Segmentationの概要をご覧ください。

** 概要<hr>
<figure class="figure-image figure-image-fotolife" title="Semi-Supervised Semantic Segmentationの概要(出典:[7])">[f:id:pit-ray:20191210050409j:plain]<figcaption>Semi-Supervised Semantic Segmentationの概要(出典:[7])</figcaption></figure>

　上図のSegmentation NetworkとはGANでいうGeneratorに当たるネットワークであり、この訓練モデルを得るのが最終的な目的です。また、図中のDiscriminatorは本物か偽物かの確率をピクセルレベルでConfidence Mapとして出力します。したがって、Discriminatorの出力は0から1のグレイスケール画像となり、本物の場合は白くなり、偽物の場合は黒くなります。

** ネットワーク構成<hr>
元の論文[7]では、GeneratorにはImageNetで訓練済みのResNet-DeepLab v2、DiscriminatorにはFCN(skip構造なし)を用いています。今回はGeneratorとして、Dilated FCN DeepLab v3とResNet101-DeepLab v3を用いました。前者はFCNにAtrous機構をくっつけただけで事前学習なしです。後者はImageNetで事前学習アリです。ただし、アップサンプリングには、Pixel Shuffler[8]を用いています。また、Discriminatorは元の論文と同じです。



** 損失関数<hr>
それでは、損失関数について見ていきます。<span style="font-size: 80%">ただし、論文中の記号はこちらで適宜変更しています。</span>

Segmentation Network(以下Generator)とDiscriminatorの損失関数を以下のように設定します。

　　[tex:L_D=L_{adv}]

　　[tex:L_G=L_{ce}+\lambda_{adv}L_{adv}+\lambda_{semi}L_{semi}]

ただし、Generatorが生成したスコアのベクトルを[tex:g]、教師onehotベクトルを[tex:x]とします。

***adversarial loss
　元の論文[7]では、GANの元の論文[2]で示されている損失関数と同じものを用いています。対して、<b>今回はhinge lossを用いました</b>。
<!--因みに、GANについては既知であるとして、最下部のAppendixにadversarial lossの詳細を載せました。-->


*** cross entropy loss
　上の[tex:L_{ce}]に対応するものですが、分類でよく用いられるものと同じです。ピクセルごとに交差エントロピーを求め、平均か和をとります。というのも、Generatorの出力はチャンネル方向にクラス分けされたスコアのベクトルです。したがって、あるピクセルがそのクラスに属する確率の分布としてみなすことができます。

　　[tex:L_{ce}=-x\log(g)]



*** semi-supervised loss
　この損失は、半教師あり学習時に適用するものです。Generatorから生成した画像とそのDiscriminatorの出力のみで学習を行います。次式は1ピクセル分のlossを表しており、最終的にはこの平均か和を取ります。

　　[tex:L_{semi}=-I(D(g)>T_{semi})*Y\log(g)]

まず、[tex:T_{semi}]は閾値(threshold)であり、Generatorが作ったラベルのどの部分を信頼するかというパラメータとなります。例えば、次のようにGeneratorが生成したラベルをDiscriminatorに入れた結果、中心部分が本物だろうという予想がなされたとします。

<figure class="figure-image figure-image-fotolife" title="confidence mapの値分布(gnuplotより作成. 画像の大きさは適当です)">[f:id:pit-ray:20191211233205j:plain]<figcaption>confidence mapの値分布(gnuplotより作成. 画像の大きさは適当です)</figcaption></figure>

このとき、次のように<b>ここまでは信頼できる</b>という値を決め、それより上を<b>本物だと断定</b>させます。

<figure class="figure-image figure-image-fotolife" title="threshold面で切り取る">[f:id:pit-ray:20191211233208j:plain:w300]<figcaption>threshold面で切り取る</figcaption></figure>

上で示した[tex:I]という関数は、このような操作をする指示関数です。Pythonで実装する場合は、比較演算子で比べるだけで実現できます。

次に、[tex:Y]について考えます。まず、Generatorの生成物は、チャンネル方向にクラスラベルが決められているようなスコアのベクトルとなります。（下に再掲）

<figure class="figure-image figure-image-fotolife" title="Generatorの生成物(再掲)">[f:id:pit-ray:20191120204618j:plain:w300]<figcaption>Generatorの生成物(再掲)</figcaption></figure>

このベクトルに対し、ピクセル単位で<b>スコアが最も大きいクラスを、そのピクセルのクラスと断定します</b>。1ピクセル目は車、2ピクセル目はネコといった具合です。[tex:Y]は、その断定したクラスのみを1とした教師データと同形状のonehotベクトルを表しています。

以上により、Discriminatorが本物と断定したonehotマップと、Generatorがクラスを断定したonehotベクトルが用意できました。これらを要素ごとに掛け算して、<b>疑似的な教師データを作り出します</b>。そのうえで、[tex:L_{ce}]と同様のcross entropy lossを計算するのです。

因みに、Chainerでは次のように実装できます。GitHubからコピペしただけです。

>|python|
    #semi-supervised loss
    #HW-filter
    confidence_mask = F.sigmoid(unlabel_d).array > opt.semi_threshold

    #C-filter (which does pixels belong to each class)
    class_num = unlabel_g.shape[1]
    xp = cuda.get_array_module(unlabel_g.array)

    predict_index = unlabel_g.array.argmax(axis=1)
    predict_mask = xp.eye(class_num,
        dtype=unlabel_g.dtype)[predict_index].transpose(0, 3, 1, 2)

    #CHW-filter
    ground_truth = confidence_mask * predict_mask

    st_loss = -F.mean(ground_truth * F.log(unlabel_g + eps))
||<
[https://github.com/pit-ray/Anime-Semantic-Segmentation-GAN/blob/master/loss.py#L71-L86:title]

　[tex:L_{ce}]と[tex:L_{semi}]は、教師アリの場合は前者、ナシの場合は後者を使うといった形に切り替えます。また、[tex:L_{semi}]を計算する際にはある程度学習したDiscriminatorが必要なので、教師アリのみでしばらく学習したのちに、発火させ、教師ナシデータも含めるようにします。


* Dilated FCNによる結果とその詳細 <hr>
　教師アリデータは66枚で、教師ナシデータは約500枚です。ただし、教師アリデータにのみ、左右反転とγ調整によるData Augmentationを施し、4倍のデータ量で行いました。

ハイパーパラメータは次の通りです。ただし、GはGenerator、DはDiscriminatorを表します。
|*ハイパーパラメータ|*値|*[https://github.com/pit-ray/Anime-Semantic-Segmentation-GAN/blob/master/options.py:title=options.py]のオプション名|
|batch size|32|batch_size|
|G Adam learning rate|0.00025|g_lr|
|G Adam beta1|0.9|g_beta1|
|G Adam beta2|0.99|g_beta2|
|G weight decay|0.0004|g_wight_decay|
|D Adam learning rate|0.0001|d_lr|
|D Adam beta1|0.9|d_beta1|
|D Adam beta2|0.99|d_beta2|
|max epoch|2000 epoch|max_epoch|
|iteration of starting semi-supervised learning|5000 iteration|semi_ignit_iteration|
|use Spectral Normalization[6]|False|conv_norm|
|G network unit size|64|ngf|
|D network unit size|64|ndf|
|ASPP unit size|256|aspp_nf|
|adversarial loss|hinge loss|adv_conv_mode|
|[tex:\lambda_{adv}]|0.01|adv_coef|
|[tex:\lambda_{adv}] (semi-supervised)|0.001|semi_adv_coef|
|[tex:\lambda_{st}]|0.1|semi_st_coef|
|[tex:T_{semi}]|0.2|semi_threshold|

これらの生成結果が次の通りです。pix2pixHDで逆に通した結果を一番上にオマケで載せておきます。一番下が後からアノテーションした正しいラベルです。

<figure class="figure-image figure-image-fotolife" title="生成結果(上からpix2pixHD, AdvDeepLab, SemiAdvDeepLab, ground-truth)">[f:id:pit-ray:20191121190241j:plain]<figcaption>生成結果(上からpix2pixHD, AdvDeepLab, SemiAdvDeepLab, ground-truth)</figcaption></figure>

当初、DeepLabをU-Netとして学習を行っていましたが、十分な結果は得られませんでした。

二段目は、[tex:L_{semi}]の係数[tex:\lambda_{semi}]を最後まで0にした時の結果です。したがって教師あり画像のみで学習した結果となります。ある程度のセグメンテーションはできていますが、pix2pixHD同様、ノイズが非常に多く、対応しきれていません。

三段目が、今回のメソッドで生成した結果ですが、データの拡張という観点から、教師ナシとして用意した訓練データからの生成結果です。この結果を見ると、左から3番目のような小さな眼や、一番右のように眼鏡をかけた場合は、うまく判別ができていないことが分かります。


* 訓練済みResNetによる結果とその詳細 <hr>
　ここでは、アノテーションを行った700枚の教師アリデータと、約10000枚の教師ナシデータで学習を行います。また、先ほどと同様に教師データにはDeta Augmentationを施し、4倍の2800枚としました。

これらの10000枚のデータセットは、safebooruでスクレイピングを行った30000枚のデータに対し、lbpcascade_animefaceや背景判定を行い、絞り込んだものです。

ハイパーパラメータは次の通りです。ただし、変更箇所だけ示します。
|*ハイパーパラメータ|*値|*[https://github.com/pit-ray/Anime-Semantic-Segmentation-GAN/blob/master/options.py:title=options.py]のオプション名|
|batch size|5|batch_size|
|G Adam beta1|0.0|g_beta1|
|G Adam beta2|0.9|g_beta2|
|D Adam beta1|0.0|d_beta1|
|D Adam beta2|0.9|d_beta2|
|max epoch|100 epoch|max_epoch|
|iteration of starting semi-supervised learning|10000 iteration|semi_ignit_iteration|
|use Spectral Normalization[6]|True|conv_norm|


しかしながら、これらのハイパーパラメータとネットワークの構成では、30000イテレーションあたりで発散してしまい、単色の画像となりましたので、その直前の結果を次に示します。
<figure class="figure-image figure-image-fotolife" title="pre-trained ResNet101によるSemantic Segmentation">[f:id:pit-ray:20200124213414j:plain]<figcaption>pre-trained ResNet101によるSemantic Segmentation</figcaption></figure>
イラストなどの著作物を研究やデータ分析の目的で用いるのは問題ありませんが、ここに張り付けるのは低解像度でもグレーゾーンであるため、ぼかしを加えております。
また、一度、700枚すべてにラベルに関して、<span style="color: #5ed361">顔と首を緑</span>としていたところを、<span style="color: #5ed361">顔だけ緑</span>と手作業で変更しました。

というのも、これと並行して行っていた、ラベル→イラスト生成においては首と顔の判別がうまくなされなかったからです。

　先ほどのDilated-FCNと比べると、特に服と重なっている髪の認識精度があがったような気がしますが、この例と同程度のクオリティはせいぜい5割といったところです。

未知のデータセットでは、次のような結果となりました。まず、比較的成功したものです。
><figure class="figure-image figure-image-fotolife" title="比較的良いもの">[f:id:pit-ray:20210321123928j:plain:w200]<figcaption>比較的良いもの</figcaption></figure><

次に失敗例です。（グロい）
><figure class="figure-image figure-image-fotolife" title="失敗例">[f:id:pit-ray:20210321124035j:plain:w200]<figcaption>失敗例</figcaption></figure><

　学習データセットから当然の結果ですが、高い精度が期待できるのは、背景が白く、頭から肩が見えるようなイラストだと言えます。逆に、背景が複雑なものであったり、画像いっぱいに顔があるような場合では、極めて精度が落ちています。

　参考にした論文[7]ほどの性能が出ていない原因としては、独自のデータセットに対するハイパーパラメータの最適化を行っていないことや、Segmentationのクラスがざっくりしすぎていること、加えて、アノテーションの甘さが出てしまった可能性も大いに考えられます。

IoU等で定量的な評価を行っていない点については、お許しください。

<hr>  

念のため、PCの環境を記載いたします。
OS: Windows10 Home
CPU:  AMD Ryzen 2600
GPU: MSI GTX 960 4GB
Lang: Python 3.7.1
framework: Chainer 7.0.0, cupy-cuda91 5.3.0


今回参考にした文献は次の通りです。

*参考文献<hr>
[1] Huikai Wu, Junge Zhang, Kaiqi Huang, Kongming Liang, Yizhou Yu. FastFCN: Rethinking Dilated Convolution in the Backbone for Semantic Segmentation. <i>arXiv preprint [https://arxiv.org/abs/1903.11816:title=arXiv:1903.11816, 2019(v1)]</i>

[2] Ian J. Goodfellow, Jean Pouget-Abadie, Mehdi Mirza, Bing Xu, David Warde-Farley, Sherjil Ozair, Aaron Courville, Yoshua Bengio. Generative Adversarial Networks. <i>arXiv preprint  [https://arxiv.org/abs/1406.2661:title=arXiv:1406.2661, 2014]</i>

[3] Jonathan Long, Evan Shelhamer, Trevor Darrell. Fully Convolutional Networks for Semantic Segmentation. <i>arXiv preprint [https://arxiv.org/abs/1411.4038:title=arXiv:1411.4038, 2015]</i>

[4] Liang-Chieh Chen, George Papandreou, Florian Schroff, Hartwig Adam. Rethinking Atrous Convolution for Semantic Image Segmentation. <i>arXiv preprint [https://arxiv.org/abs/1706.05587:title=arXiv:1706.05587, 2017 (v3)]</i>

[5] Liang-Chieh Chen, George Papandreou, Iasonas Kokkinos, Kevin Murphy, Alan L. Yuille. DeepLab: Semantic Image Segmentation with Deep Convolutional Nets, Atrous Convolution, and Fully Connected CRFs. <i>arXiv preprint [https://arxiv.org/abs/1606.00915:title=arXiv:1606.00915, 2017 (v2)]</i>

[6] Takeru Miyato, Toshiki Kataoka, Masanori Koyama, Yuichi Yoshida. Spectral Normalization for Generative Adversarial Networks. <i>arXiv preprint [https://arxiv.org/abs/1802.05957:title=arXiv:1802.05957, 2018]</i>

[7] Wei-Chih Hung, Yi-Hsuan Tsai, Yan-Ting Liou, Yen-Yu Lin, Ming-Hsuan Yang. Adversarial Learning for Semi-Supervised Semantic Segmentation. <i>arXiv preprint [https://arxiv.org/abs/1802.07934:title=arXiv:1802.07934, 2018 (v2)]</i>

[8] Wenzhe Shi, Jose Caballero, Ferenc Huszár, Johannes Totz, Andrew P. Aitken, Rob Bishop, Daniel Rueckert, Zehan Wang. Real-Time Single Image and Video Super-Resolution Using an Efficient Sub-Pixel Convolutional Neural Network. <i>arXiv preprint [https://arxiv.org/abs/1609.05158:title=arXiv:1609.05158, 2016]</i>


<!--
*Appendix <hr>
** adversarial loss
　GANはGeneratorとDiscriminatorのミニマックスゲームであり、私たちはゲーム理論でいうナッシュ均衡になることを望みます[5]。その状況を次の関数[tex:V(D, G)]として表せます[5]。[tex:\mathrm{E}] は期待値を表します。

　　[tex:\underset{G}{\mathrm{min}}\,\underset{D}{\mathrm{max}}V(D, G)=\mathrm{E}_{x\sim p(x)}[\log{D(x)}\]+\mathrm{E}_{z\sim p(z)}[\log(1-D(g))\]]

この式に関して順を追って解説します。
Discriminatorは[tex:D(x)](<span style="color: #2196f3">本物のデータ</span>を<span style="color: #2196f3">本物</span>として判断する確率)を1、[tex:D(g)]（<span style="color: #ff5252">偽物のデータ</span>を<span style="color: #2196f3">本物</span>として判断する確率）を0にすることを望みます。つまり、正しい判断を行うように訓練します。したがって、第一項目の<span style="color: #2196f3">本物</span>を<span style="color: #2196f3">本物</span>として判断する確率と、第二項目の<span style="color: #ff5252">偽物</span>を<span style="color: #ff5252">偽物</span>として判断する確率を<b>最大化</b>するように実装すればDiscriminatorの[tex:L_{adv}]が作れます。二つの確率分布を引き離すようなイメージです。

Generatorは[tex:D(g)]を1にすることを望みます。そこで先ほどのように考えていきますが、少しややこしくなっています。<span style="color: #ff5252">偽物のデータ</span>を<span style="color: #2196f3">本物</span>として判断する確率を<b>最大化</b>する行為は、<span style="color: #ff5252">偽物のデータ</span>を<span style="color: #ff5252">偽物</span>として判断する確率を<b>最小化</b>する行為と同等ですので、第二項目を<b>最小化</b>するように実装します。ここで、第一項目にはGeneratorは関与していないので第二項目だけを考えます。

どちらの損失に関しても、[tex:1-D(g)]が[tex:D(g)]の余事象であることに注意すれば整理できるでしょう。


adversarial loss
それではそれぞれの[tex:L_{adv}]を考えていきますが、前提としてloss、損失というのは小さくあるべきで、多くのフレームワークでは小さくするようにパラメータの更新を行うことに注意します。

先ほどの価値関数の実装は、大きく分けてbinary-cross-entropyとsoftplusがあります。それぞれの実装を次に示します。
<span style="font-size: 80%">DiscriminatorとGeneratorで[tex:L_{adv}]と同じ記号を使っていますが、別モノです。数学に手厳しい方ゴメンなさい...</span>
ちなみに、今回のDiscriminatorの出力はconfidence mapという二次元配列ですので、ピクセル単位でのlossに対して、平均か和をとります。

<i><b>binary-cross-entropy</b></i>
Discriminatorの出力は、sigmoidを通した確率値
<i>Discriminator</i>
　　[tex:L_{adv}=-(1-t)\log(1-D(g))-t\log{D(x)}]
　　<span style="font-size: 80%">[tex:(real: t=1, fake: t=0)]</span>

<i>Generator</i>
　　[tex:L_{adv}=-\log(D(g))]

<i><b>softplus</b></i>
Discriminatorの出力は、sigmoidを通す前のスコア
<i>Discriminator</i>
　　[tex:L_{adv}=\Sigma (1-t)\log(1+e^{D(g)})+t\log(1+e^{-D(x)})]
　　<span style="font-size: 80%">[tex:(real: t=1, fake: t=0)]</span>

<i>Generator</i>
　　[tex:L_{adv}=\log(1+e^{-D(g)})]

これらの大きな違いはDiscriminatorにsigmoidを含めるかどうかですが、前者は確率が0に近い場合に[tex:\log]が発散してしまうので、実際には後者がよく用いられます。

softplusの動作については、次のページが参考になります。
[https://tatsukawa.hatenablog.com/entry/2018/09/08/044736:embed:cite]

-->
