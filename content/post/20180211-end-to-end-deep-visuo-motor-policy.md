---
title: "End to End Deep Visuo Motor Policy"
date: 2018-02-11T19:02:26+09:00
draft: true
---


Policy search methods can allow robots to learn control policies for a wide range of
tasks, but practical applications of policy search often require hand-engineered components for perception, state estimation, and low-level control. In this paper, we aim to answer the following question: does training the perception and control systems jointly end-to- end provide better performance than training each component separately? To this end, we develop a method that can be used to learn policies that map raw image observations directly to torques at the robot’s motors. The policies are represented by deep convolutional neural networks (CNNs) with 92,000 parameters, and are trained using a guided policy search method, which transforms policy search into supervised learning, with supervision provided by a simple trajectory-centric reinforcement learning method. We evaluate our method on a range of real-world manipulation tasks that require close coordination between vision and control, such as screwing a cap onto a bottle, and present simulated comparisons to a range of prior policy search methods.

方策学習は様々なタスクを学習できるが、実用的には認識、状態推定、低レベルの制御についてハンドエンジニアリング的に用意されたものが必要となる。
この論文では、認識から制御までをまとめて学習することで、それぞれを別々に学習した場合に比べて良いパフォーマンスを実現できるか、という問題に取り組んでいる。
入力画像からモーターのトルク出力までを復数のCNNで表現して、計92000個のパラメタを学習する。
Guided policy searchを使うことで、方策学習を教師あり学習において行う工夫もされている。

that separates the problem of learning visuomotor policies into separate supervised learning and trajectory learning phases, each of which is easier than optimizing the policy directly. We also discuss a policy architecture suitable for end-to-end learning of vision and control, and a training setup that allows our method to be applied to real robotic platforms.
3.1

