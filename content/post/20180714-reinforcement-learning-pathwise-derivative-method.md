---
title: "方策勾配法 pathwise derivertive method, SVG(Stocastic Value Gradient), DPG(Deterministic Policy Gradient)"
date: 2018-07-14T21:33:33+09:00
draft: false
---

[方策勾配法の記事](../20171117-reinforcement-learning-policy-gradient)では、確率的な方策についてその勾配を評価するためのscore functionを直接求める方法について述べた。
本記事では、方策勾配を求めるために価値関数の微分を計算する手法(pathwise derivertive method)について扱う。

何らかの関数$F$があるときに、その期待値の勾配$\nabla_{\theta} \mathbb{E}[F]$を求めたいとする。
likelihood ratio methodとpathwise derivertive methodの違いは、パラメタ$\theta$が期待値にかかってくる確率分布に設定されているか、モデル$F$に存在すると考えるか、ということだといえる。

つまり、前者の場合は、[方策勾配法の記事](../20171117-reinforcement-learning-policy-gradient)で述べたように

\begin{equation}
\nabla\_{\theta} \mathbb{E}\_{x \sim p(\cdot | \theta)} [f(x)] 
= \mathbb{E}_x \[\nabla\_\theta \log p(x; \theta) f(x)\]
\end{equation}
を計算する。

後者の場合、後述するre-parametrization trickを使って

\begin{equation}
\nabla\_{\theta} \mathbb{E}\_{z \sim \mathcal{N}(0, 1)} [f(x(z, \theta))] 
= \mathbb{E}\_z [\nabla\_\theta f(x(z, \theta))]
\end{equation}
を扱う。
ここで、$f$は連続であるとしている。
そうでなければ、期待値計算の中に微分の操作を入れるような入れ替えを行うことはできない。
また、$f$は微分可能であるものとして、勾配が0にならないようにする。

# SVG
[MDPとベルマン方程式](../20160410-reinforcement-learning-mdp-belman-equation)
で導出したベルマン方程式において、終端状態を表現するためだけに事変の報酬$r^{(t)} = r(s^{(t)}, a^{(t)}, t)$を用いることを想定する(一連の動作のうち最後の状態遷移で報酬$r^{(t)}$が得られるイメージ)。
このとき、連続空間におけるベルマン方程式は以下のようになる。

\begin{equation}
V^{(t)}(s) = \int \[ r^{(t)} + \gamma \int V^{(t+1)} (s')p(s'|s, a)ds' \] p(a|s;\theta) da
\label{eq:belman_continus}
\end{equation}

ここで、時間を表すために今までの記事のように下付き文字を使わずに、一貫性が無い上付き文字を用いているが、これは元論文の中で下付き文字はその変数での微分を表すために用いられているからである。
なかなか累乗とわかりにくいので、悪あがきかもしれないが、時間については括弧で囲んでみた(汗)。

## re-parametrization trick とベルマン方程式
何らかの確率分布$\rho$から生成される確率変数$\eta \sim \rho(\eta)$、$\xi \sim \rho(\xi)$を使って、遷移確率$f$、方策$\pi$をre-parametrizationする。
具体的には
\begin{equation}
\begin{aligned}
a = \pi(s, \eta; \theta) = \mu(s; \theta) + \sigma(s; \theta) \eta \\\\\
s' = f(s, a, \xi) = \mu(s, a) + \sigma(s, a) \xi \\\\\
\eta = \xi = \mathcal{N}(0, 1)
\end{aligned}
\end{equation}
のように、確率分布を使って、ある値が生成されるというモデルを利用する。
これによって、確率的な振る舞いを入れ込みながら、変数自体は決定論的に扱うことができる。
これと、式\eqref{eq:belman_continus}より

\begin{equation}
V(s) = \mathbb{E}\_{\rho(\eta)} \left\[r(s, \pi(s, \eta; \theta)) 
+ \gamma \mathbb{E}\_{\rho(\xi)} 
\[V'(f(s, \pi(s, \eta; \theta), \xi)) \]
\right\]
\label{eq:belman_continus_det}
\end{equation}

[方策勾配法の記事](../20171117-reinforcement-learning-policy-gradient)にあるように、累積報酬の最大化をするために、累積報酬のパラメタについての勾配を求めたい。
累積報酬の期待値は状態価値関数$V$なので、その勾配を求めればよい。

詳細は[Hessら](https://arxiv.org/abs/1510.09142)のAppendix Aを参照していただくとして、パラメタ$\theta$についての勾配$V\_\theta$は式\eqref{eq:belman_continus_det}より、

\begin{equation}
\begin{aligned}
V\_s &=& \mathbb{E}\_{\rho(\eta)} \[
r\_s +
r\_a \pi\_s +
\gamma \mathbb{E}\_{\rho(\xi)}
V'\_{s'}(f\_s + f\_a \pi\_s)
\] \\\\\
V\_{\theta} &=& \mathbb{E}\_{\rho(\eta)} \[
r\_a \pi\_{\theta} +
\gamma \mathbb{E}\_{\rho(\xi)}
\[
V'\_{s'} f\_a \pi\_{\theta} +
V'\_{\theta}
\]
\]
\end{aligned}
\label{eq:belman_continus_det_div}
\end{equation}

式\eqref{eq:belman_continus_det}の期待値の計算は、確率変数$\eta, \xi$に基づくサンプリングをするモンテカルロ法を使って行う。
いちサンプルを$v\_s, v\_{\theta}$で表すと

\begin{equation}
\begin{aligned}
v\_s &=& \[
r\_s +
r\_a \pi\_s +
\gamma v'\_{s'}(f\_s + f\_a \pi\_s)
\] \|\_{\eta, \xi'} \\\\\
v\_{\theta} &=& \[
r\_a \pi\_{\theta} +
\gamma v'\_{s'} f\_a \pi\_{\theta} +
v'\_{\theta}
\] \|\_{\eta, \xi'}
\end{aligned}
\end{equation}

となる。

## SVG($\infty$)
以上の性質を利用して、SVG($\infty$)では、状態遷移確率関数$\hat{f}(s, a, \xi)$、方策$\pi(s, \eta)$を同時に学習する。
<img src="/images/2018/07/svg_inf.png" width="400px">

## 動画
<iframe width="560" height="315" src="https://www.youtube.com/embed/PYdL7bcn_cM" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
