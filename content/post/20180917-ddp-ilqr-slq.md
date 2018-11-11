---
title: Differential Dynamic Programming(DDP)/iterative LQR(iLQR)/Sequential LQR(SLQ)
date: 2018-09-17T15:22:07+09:00
lastmod: 2018-09-17T15:22:07+09:00
draft: false
categories: ["まとめ"]
tags: ["DDP", "iLQR", "SLQ"]
description: 
---

最適制御の手法である、Differential Dynamic Programming(DDP), iterative LQR(iLQR), Sequential LQR(SLQ) がたまに強化学習の論文に出てくるが、どういう文脈で生まれたものなのか、その関係性はどうなっているのかわからないので私は苦しめられてきた。
以下の資料を参考にして理解を深めていったので、分かる範囲のことをまとめたい。

# 離散システムの最適化問題
以下の離散システムを考える。
\begin{equation}
x\_{i+1} = f_i(x_i, u_i), i = 0, \dots, N-1
\label{eq:system_func}
\end{equation}
ここで、$f_i$は何らかの関数で、線形とは限らない。

このシステムにおいて、以下の評価関数を最小にしたいと考える。
\begin{equation}
J(x, u) = \sum\_{i=0}^\{N-1} L_i(x_i, u_i) + L_f(x_N)
\end{equation}
$N$は有限の時間ステップ、$L_i(x_i, x_i)$はstep $i$におけるコスト関数であり、
$L_f(X_N)$は終端時刻におけるコストである。
このときに、あるステップ$i$から終端までのコストを

\begin{equation}
J_i(x, u) = \sum\_{j=i}^{N-1} L_i(x_i, u_i) + L_f(x_N)
\end{equation}

とし、$i$から終端までの最適コストを
\begin{equation}
V_i(x) = \min_u J_i(x, u)
\end{equation}
とすると、最適コストは部分最適コストで表現できるため、以下のように再帰的に表現できる。
\begin{equation}
\begin{aligned}
V_i(x) &= \min_u \[ l(x_i, u_i) + V\_{i+1}(x\_{i+1}) \] \\\\\
&= \min_u \[ l(x_i, u_i) + V\_{i+1}(f(x_i, u_i)) \]
\end{aligned}
\end{equation}

ここで、システム表現$x\_{i+1} = f_i(x_i, u_i)$を用いている。
また、
\begin{equation}
Q_i(x_i, u_i) = l(x_i, u_i) + V\_{i+1}(f(x_i, u_i))
\end{equation}
とおくと、

\begin{equation}
V_i(x) = \min_u Q_i(x_i, u_i)
\end{equation}

であるから、最適制御入力は$x_i$の関数
\begin{equation}
u_i^{*}(x_i) = \text{argmin} Q_i(x_i, u_i)
\label{eq:q_minimize}
\end{equation}
を求めることで実現できる。

以上の離散システムにおいて、最適化を行う流れは以下のようになる

1. 初期軌道 $x_0, \cdots , x_N$を設定する。これはヒューリスティックに決定する。例えば、ランダムに設定したり、スタートからゴールまでを結ぶ直線上に配置するなどが考えられる。
2. 後退パスにおいて最適化する。$i=N$から$i=0$に向かって逆方向に式\eqref{eq:q_minimize}の最適化を行う
3. 2.で計算した最適制御入力を用いるとして、順方向パスを式\eqref{eq:system_func}に従って計算する。この結果を1.の初期値の代わりに用いて2.以降を繰り返す。

## Shooting法
$x, u$を設計変数として最適化を行うことはできるが、システム$f(x_i, u_i)$の最適化を想定しているので、$u$が分かれば$x$は得られる関係にある。
そこで、$J$を$u$の関数として表し、$u$について最適化している。
このような方法は、初期値から入力を操作してゴールを目指すことが的当てのように見えることから、Shooting法と呼ばれる。
式\eqref{eq:q_minimize}において$u$のみを設計変数として最適化していることから、このあと述べるDDP, iLQR, SLPもShooting法(Single Shooting)の一種ということになる。

## Neuton法を用いた最適化
関数$Q_i(x, u)$を$x_i + \delta x$, $u_i + \delta u$周りで2次のテイラー展開すると、近似した関数$\bar{Q}(x, u)$は
\begin{equation}
\begin{aligned}
\bar{Q}(x, u) =
Q +
Q_u \delta u + Q_x \delta x +
\frac{1}{2}
\left[
    \begin{array}{rr}
      \delta u & \delta x
    \end{array}
  \right]
