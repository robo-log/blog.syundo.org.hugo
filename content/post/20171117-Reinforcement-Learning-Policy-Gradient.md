+++
date = "2017-11-17T03:15:08+09:00"
draft = false
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
* 連続空間での行動が扱いやすくなる

などの利点がある。
一方で、

* 方策オン型の学習になるため、行動のタイミングで方策を更新しなくてはならず、学習効率が落ちる
* 行動に対して学習が敏感に影響を受けてしまうため、学習ステップの決定が難しい

などの欠点がある。

# 方策のモデルと勾配
$\theta$でパラメタライズされた確率的な方策$\pi\_{\theta}$を求める問題を考える。
$\tau$をステップ$0$から$H$までの状態-行動の系列(状態-行動空間でのパス)$\tau=(s_0, a_0, \dots, s_H, a_H)$としたとき、方策の評価関数として以下を考える。

\begin{equation}
\begin{aligned}
U(\theta) &=& E [\sum\_{t=0}^H R(s_t, u_t) ; \pi\_{\theta}] \\\\\
&=& \sum\_{\tau} P(\tau ; \theta) R(\tau)
\end{aligned}
\end{equation}
ここで、$R(\tau) = \sum\_{t=0}^H R(s_t, u_t)$としている。
また$P(\tau ; \theta)$はパスの生成モデルであり、定義より
\begin{equation}
P(\tau ; \theta) = \prod\_{t=0}^H P(s\_{t+1} | s_t, u_t) \pi\_{\theta}(u_t | s_t)
\label{eq:p_tau_theta}
\end{equation}
である。

以上の設定において、方策の学習は最終的に
\begin{equation}
\max\_{\theta} U(\theta) = \max\_{\theta} \sum\_{\tau} P(\tau ; \theta) R(\tau)
\end{equation}
を求める問題となる。
そこで、微小ステップごとに評価関数の$\theta$での微分方向
\begin{equation}
\nabla\_{\theta} U(\theta) = \nabla\_{\theta} \sum\_{\tau} P(\tau ; \theta) R(\tau)
\end{equation}
に方策を更新することで、方策を最適化することを考える。
これは以下のように変形できる。

\begin{equation}
\begin{aligned}
\nabla\_{\theta} U(\theta) &=& \nabla\_{\theta} \sum\_{\tau} P(\tau ; \theta) R(\tau) \\\\\
&=& \sum\_{\tau} \nabla\_{\theta} P(\tau ; \theta) R(\tau) \\\\\
&=& \sum\_{\tau} \frac{P(\tau ; \theta)}{P(\tau ; \theta)} \nabla\_{\theta} P(\tau ; \theta) R(\tau) \\\\\
&=& \sum\_{\tau} P(\tau ; \theta) \frac{\nabla\_{\theta} P(\tau ; \theta)}{P(\tau ; \theta)} R(\tau) \\\\\
&=& \sum\_{\tau} P(\tau ; \theta) \nabla\_{\theta} \log P(\tau ; \theta) R(\tau) \\\\\
\end{aligned}
\end{equation}

$P(\tau ; \theta)$で期待値を取る操作は、以下の平均を取る操作として近似する。
\begin{equation}
\nabla\_{\theta} U(\theta)
\approx
\hat{g} = \frac{1}{m} \sum\_{i=0}^m \nabla\_{\theta} \log P(\tau^{(i)} ; \theta) R(\tau^{(i)})
\end{equation}

この式を見ると、更新則によって、

* 報酬$R$が高いパスの存在確率を上げる
* 報酬$R$が低いパスの存在確率を下げる

方向に方策が更新されることがわかる。
これをパスではなく方策と遷移確率について立式するとどうなるだろうか。
式\eqref{eq:p_tau_theta}より、

\begin{equation}
\begin{aligned}
\nabla\_{\theta} \log P(\tau^{(i)} ; \theta)
&=& \nabla\_{\theta} \log
\left[ \prod\_{t=0}^H P(s\_{t+1}^{(i)} | s_t^{(i)}, u_t^{(i)}) \pi\_{\theta}(u_t^{(i)} | s_t^{(i)}) \right] \\\\\
&=& \nabla\_\{\theta\} \left\[ \sum\_\{t=0\}^H \log P(s\_\{t+1\}^{(i)} | s_t^{(i)}, u_t^{(i)}) \right\] +
\nabla\_\{\theta\} \left\[ \sum\_\{t=0\}^H \log \pi\_{\theta} (u_t^{(i)} | s_t^{(i)}) \right\] \\\\\
&=& \nabla\_\{\theta\} \sum\_\{t=0\}^H \log \pi\_{\theta} (u_t^{(i)} | s_t^{(i)}) \\\\\
&=& \sum\_\{t=0\}^H \nabla\_\{\theta\} \log \pi\_{\theta} (u_t^{(i)} | s_t^{(i)})
\end{aligned}
\end{equation}

となる。状態遷移確率$P(s\_\{t+1\}^{(i)} | s_t^{(i)}, u_t^{(i)})$は$\theta$をパラメタとして持たないので、$\theta$で微分するとこの項が消えるのである。

結局、勾配は
\begin{equation}
\begin{aligned}
\hat{g} = \frac{1}{m} \sum\_{i=0}^m \nabla\_{\theta} \log P(\tau^{(i)} ; \theta) R(\tau^{(i)}) \\\\\
where \hspace{15pt}
\nabla\_{\theta} \log P(\tau^{(i)} ; \theta) =
\sum\_\{t=0\}^H \nabla\_\{\theta\} \log \pi\_{\theta} (u_t^{(i)} | s_t^{(i)})
\end{aligned}
\end{equation}

すなわち
\begin{equation}
\hat{g} = \frac{1}{m} \sum\_{i=0}^m \sum\_\{t=0\}^H \nabla\_\{\theta\} \log \pi\_\{\theta\} (u_t^{(i)} | s_t^{(i)}) R(\tau^{(i)})
\label{eq:policy_gradient}
\end{equation}

で表され、方策の$\theta$での勾配のみで表現できることがわかる。

## Baseline
式\eqref{eq:policy_gradient}にbaseline $b$という値を追加する。
\begin{equation}
\hat{g} = \frac{1}{m} \sum\_{i=0}^m \sum\_\{t=0\}^H \nabla\_\{\theta\} \log \pi\_\{\theta\} (u_t^{(i)} | s_t^{(i)}) (R(\tau^{(i)}) - b)
\label{eq:base_lined_policy_gradient}
\end{equation}

baselineは$\hat{g}$の値には影響を与えないが、調整前に比べて$\nabla\_\{\theta\} \log \pi\_\{\theta\} (u_t^{(i)} | s_t^{(i)}) (R(\tau^{(i)}) - b)$の分散を減少させる効果がある。

$R(\tau^{(i)}) - b$が小さいほど分散が小さくなるので、$b$の決定法としては、$R(\tau^{(i)})$との二乗距離を小さくするように調整すればよい。
