---
title: 3D 勉強会 Deep SLAM
date: 2018-09-29T13:18:20+09:00
lastmod: 2018-09-29T13:18:20+09:00
cover: "/images/default1.jpg"
draft: true
categories: ["category1"]
tags: ["tag1", "tag2"]
description: 
---

* CNNを使って単眼画像からDepth画像をregressionするシステム
Laina16, Eigen15

motin-stereoを使っているのではなく、perspective, semantics
monoculuar slamと全く違う

### CNN Depthの特徴とは
* monocular slam
  * 特徴点があるところで性格
  * 無いとregressionできない

* CNN depth prediction
  * depth mapをestimateでっきる
  * ブラーがかかったようになってしまう
    * 細かい情報が消える

これらを相補的に使えば良くなるのではないか
* Depth prediction => [refine] => refinded depth
  * refine: short baseline stereo

baseline stereoはmatchingの確からしさを求めることができるので、その結果をCNNでの推定値にfusionする

focul length
ほとんど動かないときにすごく遠くのものの位置は推定できない

* segmentationを使って推定するので、天井、床などとともに、周囲を見ているような相対的な状態が取れることが重要
  * 広い視野があることが重要。前天球カメラを使うのが良い

### 全天球カメラ
360度画像のground truethをどうやって用意するのか
=> 大変

ground truethなしに学習できないか?
平面画像でtrainしてCNNを全天球画像でも使えればいいじゃないか
* pollerのとこの歪みが強いんじゃ (千鳥)

convolutionを歪めてやればいいじゃないか!
歪み補正のフィルタをCNNに入力前にかければ、レクタングル画像になったもので推定できるじゃん

### incremental sene understanding
InSeg
CNN-SLAM
InSegReq

### 結果
* 絶対位置誤差
  * 0.2m
* LSD-SLAMと比べて
  * 

Q & A
* どのくらいの精度?
  * あとで
* 再構成とmotoion推定どちらが主眼?
  * どっちも
* Deep NNをどの部分に使うといいのか?
  * Depthの推定をdeepNN
    * domain依存が強い。似通っていないとだめ。ドールハウス問題も生じる
      * ドールハウス問題 ... ドールハウスを見ると完全にスケールを間違える
    * どれだけgeneralizationできるかというのが問題
  * 法線を推定する
    * 距離よりもgeneral
  * ドメイン依存のない情報をdeepで扱うのが肝ではないかと考えている
* Deep-SLAMを携帯電話に実装するみたいな話は?
  * niantecで実際に実装されている
* end-to-endでのSLAM推定について、今後やられていく?
  * SLAMについて、いかにdomain依存の問題にならないかというのが問題


# 金子真也さん
https://www.slideshare.net/MasayaKaneko/neural-scene-representation-and-rendering-33d
Neural sene representation and rendering

## どのようなものか？
Generative Query Network (GQN)
SLAMがやっているlocalizationとはアプローチが違う。指定した視点での画像を生成するというもの。
SfM/SLAMでもできることではある。関連研究として挙げられている。

* x 地図の可視性
* x localization
できないけど、生成ができる

* point cloud表現は人間が空間を把握するのにはわかりやすいが、CNNには扱いにくい
  * 隣接関係がわかりにくいので、畳込みが容易ではない
  * 画像に対するconvolutionに比べて決定的な手法はない印象
  * 皆川さんのスライドがわかりやすい
    * cvpr2018pointcloudcnnsplatnet

点群の畳込みが不要なので、
計算量が増大しない

やりたいこと
* 目標1: マップを表現するような、embeddingをVAEを使って求め、再構成したい。
* 目標2: embeddingを適切に扱いたい。画像が違っても同じように扱えるようにしたい。
  * Conditional VAE
    * 数枚の別視点の画像、視点を条件として

DRAW ... VAEをRNNを使うことで自己組織的に構成

## 実験結果
* scene representation
  * GQNではシーンの3次元構造に基づくembeddingが得られている

## GQNの関連研究
Consistent GQN
* SLIM
  * 
* Reversed-GQN
  * renderingでなくlocalization

## 質問
* なぜ数枚の画像を与えるだけでそんなにうまくいくのか?
  * 似た画像をさんざん見せて学習しているから
  * 逆に、まったく同じものを見たことなくても、経験的な構造を用いて再構成しているというのが肝
    * だから
  
# SfM Learner: Unsupervised Learning of Depth and Ego-Motion from Video
* 入力: 嘆願画像
* 出呂: ego-motion(6軸)とdepth画像

# Vid2Depth
3D-lossを導入した
* SSIM lossも追加した

震度推定、optical fllow推定は

* depthとflowの間には強い相関がある
  * depth と flowを同時に教師無しで学習する
* 

# DeepTAM

## 関連研究
* DTAM
  * 古典的SLAM
    * keyframeを使いつつ、密なdepth mapを推定しながらトラッキング
* DeMoN
  * 学習ベースSLAM

## 概要
ポーズ遷移をdeepNNで求める
f(keyframe, currentImage)
keyframe = (image, depth)

# Active Neural Localization
慶応大M1, 青木研究室
産総研RA
Graffity, inc

Global Localization
* Markov Localization
* Active Markov Localization
次の行動を決める

# Searching Toward Pareto-Optimal Device-Aware Neural Architectures (arXiv), MnasNet: Platform-Aware Neural Architecture Search for Mobile

* CNNの構造最適化手法の概要
* 性能と処理速度を考慮した構造最適化手法の

おおまかな内訳
* 探索範囲
* 探索アルゴリズム
* 高速化テクニック

* アーキテクチャ探索
  * CIFFAR-10でベンチマーク

GPU days

## GAを使った方法
### Large scale Evolution
  * GAを用いた手法
  * GPU 250台

## RLを使った方法
### Neural Architecture Search (NAS)
RAINFORCEを利用
### NasNet
RAINFORCEを利用

## SMBOによる構造最適化
GA

## タイトルの論文の説明
パレート最適の集合を求める

# BA-Net: Dense Bundle Adjustment Network
BAについては... bundle adjustment as modern system (鉄板)

Levenvereg-Marquardt method
gauss-newton