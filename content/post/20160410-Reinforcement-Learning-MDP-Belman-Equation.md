+++
date = "2016-04-10T15:19:17+09:00"
draft = false
categories = ["まとめ"]
tags = ["Reinforcement Learning"]
title = "強化学習についてまとめる(1) MDPとベルマン方程式"

+++

強化学習について古典的なものからDeepNNを使ったものまでまとめていきたい。

# マルコフ決定過程
マルコフ決定過程(Markov Decision Process; MDP)は状態の遷移が確率的に起こり、マルコフ過程を満たす過程のことをいう。
MDPは状態$s$、行動$a$、遷移先の状態を$s'$、状態遷移確率$P(s'|s, a)$の組で表現される。
また、状態$s$において行動$a$を選択したとき、即時報酬$r(s, a, s')$が得られるとする。

とくに時間的な過程の進展を表すため、特に時刻$t$から$t+1$の状態の遷移について

$$
状態: s_t\\\\\
行動: a_t\\\\\
状態遷移確率: P(s\_{t+1}|s_t, a_t)\\\\\
報酬関数: r_t = r(s_t, a_t, s\_{t+1})
$$

を考える。

![マルコフ決定過程](/images/2017/11/MDP.png)

# 価値関数
時刻$t$において将来($t \rightarrow \infty $)にわたって得られる報酬について、割引累積報酬$G_t$を定義する。
\begin{equation}
G_t = \sum\_{k=0}^{\infty} \gamma ^k R\_{t+k+1}
\end{equation}
ここで、$R\_{t+1}$は$r(s_t, a_t, s\_{t+1})$の値とする。
また、$\gamma$は$0 \le \gamma < 1$の値で、遠い将来に得られるであろう報酬を低く見積もるために使う。

状態$s$において、行動$a$が選択される確率を$\pi = \pi(a | s)$とする。
この$\pi$を**方策**と呼ぶ。
ロボットで言うと次の状態をどう選ぶかの判断を行う部分である。

さて、ある方策$\pi$を採用したときの報酬がどの程度のものか見積もりたい。
方策$\pi$のもとで、以下のように関数$V^{\pi}$を定義する。
\begin{equation}
V^{\pi}(s) = \mathbb{E}[G_t|s_t=s] = \mathbb{E}\[R\_\{t+1\}\ + \gamma R\_\{t+2\} + \dots | s_t = s]
\label{eq:v_func}
\end{equation}
これを**状態価値関数**(あるいは単に**価値関数**)と呼ぶ。
期待値を取っているのは、方策$\pi$は確率的であるから、$s_t=s$となるのも確率的であるため、$s_t$について周辺化して評価したいためである。

同様に、状態だけでなく、行動についても条件として
\begin{equation}
Q^{\pi}(s,a) = \mathbb{E}[G_t|s_t=s, a_t=a] = \mathbb{E}\[R\_\{t+1\}\ + \gamma R\_\{t+2\} + \dots | s_t = s, a_t = a]
\label{eq:q_func}
\end{equation}
も考える。これを、**行動価値関数** と呼ぶ。

以上の枠組みにおいて、最も良い方策、**最適方策** を$\pi^{\*}$とし、
この方策を採用したときの価値関数を**最適行動価値関数**
\begin{equation}
Q^{\*}(s,a) = Q^{\pi^{\*}}(s,a) = \max\_{\pi} Q^{\pi}(s,a)
\end{equation}
とする。

## ベルマン方程式の導出
\eqref{eq:v_func}式において$\mathbb{E}[\*]$は線形の演算のため、

\begin{equation}
\begin{aligned}
V^{\pi}(s) &=  \mathbb{E}\[\sum\_{k=0}^{\infty} \gamma^k R\_{t+k+1} | s_t = s]\\\\\
&= \mathbb{E}\[R\_{t+1}\ | s_t = s] + \mathbb{E}\[\sum\_{k=1}^{\infty} \gamma^k R\_{t+k+1} | s_t = s]\\\\\
&= \mathbb{E}\[R\_{t+1}\ | s_t = s] + \gamma \mathbb{E}\[\sum\_{k=1}^{\infty} \gamma^{k-1} R\_{t+k+1} | s_t = s]
\end{aligned}
\label{eq:q_func_trans}
\end{equation}

とすることができる。

ここで、\eqref{eq:q_func_trans}式右辺第1項は

\begin{equation}
\begin{aligned}
\mathbb{E}\[R\_{t+1}\ | s_t = s]
=\sum\_{a \in A(s)} \pi(a|s) \sum\_{s' \in S} P(s'|s, a) r(s, a, s')
\end{aligned}
\label{eq:q_func_trans_1}
\end{equation}

となる。
$s_t=s$において行動$a$を取る確率が$\pi(a,|s)$、状態遷移して$s'$に移動する確率が$P(s'|s,a)$であるから、行動$a$を取って、状態$s'$に遷移する確率は$\pi(a,|s)P(s'|s,a)$である。
その上で、状態$s$において取れる行動の全集合$A(s)$と次に取れる全状態$S$について$r(s,a,s')$の期待値を取る、というのが上式で行われていることである。

次に、\eqref{eq:q_func_trans}式右辺第2項は
\begin{equation}
\begin{aligned}
\mathbb{E}\[\sum\_{k=1}^{\infty} \gamma^{k-1} R\_{t+k+1} | s_t = s]
&= \sum\_{a \in A(s)} \pi(a|s) \sum\_{s' \in S} P(s'|s,a) \mathbb{E}\[\sum\_{k=1}^{\infty} \gamma^{k-1} R\_{t+k+1} | s\_{t+1} = s']\\\\\
&= \sum\_{a \in A(s)} \pi(a|s) \sum\_{s' \in S} P(s'|s,a) \mathbb{E}\[\sum\_{k=0}^{\infty} \gamma^k R\_{(t+1)+k+1} | s\_{t+1} = s']\\\\\
&= \sum\_{a \in A(s)} \pi(a|s) \sum\_{s' \in S} P(s'|s,a) V^{\pi}(s')
\end{aligned}
\label{eq:q_func_trans_2}
\end{equation}

と変形できる。
以上\eqref{eq:q_func_trans}, \eqref{eq:q_func_trans_1}, \eqref{eq:q_func_trans_2} 式により、

\begin{equation}
\begin{aligned}
V^{\pi}(s) = \sum\_{a \in A(s)} \pi(a|s) \sum\_{s' \in S} P(s'|s, a) \left(r(s, a, s') + \gamma V^{\pi}(s') \right)
\end{aligned}
\label{eq:belman_value_func}
\end{equation}

が導出される。これを**ベルマン方程式**と呼ぶ。

また、$Q^{\pi}(s,a)$の定義より
\begin{equation}
\begin{aligned}
V^{\pi}(s) = \sum\_{a \in A(s)} \pi(a|s) Q^{\pi}(s,a)
\end{aligned}
\end{equation}

であるから、
\begin{equation}
\begin{aligned}
Q^{\pi}(s, a) &= \sum\_{s'} P(s'|s, a) \left(r(s, a, s') + \gamma V^{\pi}(s') \right)\\\\\
&= \sum\_{s'} P(s'|s, a) \left(r(s, a, s') + \gamma \sum\_{a \in A(s')} \pi(a'|s') Q^{\pi}(s',a') \right)
\end{aligned}
\end{equation}

と行動価値関数についてのベルマン方程式を導出できる。

<!-- 上記のベルマン方程式において、常に最適な方策を取るという前提を置けば、以下のベルマン最適方程式が

さて、このベルマン方程式を解けば、$Q^{\pi}$を計算できるのだが、上式を見ればわかるように、そのためには状態遷移確率が既知である必要がある。
扱いたいのは環境が未知の問題であるため、状態遷移確率を用いずに反復的に価値関数を求める。

## 価値反復を使ったアルゴリズム
状態遷移確率が未知であるため、実際に行動してみて、状態遷移確率分布からサンプリングすることで価値観数を求めていくのが、価値反復を使ったアルゴリズムである。
現在の値を目標値に少しずつ更新していく手法は、強化学習の文脈において問題を解くために重要になる。
そのようなアルゴリズムとしてSarsaとQ-Learningを取り上げる。

### Sarsa
時刻$t$において状態$s_t$であり$a_t$を行った結果、次の状態$s\_{t+1}$と報酬$r_t$を観測したとき、$s\_{t+1}$において行う予定の行動$a\_{t+1}$をもとに、以下の更新則でQ値を更新する。

\begin{equation}
Q(s_t, a_t)
\leftarrow
(1 - \alpha) Q(s_t, a_t) + \alpha (r\_{t+1} + \gamma Q(s\_{t+1}, a\_{t+1}))
\end{equation}

$\alpha(0 \le \alpha < 1)$は学習率と呼ばれるパラメータで、アルゴリズムの設計者が調整する。
このアルゴリズムは現在のQ値と$r\_{t+1} + \gamma Q(s\_{t+1}, a\_{t+1})$との内分によって、Q値を更新していく手法である。
先に述べたように、状態遷移確率を用いずに、行動によって発生する状態遷移の比率が、無限回試行していくごとに状態遷移確率に近づいていく性質を用いてQ値を求めている。

### Q-Learning
Sarsaと同様の設定で、以下の更新則でQ値を更新するのがQ-Learningである。

\begin{equation}
Q(s_t, a_t)
\leftarrow
(1 - \alpha) Q(s_t, a_t) + \alpha (r\_{t+1} + \gamma \max\_{a^{\prime} \in A} Q(s\_{t+1}, a^{\prime}))
\end{equation}

Sarsaと違って、次に何の行動を取ったかはQ値の更新には関わってこない。
$\max\_{a^{\prime} \in A} Q(s\_{t+1}, a^{\prime})$つまり、次の状態において取れる行動のうち、最大の価値を得られる行動(つまり最適方策$\pi^{\*}$)を採った時のQ値を使って更新する。
Sarsaが実際に取った方策に更新が依存するのに対して、Q-LearningではQ値は環境に対して一定の値に収束する。

このように実際に採用している方策と異なる方策を学習する方法は**方策オフ型**学習と呼ばれる。
一方で、Sarsaのような、学習した方策をそのときどき採用する方法を**方策オン**型学習と呼ぶ。


### 方策の決定方法
Q値が求まったら、最も高いQ値を持つ行動$a$を選択すれば、将来にわたって最大の報酬が得られることが期待できる。
つまり、行動$a^{\*}$は

$$
a^* = \arg\max(Q^{\*})
$$

となる方策を採用するということになる。
これはgreedy方策と呼ばれている。

さて、この方策では$Q^{\*}$がわかっていれば最適な方策となるが、強化学習では行動をしながら$Q$を更新していくので$Q^{\*}$は常に不明である。
探索をするために、ある程度ランダムな方策を取る必要がある。
そのように確率$\epsilon$でランダムな行動を取る方策を$\epsilon$-greedy方策という。 -->
