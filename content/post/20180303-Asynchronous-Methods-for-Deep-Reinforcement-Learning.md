---
title: "論文 Asynchronous Methods for Deep Reinforcement Learning"
date: 2018-03-03T16:28:53+09:00
draft: false
categories: ["論文読み"]
tags: ["Reinforcement Learning"]
---

Asynchronous Methods for Deep Reinforcement Learning を読んだ。

Arxivリンク https://arxiv.org/abs/1602.01783

Actor-Criticを並列に学習することによって、Atariのタスクにおいて好成績を出せるようになったというのがこの論文のcontribute。
いわゆるA3Cと呼ばれるアルゴリズムにあたる。

方策オン型の強化学習は、更新前後の方策が時間的に強く相関しているために、安定して収束しないというのは共通認識であった。
そこで、方策オフ型の手法とExperience Bufferを使って、相関を除去するという手法が取られていた。
しかし、このような手法は、方策オフ型の学習に限られてしまうし、経験の保持のために多くのメモリを必要としてしまう。

そこで、復数のエージェントを並列で走らせ、その結果を使って学習することで時間的相関を緩和する手法が考えられた。
これはSarsaやActor-Criticのような方策オン型学習にも、Q-Learningのような方策オフ型の手法にも、もちろんDeep Learningを使う方法にも適用できる考え方である。
この方法の副次的な利点として、従来は１台の強力なGPUを持ったコンピュータを使ってExperience Replayを行う必要があったが、分散的に学習できるようになったために、通常のCPUを持った復数のコンピューターがあれば性能の高い学習を実現することができる、ということも確認されている。

# 学習手法
コンポーネントとしては、複数のActor-Learnerがあり、そのパラメタを動悸する
Actor-Learner では数ステップ分の勾配を蓄積する。
これによって、Actor-Learnerがお互いにパラメタを上書きしてしまうのを防止する。
蓄積をどれだけするかというのは、計算量とデータ量のどちらを優先するかというトレードオフになる。
蓄積してバッチ的にパラメタを更新することで計算量を減らすことができるが、一方で学習のために必要なデータ量が増えるということである。

## 探索
各Actor-Learnerでの探索の方法は、それぞれ違っていたほうがロバスト性の高い方策を学習できるという。
この論文では、$\epsilon$-greedy 法において各Learnerで$\epsilon$をある分布からサンプルして決定することでLearnerごとの探索方法に差異をもたせた。

また、目的関数に方策$\pi$のエントロピーが用いられている。
目的関数にエントロピーの項を加え、正則化項として働かせることで、すぐに局所最適解に陥ってしまうのを防ぐ効果があることが、(Williams & Peng, 1991)によって確認されている。

## 実験
* 57 Atari games
* TORCS Car Racing Simulator
* Continuous Action Control Using the MuJoCo Physics Simulator
* Labyrinth

の課題を解いた。
16個のCPUを用いて、当時のSOTAを達成した。