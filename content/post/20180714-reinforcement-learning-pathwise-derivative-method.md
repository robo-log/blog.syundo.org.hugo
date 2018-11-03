---
title: "pathwise derivative method: Stocastic Value Gradient(SVG), (Deep) Deterministic Policy Gradient(DPG/DDPG)"
date: 2018-07-14T21:33:33+09:00
draft: false
---

[方策勾配法の記事](../20171117-reinforcement-learning-policy-gradient)では、確率的な方策についてその勾配を評価するためのscore functionを直接求める方法について述べた。
本記事では、方策勾配を求めるために価値関数の微分を計算する手法(pathwise derivative method)について扱う。

何らかの関数$F$があるときに、その期待値の勾配$\nabla_{\theta} \mathbb{E}[F]$を求めたいとする。
likelihood ratio methodとpathwise derivative methodの違いは、パラメタ$\theta$が期待値にかかってくる確率分布に設定されているか、モデル$F$に存在すると考えるか、ということだといえる。

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

# Stocastic Value Gradient(SVG)
[MDPとベルマン方程式](../20160410-reinforcement-learning-mdp-belman-equation)
で導出したベルマン方程式において、終端状態を表現するためだけに時変の報酬$r^{(t)} = r(s^{(t)}, a^{(t)}, t)$を用いることを想定する(一連の動作のうち最後の状態遷移で報酬$r^{(t)}$が得られるイメージ)。
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
\label{eq:sample_v}
\end{equation}

となる。

## SVG($\infty$)
時間ステップ$T$について、式\eqref{eq:sample_v}を用いて逆向きに計算し、$v^{(0)}\_{\theta}$を実際に用いるのがSVG($\infty$)である。
SVG($\infty$)のアルゴリズムを見ると、以下のようになっている。
<img src="/images/2018/07/svg_inf.png" width="400px">
つまり、

1. $t=0 \dots T$の順方向パスにおいて、現在の方策$\pi$を実行し、$(s,a,r,s')$のペアを$\mathcal{D}$に記録する。
2. $\mathcal{D}$から近似モデル$\hat{f}$を推定する。モデルとしては微分可能であれば、何でも使うことができる。論文ではNeural Networkを用いている。
3. $t=T+1$での初期値として$v's_s=0, v'\_{\theta}=0$とする
4. $t=T \dots 0$について、$\xi, \eta$の推定、式\eqref{eq:sample_v}の計算を時間ステップの逆方向にしていく。$\xi, \eta$は\mathcal{D}における同じ時刻$t$の情報を用いて推定できる。
5. $v^{(0)}\_{\theta}$を用いてパラメタ$\theta$を更新する

以上の1. - 5.を収束するまで繰り返し計算する。

ちなみに、SVG($\infty$)の$\infty$はモデル$f$を使ってどれだけの時間ステップ軌道を計算しているかということを表している。
そうすると、$\infty$でなく$T$な気がするが、私はそのあたりのニュアンスはよく理解できていない。

## SVG(1)
SVG(1)では、SVG($\inf$)とは違い、$\mathcal{D}$から価値関数$V$を学習する。
これによって、SVG($\inf$)にあった$v_s$の時間方向の更新は必要なくなる。
また、$v_s$の更新はモデル$f$を用いて1時間ステップのみ行われる。
さらに、重点サンプリングを用いて$v\_{\theta}$を更新する。
この重点サンプリングを用いる方法を論文中では特にSVG(1)-ER(Experience Replay)と呼んでいる。
論文中では$SVG(1)-ER$が最も性能が高いと報告している。

SVG(1)のアルゴリズムを以下に引用する。
<img src="/images/2018/07/svg_1_er.png" width="400px">

価値関数$\hat{V}$の推定にはFitted Policy Evaluationが用いられている。
<img src="/images/2018/07/fitted_policy_eval.png" width="400px">

## SVG(0) と DPG/DDPG
SVG(0)はSVG(1)と似通った手法だが、モデル$f$を用いない、モデルフリーな手法である。
そのかわりに、$V$でなく行動価値関数$Q$を学習する。
<img src="/images/2018/07/svg_0.png" width="400px">

SVG(0)はDeterministic Policy Gradient(DPG)あるいは特にActor, CriticにDeep Neural Networkを用いるDeep Deterministic Policy Gradient(DDPG)とほとんど同じ手法となっている。
DPGとの違いは、SVG(0)では方策$\pi$を確率的なものとして扱っているので、

$\pi$の確率変数$\eta$を推定し、
$\mathbb{E}\_{\rho(\eta)} \left\[
  Q_a \pi\_{\theta} | \eta
\right\]$
を求める。
これは実際には上記のアルゴリズムにあるようにモンテカルロ法で計算される。

# 動画
<iframe width="560" height="315" src="https://www.youtube.com/embed/PYdL7bcn_cM" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

# まとめ
SVGとその亜種についてまとめた。
DPG/DDPGと非常に近い手法であることが理解できてよかった。
