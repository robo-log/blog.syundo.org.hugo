+++
date = "2017-11-17T03:15:08+09:00"
draft = false
categories = ["まとめ"]
tags = ["Reinforcement Learning"]
title = "強化学習についてまとめる(3) 方策勾配"

+++

<!-- 強化学習について古典的なものからDeepNNを使ったものまでまとめていきたい。
<!-- ![](/images/2016/04/lis.gif) -->
<!-- [Deep RL Bootcamp](https://sites.google.com/view/deep-rl-bootcamp/lectures)を土台にして他文献を参照しながら構成していく。
取り上げる順番はこだわらず、自分のわかりやすい方法を採る。
# ロードマップ  -->
[前回、](../20171110-Reinforcement-Value-Policy-Iteration)
[前々回](../20160410-Reinforcement-Learning-MDP-Belman-Equation)
では、価値関数を求め、それを基に行動を決定する手法について扱ってきた。
しかし、そもそもロボットの行動を決めるのは方策であるのだから、その方策を直接学習できないのだろうか？という疑問が湧く。

今回は、前回までとは全く違うアプローチとして、**方策勾配法** をまとめる。
方策勾配法は、方策をあるパラメタで表される関数とし、そのパラメタを学習することで、直接方策を学習していくアプローチである。

方策を直接扱うことで

* $V^{\pi}$や$Q^{\pi}$を求めるような複雑でメモリを消費する手法を使わなくて良い
* 連続空間における行動を扱いやすくなる

などの利点がある。

方策勾配法においては、確率的な方策を扱う場合と確定的な方策を扱う場合がある。本記事では方策は確率的なものを扱う。(これは確定的な方策の一般化したものであるため)。
また後述するように、価値関数を最大化するように方策の勾配を求めたいが、そのときに方策(のlogを取ったもの)と報酬をかけ合わせたものの比率(これをここではscore functionと呼ぶ)を用いる方法、いわゆるlikelihood ratio methodあるいはscore function methodというものと、状態遷移パスの各点において勾配を計算することで直接価値関数の勾配を求める方法、いわゆるpathwise derivative methodと呼ばれ、DPG、SVGなどがこれにあたるものがある。
本記事では前者について述べている。

# 方策のモデルと勾配
$\theta$でパラメタライズされた確率的な方策$\pi\_{\theta}$を求める問題を考える。
$\tau$をステップ$0$から$H$までの状態-行動の系列(状態-行動空間でのパス)$\tau=(s_0, a_0, \dots, s_H, a_H)$としたとき、方策の評価関数として以下を考える。

\begin{equation}
\begin{aligned}
J(\theta) = \mathbb{E}\_{\pi\_{\theta}} \[\sum\_{t=0}^H R(s_t, a_t)\] \\\\\
= \sum\_{\tau} P(\tau ; \theta) R(\tau)
\end{aligned}
\end{equation}
ここで、$R(\tau) = \sum\_{t=0}^H R(s_t, a_t)$としている。
また$P(\tau ; \theta)$はパスの生成モデルであり、定義より
\begin{equation}
P(\tau ; \theta) = \prod\_{t=0}^H P(s\_{t+1} | s_t, a_t) \pi\_{\theta}(a_t | s_t)
\label{eq:p_tau_theta}
\end{equation}
である。

以上の設定において、方策の学習は最終的に
\begin{equation}
\max\_{\theta} J(\theta) = \max\_{\theta} \sum\_{\tau} P(\tau ; \theta) R(\tau)
\end{equation}
を求める問題となる。
そこで、微小ステップごとに評価関数の$\theta$での微分方向
\begin{equation}
\nabla\_{\theta} J(\theta) = \nabla\_{\theta} \sum\_{\tau} P(\tau ; \theta) R(\tau)
\end{equation}
に方策を更新することで、方策を最適化することを考える。
これは以下のように変形できる。

\begin{equation}
\begin{aligned}
\nabla\_{\theta} J(\theta) &= \nabla\_{\theta} \sum\_{\tau} P(\tau ; \theta) R(\tau) \\\\\
&= \sum\_{\tau} \nabla\_{\theta} P(\tau ; \theta) R(\tau) \\\\\
&= \sum\_{\tau} \frac{P(\tau ; \theta)}{P(\tau ; \theta)} \nabla\_{\theta} P(\tau ; \theta) R(\tau) \\\\\
&= \sum\_{\tau} P(\tau ; \theta) \frac{\nabla\_{\theta} P(\tau ; \theta)}{P(\tau ; \theta)} R(\tau) \\\\\
&= \sum\_{\tau} P(\tau ; \theta) \nabla\_{\theta} \log P(\tau ; \theta) R(\tau) \\\\\
&= \mathbb{E}\_{\pi\_{\theta}}\[\nabla\_{\theta} \log P(\tau ; \theta) R(s_t, a_t)\] \\\\\
\end{aligned}
\end{equation}

この式を見ると、更新則によって、

* 報酬$R$が高いパスの存在確率を上げる
* 報酬$R$が低いパスの存在確率を下げる

方向に方策が更新されることがわかる。
これをパスではなく方策と遷移確率について立式するとどうなるだろうか。
式\eqref{eq:p_tau_theta}より、

\begin{equation}
\begin{aligned}
\nabla\_{\theta} \log P(\tau ; \theta)
&= \nabla\_{\theta} \log
\left[ \prod\_{t=0}^H P(s\_{t+1} | s_t, a_t) \pi\_{\theta}(a_t | s_t) \right] \\\\\
&= \nabla\_\{\theta\} \left\[ \sum\_\{t=0\}^H \log P(s\_\{t+1\} | s_t, a_t) \right\] +
\nabla\_\{\theta\} \left\[ \sum\_\{t=0\}^H \log \pi\_{\theta} (a_t | s_t) \right\] \\\\\
&= \nabla\_\{\theta\} \sum\_\{t=0\}^H \log \pi\_{\theta} (a_t | s_t) \\\\\
&= \sum\_\{t=0\}^H \nabla\_\{\theta\} \log \pi\_{\theta} (a_t | s_t)
\end{aligned}
\end{equation}

となる。状態遷移確率$P(s\_\{t+1\} | s_t, a_t)$は$\theta$をパラメタとして持たないので、$\theta$で微分するとこの項が消えるのである。

結局、勾配は
\begin{equation}
\begin{aligned}
\hat{g} = \mathbb{E}\_{\pi\_{\theta}}\[\nabla\_{\theta} \log P(\tau ; \theta) R(s_t, a_t)\] \\\\\
where \hspace{15pt}
\nabla\_{\theta} \log P(\tau ; \theta) =
\sum\_\{t=0\}^H \nabla\_\{\theta\} \log \pi\_\theta (a_t | s_t)
\end{aligned}
\end{equation}

すなわち
\begin{equation}
\hat{g} = \mathbb{E}\_{\pi\_\theta}\[\sum\_\{t=0\}^H \nabla\_\{\theta\} \log \pi\_\{\theta\} (a_t | s_t) R(s_t, a_t)\]
\label{eq:policy_gradient}
\end{equation}

で表され、方策の$\theta$での勾配のみで表現できることがわかる。

## Baseline
式\eqref{eq:policy_gradient}にbaseline $b$という値を追加する。
\begin{equation}
\hat{g} = \mathbb{E}\_{\pi\_\theta}\[\sum\_\{t=0\}^H \nabla\_\{\theta\} \log \pi\_\{\theta\} (a_t | s_t) (R(s_t, a_t) - b)\]
\label{eq:base_lined_policy_gradient}
\end{equation}

勾配の計算に$H$ステップの加算を含むため、分散は増大しやすくなる。
baselineは$\hat{g}$の値には影響を与えないが、調整前に比べて$\hat{g}$の分散を減少させる効果がある。

$R(s_t, a_t) - b$が小さいほど分散が小さくなるので、$b$の決定法としては、$R(s_t, a_t)$との二乗距離を小さくするように調整すればよい。

## 行動価値関数 $Q(s_t, a_t)$ との関係
式\eqref{eq:policy_gradient}では、$H$ステップの累積を想定して立式したが、$H \rightarrow \infty$としても同様に成り立つ。
また、報酬の加算についても割引報酬を用いても同様に導出される。

\begin{equation}
\begin{aligned}
\hat{g} &= \mathbb{E}\_{\pi\_\theta}\[\sum\_\{t=0\}^{\infty} \nabla\_\{\theta\} \log \pi\_\{\theta\} (a_t | s_t) \gamma^t r(s_t, a_t)\] \\\\\
&= \mathbb{E}\_{\pi\_\theta}\[\sum\_\{t=0\}^{\infty} \nabla\_\{\theta\} \log \pi\_\{\theta\} (a_t | s_t) 
\[\sum\_\{k=0\}^{t-1} \gamma^k r(s_k, a_k) + \sum\_\{k=t\}^{\infty} \gamma^k r(s_k, a_k)\]\] \\\\\
\label{eq:policy_gradient_q_func}
\end{aligned}
\end{equation}

ここで、$\sum\_\{k=0\}^{t-1} \gamma^k r(s_k, a_k)$の項は、時刻$t$においての行動$a_t$の影響を受けない部分であるため、除去する。
これによって分散を増やす要素を削減することができる。

すると、行動価値関数を用いて以下のように表すことができる。

\begin{equation}
\begin{aligned}
\hat{g} &= \mathbb{E}\_{\pi\_\theta}\[\sum\_\{t=0\}^{\infty} \nabla\_\{\theta\} \log \pi\_\{\theta\} (a_t | s_t) 
\sum\_\{k=t\}^{\infty} \gamma^k r(s_k, a_k)\] \\\\\
&= \mathbb{E}\_{\pi\_\theta}\[\sum\_\{t=0\}^{\infty} \nabla\_\{\theta\} \log \pi\_\{\theta\}(a_t | s_t)\]
Q(s, a) \\\\\
\end{aligned}
\end{equation}

このように行動価値関数に方策のlog微分をかけ合わせたものが、勾配の不偏推定量であることは方策勾配定理として知られている。