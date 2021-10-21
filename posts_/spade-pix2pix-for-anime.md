---
layout: post
title: ラベルマップから二次元美少女イラストを生成する (SPADE pix2pix)
tags:
  - GAN
  - ML
  - Python
  - hatenablog
---


今回は、ピクセル上の位置情報から二次元美少女イラストを生成したのでその結果をまとめます。

ソースコードは、GitHubで参照することができます。今は亡きChainerでの実装となります。
[https://github.com/pit-ray/SPADE-pix2pix-for-Anime:embed:cite]

* モデルの概要 <hr>  
次の画像のようにSemanticなラベルと髪色の色マップを入力として与えると、ラベルに沿った構成と指定した髪色で着色されたイラストが生成されます。
<img src="https://github.com/pit-ray/SPADE-pix2pix-for-Anime/blob/master/model_overview.png?raw=true">  


今回の構成は、GauGANをベースにしています。
* アーキテクチャ <hr>
- オリジナルのSPADEでは、それぞれのSPADEが別々にラベルをエンコードする必要がありますが、StyleGANのようにマッピングネットワークで事前に計算し、そのweightをSPADEに入れ込むようにしました。ただし、FCNではダウンサンプリングにより位置情報が失われてしまうので、AtrousConvolutionで解像度を変えないConstant-FCNを実装しました。

- 髪色を指定したかったため、マッピングネットワークを二つ用意し、一つはSemanticなラベルを入力し、もう一つには髪色とその位置情報をもつ同解像度のRGBマップを入力しました。また、StyleGANのように複数のweightは混ぜて渡すようにしました。

- StyleGANのようにノイズを入れるレイヤを追加しました。

- Discriminatorには、Multi-scale discriminatorの代わりにSelfAttentionを持つPatch Discriminatorとしました。

- Discriminatorの損失には、Hinge lossとZero Centered Gradient Penaltyを用いました。

- Generatorの損失には、Hinge lossとFeature Matching lossとPerceptual lossを用いました。

そのほかは、GauGANとほぼ同じ構成なので、割愛します。というのも、SPADEが登場したときは、NVIDIAの新技術ということでバズり、日本語の解説記事がゴロゴロ転がっているからです。


* 結果 <hr> 
<img src="https://github.com/pit-ray/SPADE-pix2pix-for-Anime/raw/master/tile.png?raw=true">

データセットには、Pixivイラスト500枚の非常に小規模なものを用いています。
また、上に示したテストデータセットは、前回実装した[https://www.pit-ray.com/entry/semi-seg:title=Anime-Semantic-Segmentation-GAN]を用いて生成しました。訓練データは、手作業でアノテーションしたものを用いましたが、元の画像の著作権のため公開することはできません。

　今回得られた学習済みのパラメータはGitHubのリポジトリに置いているスクリプトを実行することで、私のGoogleDriveからダウンロードできます。100 MBしかないので比較的短い時間で落とせるはずです。
[https://github.com/pit-ray/SPADE-pix2pix-for-Anime#pretrained-weights]



* まとめ <hr>
生成された画像のクオリティはそこまで高くありませんが、粗いアノテーションと、小規模のデータセットということを考えると、十分な結果が得られたと思います。
少々手抜きすぎる記事でしたが、参考になると幸いです。


* 参考文献 <hr>
[1] Taesung Park, Ming-Yu Liu, Ting-Chun Wang, Jun-Yan Zhu. Semantic Image Synthesis with Spatially-Adaptive Normalization. <i>arXiv preprint  <a href="https://arxiv.org/abs/1903.07291">arXiv:1903.07291, 2019</a></i>  

[2] Tero Karras, Samuli Laine, Timo Aila. A Style-Based Generator Architecture for Generative Adversarial Networks
. <i>arXiv preprint <a href="https://arxiv.org/abs/1812.04948">arXiv:1812.04948, 2019</a></i>  

[3] Han Zhang, Ian Goodfellow, Dimitris Metaxas, Augustus Odena.  Self-Attention Generative Adversarial Networks. <i>arXiv preprint <a href="https://arxiv.org/abs/1805.08318">arXiv:1805.08318, 2019</a></i>  

[4] Ian J. Goodfellow, Jean Pouget-Abadie, Mehdi Mirza, Bing Xu, David Warde-Farley, Sherjil Ozair, Aaron Courville, Yoshua Bengio. Generative Adversarial Networks. <i>arXiv preprint  <a href="https://arxiv.org/abs/1406.2661">arXiv:1406.2661, 2014</a></i>

[5] Jonathan Long, Evan Shelhamer, Trevor Darrell. Fully Convolutional Networks for Semantic Segmentation. <i>arXiv preprint <a href="https://arxiv.org/abs/1411.4038">arXiv:1411.4038, 2015</a></i>

[6] Liang-Chieh Chen, George Papandreou, Florian Schroff, Hartwig Adam. Rethinking Atrous Convolution for Semantic Image Segmentation. <i>arXiv preprint <a href="https://arxiv.org/abs/1706.05587">arXiv:1706.05587, 2017 (v3)</a></i>

[5] Liang-Chieh Chen, George Papandreou, Iasonas Kokkinos, Kevin Murphy, Alan L. Yuille. DeepLab: Semantic Image Segmentation with Deep Convolutional Nets, Atrous Convolution, and Fully Connected CRFs. <i>arXiv preprint <a href="https://arxiv.org/abs/1606.00915">arXiv:1606.00915, 2017 (v2)</a></i>

[6] Takeru Miyato, Toshiki Kataoka, Masanori Koyama, Yuichi Yoshida. Spectral Normalization for Generative Adversarial Networks. <i>arXiv preprint <a href="https://arxiv.org/abs/1802.05957">arXiv:1802.05957, 2018</a></i>

[7] Wenzhe Shi, Jose Caballero, Ferenc Huszár, Johannes Totz, Andrew P. Aitken, Rob Bishop, Daniel Rueckert, Zehan Wang. Real-Time Single Image and Video Super-Resolution Using an Efficient Sub-Pixel Convolutional Neural Network. <i>arXiv preprint <a href="https://arxiv.org/abs/1609.05158">arXiv:1609.05158, 2016</a></i>

[8] Qi Mao, Hsin-Ying Lee, Hung-Yu Tseng, Siwei Ma, Ming-Hsuan Yang. Mode Seeking Generative Adversarial Networks for Diverse Image Synthesis. <i>arXiv preprint <a href="https://arxiv.org/abs/1903.05628">arXiv:1903.05628, 2019(v6)</a></i>  

[9] Lars Mescheder, Andreas Geiger, Sebastian Nowozin. Which Training Methods for GANs do actually Converge?. <i>arXiv preprint <a href="https://arxiv.org/abs/1801.04406">arXiv:1801.04406, 2018</a></i>  

[10] Ting-Chun Wang, Ming-Yu Liu, Jun-Yan Zhu, Andrew Tao, Jan Kautz, Bryan Catanzaro. High-Resolution Image Synthesis and Semantic Manipulation with Conditional GANs. <i>arXiv preprint <a href="https://arxiv.org/abs/1711.11585">arXiv:1711.11585, 2018</a></i>  

[11] Phillip Isola, Jun-Yan Zhu, Tinghui Zhou, Alexei A. Efros. Image-to-Image Translation with Conditional Adversarial Networks. <i>arXiv preprint <a href="https://arxiv.org/abs/1611.07004">arXiv:1611.07004, 2018</a></i>  
