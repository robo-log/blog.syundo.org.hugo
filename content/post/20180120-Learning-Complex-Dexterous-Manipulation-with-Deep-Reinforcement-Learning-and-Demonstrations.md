---
title: "論文 Learning Complex Dexterous Manipulation With Deep Reinforcement Learning and Demonstrations"
date: 2018-01-20T20:15:22+09:00
draft: false
categories: ["論文読み"]
tags: ["Reinforcement Learning", "Imitation Learning"]
---

[Learning Complex Dexterous Manipulation With Deep Reinforcement Learning and Demonstrations](https://sites.google.com/view/deeprl-dexterous-manipulation)

[arxiv](https://arxiv.org/abs/1709.10087)を読む。

以下の動画のようにハンドを使った複雑なタスクの学習を少ない学習ステップ数で実現している。
人間のデモンストレーションを学習に用いることで、他手法と比べて人間らしい動作になっている。
例えば変に指先が曲がったりしていない。

<iframe width="560" height="315" src="https://www.youtube.com/embed/jJtBll8l_OM" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

この論文では、

1. 自然方策勾配法を使って、方策の学習をしている
2. 方策をbehavior closingを使って事前学習している
3. RLのプロセスにおける方策の更新でも人間によるデモとの類似を使っている

behavior closingはimitation learningの一種の方法で、人間によるデモを再現できるように方策を教師あり学習することである。

事前学習が終わったらMDPにおける学習を始める。
方策勾配は以下のようにする。

\begin{equation}
g\_{aug} =
\sum\_{(s, a) \in \rho\_{\pi}} 
\nabla\_\{\theta\} \log \pi\_\{\theta\} (a | s) A^{\pi}(s, a) +
\sum\_{(s, a) \in \rho_D} 
\nabla\_\{\theta\} \log \pi\_\{\theta\} (a | s) w(s, a)
\end{equation}

ここで、$rho\_{\pi}$はMDPにおいて得られたデータセット、$\rho_D$は事前学習に用いたデータセットである。
$w(s, a)$は重みづけ関数であり、スケール的には
$w(s, a) = A^{\pi}(s,a) \hspace{5mm} \forall (s, a) \in \rho_D$を用いるのが妥当だが、計算できないので、ヒューリスティック的に以下を用いる。

\begin{equation}
w(s, a) =
\lambda_0 \lambda_1^k
\max\_{(s', a') \in \rho\_{\pi}}
A^{\pi}(s', a')
\hspace{5mm}
\forall (s, a) \in \rho_D
\end{equation}

ここで、$\lambda_0, \lambda_1$はハイパーパラメータ、$k$は学習ステップ数であり、この$k$の項によって、学習初期はデモデータを重視するように制御している。

以上の手法によって、24自由度のハンドを使って、シミュレーション上ではあるがロボットの実時間にして数時間以内にタスクを達成することができている。