\left\[
    \begin{array}{rr}
      Q\_{uu} & Q\_{ux} \\\\\
      Q\_{xu} & Q\_{xx}
    \end{array}
  \right\]
\left[
    \begin{array}{r}
      \delta u \\\\\
      \delta x
    \end{array}
  \right]
\end{aligned}
\end{equation}
となる。
ただし、以下では$Q_i(x_i, u_i)$を単に$Q$と、
また、関数$f(x, y)$の偏微分$\frac{\partial f}{\partial x}, \frac{\partial f^2}{\partial x^2}, \frac{\partial f^2}{\partial xy}$は$f_x, f\_{xx}, f\_{xy}$とする表記を用いる。

ここで、
$\xi = \left[
    \begin{array}{r}
      \delta u \\\\\
      \delta x
    \end{array}
  \right]
$,
$p = \left[
    \begin{array}{r}
      Q_u \\\\\
      Q_x
    \end{array}
  \right]
$,

$\mathrm{H} = \left[
    \begin{array}{rr}
      Q\_{uu} & Q\_{ux} \\\\\
      Q\_{xu} & Q\_{xx}
    \end{array}
  \right]
$
とおくと、

\begin{equation}
\bar{Q}(x, u) = Q
+ p^{\mathrm{T}} \xi
+ \xi^{\mathrm{T}} \mathrm{H} \xi
\end{equation}
であり、さらに展開すると
\begin{equation}
\begin{aligned}
\bar{Q}(x, u)
&= Q
- \frac{1}{2} p^{\mathrm{T}} \mathrm{H}^{-1} p
+ \frac{1}{2} (
\xi^{\mathrm{T}} \mathrm{H} \xi
+ 2 p^{\mathrm{T}} \xi
+ p^{\mathrm{T}} \mathrm{H}^{-1} p) \\\\\
&= Q
- \frac{1}{2} p^{\mathrm{T}} \mathrm{H}^{-1} p
+ \frac{1}{2} (
\xi^{\mathrm{T}} \mathrm{H}
+ p^{\mathrm{T}})
(
\xi + \mathrm{H}^{-1} p) \\\\\
&= Q
- \frac{1}{2} p^{\mathrm{T}} \mathrm{H}^{-1} p
+ \frac{1}{2} (
\xi + \mathrm{H}^{-1} p)
\mathrm{H}
(
\xi + \mathrm{H}^{-1} p) \\\\\
\end{aligned}
\end{equation}
であるから、
$\bar{Q}$がもとの関数を十分に近似できていると仮定したとき、
\begin{equation}
\xi = - \mathrm{H}^{-1} p
\label{eq:gradient_direction}
\end{equation}
のときに、$Q$が最小となる。
このように2次の近似を用いて最適解を求める方法は、Newton法にほかならない。

式$\eqref{eq:gradient_direction}$をさらに計算すると

\begin{equation}
\begin{aligned}
\left[
    \begin{array}{r}
      \delta u \\\\\
      \delta x
    \end{array}
  \right]
&=
-\left[
    \begin{array}{cc}
      Q\_{uu}^{-1} + Q\_{uu}^{-1}Q\_{ux}S^{-1}Q\_{xu}Q\_{uu}^{-1} & -Q\_{uu}^{-1}Q\_{ux}S^{-1} \\\\\
      -S^{-1}Q\_{xu}Q\_{uu}^{-1} & S^{-1}
    \end{array}
  \right]
\left[
    \begin{array}{r}
      Q_u \\\\\
      Q_x
    \end{array}
  \right]
\end{aligned}
\end{equation}

\begin{equation}
\begin{aligned}
=-
\left[
    \begin{array}{r}
      Q\_{uu}^{-1}Q\_u + Q\_{uu}^{-1}Q\_{ux}\[S^{-1}Q\_{xu}Q\_{uu}^{-1} -S^{-1}Q\_x\] \\\\\
      -S^{-1}Q\_{xu}Q\_{uu}^{-1}Q\_u + S^{-1}Q\_x
    \end{array}
  \right]
\end{aligned}
\end{equation}
となる。
これより、最適制御入力は以下で与えられる。
\begin{equation}
\delta u^* (\delta x) = \mathrm{k} + \mathrm{K} \delta x
\label{eq:optimal_input}
\end{equation}
\begin{equation}
\mathrm{k} = Q\_{uu}^{-1}Q\_u , \mathrm{K} = Q\_{uu}^{-1}Q\_{ux}
\label{eq:optimal_input_k}
\end{equation}

