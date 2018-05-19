+++
date = "2017-12-02T16:41:14+09:00"
draft = false
title = "強化学習についてまとめる(4) 方策勾配に基づくアルゴリズム、Actor-Critic"

+++

[前回](../20171117-Reinforcement-Learning-Policy-Gradient)
は方策の勾配の求め方、勾配の分散を減少させるための、baselineを導入するところまでまとめた。
今回は、方策勾配を用いて方策を更新する実際のアルゴリズムについて扱う。

# vanilla policy gradient
baselineの調整と方策の更新を逐次的に更新していくという、方策勾配法の基本的な方法に則った基本的なアルゴリズムを、vanilla policy gradient method と呼ぶことにする。
vanilla policy gradient methodの擬似コードは以下のようになる。

1. パラメタ $\theta$, ベースライン$b$の初期化
1. $for$ $i=1,2,\dots$ $do$
1. 現在の方策に従って行動し、復数パスを収集し、$R_t = \sum\_{t'=t}^{H-1} \gamma^{t'-t} R\_{t'}$を計算する。
1. $\|R_t - b\|^2$ を最小にするようにbaselineを調整する
1. $\hat{g} = \sum\_\{t=0\}^H \nabla\_\{\theta\} \log \pi\_\{\theta\} (u_t^{(i)} | s_t^{(i)}) (R(\tau^{(i)}) - b)$で勾配を更新する
1. $end$ $for$

baselineの調整と、勾配の更新を繰り返していき、方策を最適化する。

## REINFORCE アルゴリズム
上記の復数パスの情報を使って計算した$R_t$の代わりに、そのときどきの報酬$r_t$を使う方法は **REINFORCE** アルゴリズムとして知られている。
baselineとしては報酬の平均値$\bar{b} = \frac{1}{MT} \sum\_{m=1}^M \sum\_{t=1}^T R_t^m$がよく用いられるらしい。

# Actor-Critic
baselineを導入したとこで、方策の更新は、方策の分散を小さくする評価部と、方策を更新する部分の2つに分けられることがわかる。
ここで、baselineとして、価値関数$V^{\pi}$を使うと
ここで、$R(\tau) = \sum\_{t=0}^H R(s_t, u_t)$の代わりに行動価値関数$Q^{\pi}(s, a)$を、baselineとして価値関数$V^{\pi}(s)$を使うことにする。
baselineとして状態$s$の関数を用いても、勾配の平均値には影響がないため、baselineとして採用できる。

以下の行動価値感数と状態価値関数の差分$A^{\pi}(s, a)$をアドバンテージ関数と呼ぶ。
\begin{equation}
A^{\pi}(s, a) = Q^{\pi}(s, a) - V^{\pi}(s)
\end{equation}

アドバンテージを使って、勾配は
\begin{equation}
\hat{g} = \frac{1}{m} \sum\_{i=0}^m \sum\_\{t=0\}^H \nabla\_\{\theta\} \log \pi\_\{\theta\} (u_t^{(i)} | s_t^{(i)}) A^{\pi}(s, a)
\label{eq:base_lined_policy_gradient}
\end{equation}
で更新できる。
このアドバンテージを小さくする **Critic** と方策を更新する **Actor** を組み合わせた方法全般を **Actor-Critic** 法と呼ぶ。

ActorとCriticはそれぞれ様々な方法で実装できる。
Criticとしては前々回までで扱った価値反復法や、最小二乗法を用いることができる。
Actorは通常の方策勾配法や、自然勾配を用いる方法など様々考えられる。
