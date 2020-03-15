---
title: 20200204 Rl Mtg Rl in Field
date: 2020-02-04T19:10:22+09:00
lastmod: 2020-02-04T19:10:22+09:00
cover: "/images/default1.jpg"
draft: true
categories: ["category1"]
tags: ["tag1", "tag2"]
description: 
---
安本雅啓さん
アラヤ

Reinforcement Learning for Real Life ICML 2019
June 14, 2019

# Challenges of Real-World Reinforcement Learning
Google Research, Brain Team, DeepMind
9個の課題を上げて、それに対する改善のmetricsを述べた

1. オフラインバッチ、オフポリシー学習
off-policy evaluation
* importance sampling
* direct method
既存のデータで報酬の予測器を作る => 新しいので評価

* doubly-robust estimators
充填サンプリング
* MAGIC
* More robust doubly robust off-policy

2. サンプル効率の問題
* Model Agnostic Meta-Learning (MAML)
* bootstrap DQN
Q-Netのensanble
* エキスパートデータを用いたbootstrap
Deep Q-Learning from Demonstrations

* モデルベース手法

3. 連続、高次元の状態行動空間
* Deep RL in Large Discrete Action Spaces
* Learn What not to Learn
* Deep RL with a natural language action space

4. Safety constraints
* CMPD (constrained MDP)
* budgeted MDP

5. 部分観測性、動的システム
* 履歴を用いる
* RNN
* sim2real
	* robust mdp
	* domain randomization
		* sim-to-real transfer of robotic control with dynamics randomization
			* paramを分布から取得
			* 観測にもノイズを入れる
	* system identification
		* like MAML

6. 非明示的、複数目的最適の報酬関数
期待値で良くなく、全てよく動く必要がある

* conditional value at risk (CVaR) objective
* distributional DQN
	* 報酬の分布をモデル化
* inverse RL
	* inverse reward design
* hybrid reward architecture for reinforcement learning

7. 説明可能性
* programmatically interpretable reinforcement learning
	* NNのポリシーを探索によって明示的なdomain-specificプログラミング言語に蒸留する。

8. realtime inference
* a real-time model-based reinforcement learning architecture for robot control

9. system delay
* texplore: real-time sample-efficient reinforcement learning
* optimizing agent behavior over long time scales by transporting value
* RUDDER Return Decomposition for Delayed Rewards

評価環境作った
DeepMind control suite

# Park: An Open platform for Learning augmented computer systems
* ネットワークの輻輳制御
* ビデオストリーミングにおけるビットレート最適化
* jobのアサイン

# Horizon: Facebook's Open Source Applied Reinforcement learning platform
Horizon => ReAgent

