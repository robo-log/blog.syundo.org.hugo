---
title: "論文 Asynchronous Methods for Deep Reinforcement Learning"
date: 2018-03-03T16:28:53+09:00
draft: true
---

Asynchronous Methods for Deep Reinforcement Learning
https://arxiv.org/abs/1602.01783

# abstract
<!-- Actor-Criticを並列に学習することによって、Atariのタスクにおいて好成績を出せるようになった。
連続行動空間への適用については3次元の迷路の探索タスクで検証できた。 -->

方策オン型の強化学習は、更新前後の方策が時間的に強く相関しているために安定して収束しないというのは共通認識であった。
そこで、方策オフ型の手法とExperience Bufferを使って、相関を除去するという手法が取られていた。
しかし、このようなやり方は、方策オフ型の学習に限られてしまうし、経験の保持のために多くのメモリを必要としてしまう。

そこで、Experience Bufferを使うのではなく復数のエージェントを並列で走らせ、その結果を使って学習することで時間的相関を緩和する手法が考えられた。
これはSarsaやActor-Criticのような方策オン型学習にも、Q-Learningのような方策オフ型の手法にも、もちろんDeep Learningを使う方法にも適用できる考え方である。
この方法の副次的な利点として、従来は１台の強力なGPUを持ったコンピュータをつかってExperience Replayを行う必要があったが、分散的に学習できるようになったために、通常のCPUを持った復数のコンピューターがあれば性能の高い学習を実現することができる、ということも確認されている。

First, we ob- tain a reduction in training time that is roughly linear in the number of parallel actor-learners. Second, since we no longer rely on experience replay for stabilizing learning we are able to use on-policy reinforcement learning methods such as Sarsa and actor-critic to train neural networks in a stable way.


In this paper we provide a very different paradigm for deep reinforcement learning.
Instead of experience replay, we asynchronously execute multiple agents in parallel, on mul- tiple instances of the environment.

手法
Actor-Learner では数ステップ分の勾配を蓄積する。
これによって、Actor-Learnerがお互いにパラメタを上書きしてしまうのを防止する。
蓄積をどれだけするかというのは、計算量とデータ量のどちらを優先するかというトレードオフになる。
蓄積してバッチ的にパラメタを更新することで計算量を減らすことができるが、一方で学習のために必要なデータ量が増えるということである。

探索
各Actor-Learnerでの探索の方法は、それぞれ違っていたほうがロバスト性の高い方策を学習できるという。
この論文では、$\epsilon$-greedy 法において各Learnerで$\epsilon$をある分布からサンプルして決定することでLearnerごとの探索方法に差異をもたせた。

We also found that adding the entropy of the policy π to the objective function improved exploration by discouraging premature convergence to suboptimal deterministic poli- cies. This

目的関数にエントロピーの項を加え、正則化項として働かせることで、すぐに局所最適解に陥ってしまうのを防ぐ効果があることが、(Williams & Peng, 1991)によって確認されていた。
この論文でも

実験
* 57 Atari games
* TORCS Car Racing Simulator
* Continuous Action Control Using the MuJoCo Physics Simulator
* Labyrinth

をやった。
結論s
* Our results show that in our proposed framework stable training of neural networks through reinforcement learning is possible with both value- based and policy-based methods, off-policy as well as on- policy methods, and in discrete as well as continuous do- mains. When trained on the Atari domain using 16 CPU cores, the proposed asynchronous algorithms train faster than DQN trained on an Nvidia K40 GPU, with A3C sur- passing the current state-of-the-art in half the training time.
One

* One of our main findings is that using parallel actor- learners to update a shared model had a stabilizing effect on the learning process of the three value-based methods we considered. While this shows that stable online Q-learning is possible without experience replay, which was used for this purpose in DQN, it does not mean that experience re- play is not useful. Incorporating experience replay into the asynchronous rein