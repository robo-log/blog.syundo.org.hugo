+++
date = "2017-11-10T19:54:59+09:00"
draft = false
title = "強化学習についてまとめる(2) 反復による価値の推定"

+++

[前回](../20160410-Reinforcement-Learning-MDP-Belman-Equation)はMDPとベルマン方程式について扱った。
ベルマン方程式を解くことができれば、$Q^{\pi}$を計算できるのだが、どう計算するのか、価値関数からどのように方策を決定するのかという問題がある。
今回は、多数のデータを使って反復的に計算することでこれを求める方法について扱いたい。
<!-- 上式を見ればわかるように、そのためには状態遷移確率が既知である必要がある。
扱いたいのは環境が未知の問題であるため、状態遷移確率を用いずに反復的に価値関数を求める。 -->

# 価値推定と反復
価値関数についてのベルマン方程式において、常に最適な方策を取るという前提を置けば、以下の**最適ベルマン方程式**を定めることができる。
\begin{equation}
\begin{aligned}
V^{\pi}(s) = \max\_{a \in A} \sum\_{s' \in S} P(s'|s, a) \left(r(s, a, s') + \gamma V^{\pi}(s') \right)
\end{aligned}
\label{eq:belman_value_func_max}
\end{equation}

\begin{equation}
\begin{aligned}
Q^{\pi}(s, a) = \sum\_{s'} P(s'|s, a) \left(r(s, a, s') + \gamma \max\_{a' \in A} Q^{\*}(s',a') \right)
\end{aligned}
\label{eq:belman_q_func_max}
\end{equation}

あるいは、取り得る方策が確率的でない、常に方策が決まっている定常方策を取るとすると、以下のようになる。
\begin{equation}
\begin{aligned}
V^{\pi}(s) =  \sum\_{s' \in S} P(s'|s, a) \left(r(s, a, s') + \gamma V^{\pi}(s') \right)
\end{aligned}
\label{eq:belman_value_func_const}
\end{equation}

\begin{equation}
\begin{aligned}
Q^{\pi}(s, a) = \sum\_{s'} P(s'|s, a) \left(r(s, a, s') + \gamma Q^{\pi}(s',a') \right)
\end{aligned}
\label{eq:belman_q_func_const}
\end{equation}

以上の性質は価値関数を逐次的に更新していくことで求めていくために重要になる。
また、その価値関数に従って最適な方策を決定し行動していくことで逐次的に方策を更新していくこともできる。
前者を価値反復、後者を方策反復と呼ぶ。
このとき、方策の決定方法としては以下のようなgreedy方策が考えられる。
$$
a^* = \arg\max(Q^{\*})
$$

また、実際に採用している方策と異なる方策を学習する方法は**方策オフ型**学習と呼ばれる。
一方で、学習した方策をそのときどき採用する方法を**方策オン**型学習と呼ぶ。

## TD学習
MDPにおいて価値関数を推定したい。
TD学習(Temporal difference learning)は、現在の推定値を学習中の目標値として使用することで、問題を解いていく手法である。

ベルマン方程式は、状態遷移確率が未知の場合、そのまま解くことはできない。
そこで、状態遷移確率は実際に観測する(サンプリングする)ことによって、確率分布を近似し、ベルマン方程式を扱う。

### TD(0)法
以下のように価値関数の推定値を更新していく手法がTD(0)法である。
\begin{equation}
V\_{t+1}(s_t)
\leftarrow
(1 - \alpha_t) V_t(s_t) + \alpha_t (R\_{t+1} + \gamma V_t(s\_{t+1}))
\end{equation}

現在の値$V_t(s_t)$と目標値$R\_{t+1} + \gamma V_t(s\_{t+1})$との内分によって推定値を更新していくのがこの手法である。
**TD** 誤差として$\delta_t$を以下のように定義することで、TD誤差を小さくする方向に更新するアルゴリズムとして捉えることもできる。

\begin{equation}
\delta_t = (R\_{t+1} + \gamma V_t(s\_{t+1})) - V_t(s_t)\\\\\
V\_{t+1}(s_t)
\leftarrow
V_t(s_t) + \alpha_t \delta_t
\end{equation}

$\delta_t$は目標値と現在の値の差分であるから、この立式のほうがわかりやすいかもしれない。

### Sarsa
TD(0)法を行動価値数に拡張したものが、Sarsaである。

時刻$t$において状態$s_t$であり$a_t$を行った結果、次の状態$s\_{t+1}$と報酬$r_t$を観測したとき、$s\_{t+1}$において行う予定の行動$a\_{t+1}$をもとに、以下の更新則でQ値を更新する。

\begin{equation}
Q(s_t, a_t)
\leftarrow
(1 - \alpha) Q(s_t, a_t) + \alpha (r\_{t+1} + \gamma Q(s\_{t+1}, a\_{t+1}))
\end{equation}

### Q-Learning
最適ベルマン方程式を解くためにで、以下の更新則でQ値を更新するのがQ-Learningである。

\begin{equation}
Q(s_t, a_t)
\leftarrow
(1 - \alpha) Q(s_t, a_t) + \alpha (r\_{t+1} + \gamma \max\_{a^{\prime} \in A} Q(s\_{t+1}, a^{\prime}))
\end{equation}

Sarsaと違って、次に何の行動を取ったかはQ値の更新には関わってこない。
$\max\_{a^{\prime} \in A} Q(s\_{t+1}, a^{\prime})$つまり、次の状態において取れる行動のうち、最大の価値を得られる行動(つまり最適方策$\pi^{\*}$)を採った時のQ値を使って更新する。
Sarsaが実際に取った方策に更新が依存するのに対して、Q-LearningではQ値は環境に対して一定の値に収束する、方策オフ型学習である。

<!-- ### 方策の決定方法
Q値が求まったら、最も高いQ値を持つ行動$a$を選択すれば、将来にわたって最大の報酬が得られることが期待できる。
つまり、行動$a^{\*}$は


となる方策を採用するということになる。
これはgreedy方策と呼ばれている。

さて、この方策では$Q^{\*}$がわかっていれば最適な方策となるが、強化学習では行動をしながら$Q$を更新していくので$Q^{\*}$は常に不明である。
探索をするために、ある程度ランダムな方策を取る必要がある。
そのように確率$\epsilon$でランダムな行動を取る方策を$\epsilon$-greedy方策という。 -->
