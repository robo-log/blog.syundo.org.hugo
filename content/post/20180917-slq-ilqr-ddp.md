---
title: Sequential LQR(SLQ)/iterative LQR(iLQR)/Differential Dynamic Programming(DDP)とは何なのか?
date: 2018-09-17T15:22:07+09:00
lastmod: 2018-09-17T15:22:07+09:00
cover: "/images/default1.jpg"
draft: true
categories: ["まとめ"]
tags: ["SLQ", "iLQR", "DDP"]
description: 
---

離散システム最適化の手法である、Sequential LQR(SLQ), iterative LQR(iLQR), Differential Dynamic Programming(DDP) がたまに強化学習の論文に出てくるが、どういう文脈で生まれたものなのか、その関係性はどうなっているのかわからないので私は苦しめられてきた。
以下の資料を参考にして理解を深めていったので、分かる範囲のことをまとめたい。

# 離散システムの最適化問題
以下の離散システムを考える。
\begin{equation}
x\_{i+1} = f(x_i, u_i)
\end{equation}
ここで、$f$は何らかの関数で、線形とは限らない。

また、以下の評価関数を最小にしたいと考える。
\begin{equation}
\begin{aligned}
J(x, u) &=& \sum\_{i=0}^N l(x_i, u_i) + l_N \\\\\
x_0 &=& \bar{x_0} \\\\\
u_0 &=& \bar{u_0}
\end{aligned}
\end{equation}
$N$は有限の時間ステップ、$l(x_i, x_i)$はstep $i$におけるコスト関数である。

$x, u$を設計変数として最適化を行うことはできるが、システム$f(x_i, u_i)$の最適化を想定しているので、$u$が分かれば$x$は得られる関係にある。
そこで、$J$を$u$の関数として表し、$u$について最適化することが考えられる。
このような方法は、初期値から入力を操作してゴールを目指すことが的当てのように見えることから、Shooting法と呼ばれる。

## 線形システムへの近似とQuadratic Cost functionの採用
ある状態$\bar{x}_0$, $\bar{x}_0$において線形化を行う

\begin{equation}
x + \Delta x = \mathrm{A} (x-x_0) + \mathrm{B} (u - u_0)
\end{equation}

座標系を変換することで時変線形システムとしてモデリングできる

## Shooting法

# まとめ
SLQ, iLQR, DDPはとても似通った手法だった。
誤解を恐れずに書くと、SLQ $neq$ iLQR $eq$ DDP みたいなかんじだった。