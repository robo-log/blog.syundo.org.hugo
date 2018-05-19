---
title: "2017 12 07 Ros Learn Rl"
date: 2017-12-07T19:05:18+09:00
draft: true
---

12/7 に ROS Japan UG #19 機械学習・AI勉強会 に参加してきた
予定: https://rosjp.connpass.com/event/72287/

# 実社会・実環境におけるロボットの機械学習 by PFN 高橋さん
## ロボット制御
### 従来のロボット制御
* モデルがある、=> 認識 => 軌道計画
* amazon picking challenge

しかし身体と知能は不可分
従来の例: サブサンプションアーキテクチャ

### これからのロボット
強化学習とロボット
現在の問題
* 方策を獲得するまでの試行回数の多さ
	-> 実際のロボットで行うと時間がかかる
* シミュレーションと実機との差異
* 道な環境への対応

#### 試行回数を減らす工夫
模倣学習
* 課題
	* 成功例から逸脱するとうまくいかなくなる
	* ほかの環境でうまくいかなくなる
		* 光の加減でだめ

		domain randomization deep neural networks from simulation to the real world iros2017

## 実機とシミュレーターの組み合わせ
Google brain

教師あり + 強化学習

## 未知な環境への即時対応能力
map-based multi-policy reinforcement learning (PFN)

# インテック @sojun さん
出力数を大きくすると、むしろ精度が上がる部分がある。
不純物が精度を高める効果があるのでは

# ROBOTIS柴田さん

# TensorFlow ObjectDetectionAPIを使った取り組み@つくばチャレンジ
有松さん
Project C.G.S. つくばロボットチャレンジに参加

Camera + LIDAR

SSDを転移学習

TensorFlow Object Detection API
ロボット実機では数ヘルツ

# SenseTimeのかた
Googleの論文紹介

どうsルルカ
domain adaptation(DA)
* 出力層
* 入力層

# ROS,GazeboとChainerを用いた畳込みニューラルネットワークによる３次元形状の学習
* PCLで前処理
\/
* 実で～あ背収拾は難しい

voxel

# hashimoto

ライブラリ、実機

課題
報酬