ハミルトニアン$H$の要素は
\begin{equation}
\begin{aligned}
Q_x &= l_x + f_x^{\mathrm{T}} V'_x \\\\\
Q_u &= l_u + f_u^{\mathrm{T}} V'_x \\\\\
Q\_{xx} &= l\_{xx} + f_x^{\mathrm{T}} V'\_{xx} f_x + V'_x \dot f\_{xx} \\\\\
Q\_{ux} &= l\_{ux} + f_x^{\mathrm{T}} V'\_{xx} f_x + V'_x \dot f\_{ux} \\\\\
Q\_{uu} &= l\_{uu} + f_u^{\mathrm{T}} V'\_{xx} f_u + V'_x \dot f\_{uu} \\\\\
\end{aligned}
\label{eq:hamil}
\end{equation}
であり、
\begin{equation}
\begin{aligned}
\Delta V &= - \frac{1}{2} \mathrm{k}^{\mathrm{T}} Q\_{uu} \mathrm{k} \\\\\
V_x &= Q_x - \mathrm{K}^{\mathrm{T}} Q\_{uu} \mathrm{k} \\\\\
V\_{xx} &= Q\_{xx} - \mathrm{K}^{\mathrm{T}} Q\_{uu} \mathrm{K} \\\\\
\end{aligned}
\label{eq:vs}
\end{equation}

以上の式\eqref{eq:optimal_input}, \eqref{eq:optimal_input_k}, \eqref{eq:hamil}, \eqref{eq:vs}を用いて実際に最適制御入力を求めることができる。

# Differential Dynamic Programming(DDP)
以上をふまえて、離散システムの逐次最適化における手順2に、式\eqref{eq:optimal_input}を用いる、
すなわち、

1. 初期軌道 $x_0, \cdots , x_N$を設定する。
2. $i=N$から$i=0$に向かって、式\eqref{eq:optimal_input}における係数を計算する。
3. 2.で計算した最適制御入力を用いるとして、順方向パスを式\eqref{eq:system_func}に従って計算する。この結果を1.の初期値の代わりに用いて2.以降を繰り返す。

の手順を踏む手法が、DDPである。

# 線形システムへの近似とQuadratic Cost functionの採用
非線形システム$f(x, u)$をstep $i$の状態$x_k$, 入力$u_k$において線形化することを考える

\begin{equation}
(x - x_k) = \mathrm{A_k} (x - x_k) + \mathrm{B_k} (u - u_k)
\end{equation}

ここで、
\begin{equation}
\begin{aligned}
\mathrm{A_k} = \frac{\partial f_k}{\partial x} (x_k, u_k) \\\\\
\mathrm{B_k} = \frac{\partial f_k}{\partial u} (x_k, u_k)
\end{aligned}
\end{equation}

である。
座標系を変換することで時変線形システムとしてモデリングできることがわかる。
線形システムの最適化問題としては、LQR(Linear Quadratic Regrator)がよく知られており、そのコスト関数は

\begin{equation}
l_k(x_k, u_k) = \frac{1}{2} x^{\mathrm{T}} \mathrm{Q} x +
\frac{1}{2} u^{\mathrm{T}} \mathrm{R} u
\end{equation}

この離散システムを最小化する解は以下のRicatti方程式を解くことで得られることが知られている。

\begin{equation}
S(k) = Q + A_k^{\mathrm{T}} S(k+1) A_k
- A_k^{\mathrm{T}}S(k+1)B_k(R + B_k^{\mathrm{T}}S(k+1)B_k)^{-1}B_k^{\mathrm{T}}S(k+1)A_k
\label{eq:ricatti_eq}
\end{equation}

# iterative LQR(iLQR)
式\eqref{eq:ricatti_eq}を満たす最適制御入力は式\eqref{eq:optimal_input_k}から$f$の二階微分の項を除いたものになっている。
このため、DDPがニュートン法であったのに対してiLQRはガウスニュートン法を用いたものであるとも言える。
iLQRはDDPのようにヘッシアンを計算する必要も逆行列を計算する必要もないため、DDPに比べて高速に処理できる。
一方で、DDPよりも収束は遅いため、iteration数は大きくなるようだ。

# Sequential LQR(SLQ)
SLQはiLQRに対して、離散システムの逐次最適化における手順3で、式\eqref{eq:optimal_input}を用いたものである。

# 実験
## MATLABでの計算
## pythonでの計算

# まとめ
DDP, iLQR, SLQの関係をまとめた。
どれも、2次の近似における最適化だということがわかった。

EXPLORATION BY RANDOM NETWORK DISTILLATION