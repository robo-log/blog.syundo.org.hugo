---
title: "強化学習についてまとめる(6) モデルベース強化学習"
date: 2017-12-06T03:41:20+09:00
categories: ["まとめ"]
tags: ["Reinforcement Learning"]
draft: false
---

前回までは、MDPにおける状態遷移について、何の仮定も置かない方法を扱ってきた。
例えば、TD誤差を用いる方法では、状態遷移確率はサンプリングによって近似していたし、
方策勾配法では状態遷移確率は扱わなくても良いものであった。
今回は、状態遷移確率をモデルとして陽に扱う手法を扱う。
これをモデルベース強化学習と呼ぶ。

さて、強化学習全般の大まかな流れは以下のようになっていると言える。

1. 適当な方策を実行し、サンプルを集める
2. モデルを更新する(Q-Learning, Actor-Criticでは$Q$の更新, モデルベースでは$P(s'|s,a)$の推定)
3. 方策を更新する(方策の更新、最大のQ値を取る$a$の選択)

モデルベース強化学習では、上記2.については教師あり学習で行う。
そして、3.の部分で状態遷移確率を使って方策を更新する。

そのため、モデルベース強化学習の利点として

* サンプル数が少なくても学習できる
* 学習したモデルを他のタスクに転移して利用することができる。つまり汎用的な学習ができる。

ということが挙げられる。

<!-- # 状態遷移確率のモデル化
モデルベース強化学習では、状態遷移確率$P(s'| s, a)$を何らかの形式でモデル化し、教師あり学習によってそのパラメタを更新していく。
モデルとしてはガウス分布、ニューラルネットワーク、ガウス混合モデルなどが用いられる。 -->

# 基本的なアルゴリズム
現在の状態$s_t$と行動$a_t$を取って、次の状態$s\_{t+1}$を生成する関数を$f\_{\phi}(s_t, a_t)$とする。
以下にモデルベース学習で用いられるアルゴリズムの概要を示す。
以下では、学習したいタスクの一連の系列を一イテレーションとして、そのindexを$k$としている。

1. サンプル$\mathcal{D}=\left\\{ (s, a, s' )\_i \right\\}$を集めるために、何らかの方策$\pi_0(a_t|s_t)$(例えばランダム)を実行する
1. $for$ $k=1,2,\dots$ $do$
1. $f\_{\phi}(s_t, a_t)$ を学習する ($\sum_i \\\| f\_{\phi}(s_t, a_t) -s'_i\\\|^2$ を最小化)
1. $f\_{\phi}(s_t, a_t)$ を使って$\pi\_{\theta}(a_t | s_t)$を最適化する
1. $\pi\_{\theta}(a_t | s_t)$実行して、タプル$(s, a, s')$を$\mathcal{D}$に記録する
1. $end$ $for$

ただし、以上のように、全イテレーションにおいて、共通の$f\_{\phi}(s_t, a_t)$を学習する方法では

* すべての状態において収束する性質の良いモデルを学習しなければならない
* モデルが未学習のため、状態を楽観的に評価してしまうなどして、誤った方向に方策を学習してしまう

などの問題がある。
そこで、時変の関数として$f\_{\phi}(s_t, a_t)$を学習するようにする。
そのようなモデルをlocal modelと呼ぶ。

また、$\pi\_{\theta}(a_t | s_t)$を直接算出することはせず、別の扱いやすい方策を計算し、それに$\pi\_{\theta}(a_t | s_t)$をフィッティングすることで任意の方策を学習できるようにする方法をGuided policy searchと呼ぶ。
例えば、簡素な方策として線形ガウスモデル化を使い、より複雑な方策のモデルとしてニューラルネットワークを使うということがある。

<!-- local modelを使って方策を計算し、任意の方策のパラメタを目標に
方策の更新を進めていく方法をguided policy searchと呼ぶ。

この記事完成 -16:00
* 論文読み - 19:00
DeepDQN 記事 - 18:00

逆強化学習勉強 - 19:00
その記事 - 20:30 -->

## 未知のダイナミクスを用いた guided policy search
Levineら[^1]はダイナミクス$f\_{\phi}(s_t, a_t)$の学習と、guided policy search を組み合わせて、少ないサンプル数でもペグの穴への挿入や、歩行などの複雑なタスクをシミュレーション上で実現した。

[^1]: S.Levine et.al "Learning Neural Network Policies with Guided Policy Search under Unknown Dynamics" https://people.eecs.berkeley.edu/~svlevine/papers/mfcgps.pdf

さらに前イテレーションでの方策と現在の方策の間のKLダイバージェンスによる制約を適応的に変化させることで更にサンプル数を減少させ、実機でのタスクも実現した。[^2]

[^2]: S.Levine et.al "Learning Contact-Rich Manipulation Skills with Guided Policy Search" https://arxiv.org/pdf/1501.05611.pdf

guided policy searchと組み合わせたモデルベース学習の概略を以下に示す。
<!-- ただし、方策は前イテレーションと大きく乖離して学習を悪化させないようにKLダイバージェンスによる制約を加えている。 -->

1. サンプル$\mathcal{D}=\left\\{ (s, a, s' )\_i \right\\}$を集めるために、何らかの方策$\pi_0(a_t|s_t)$(例えばランダム)を実行する
1. $for$ $k=1,2,\dots$ $do$
1. local model $f\_{\phi}(s_t, a_t)$ をサンプルにフィッティングする
1. サンプル$\mathcal{D}$を使って、任意の方策をguided policy searchする
1. local model $f\_{\phi}(s_t, a_t)$ を使ってlocalな方策$p(a_t | s_t)$を最適化する (KLダイバージェンスで制約)
1. $\pi\_{\theta}(a_t | s_t)$から生成される一連の行動を実行して、タプル$(s, a, s')$を$\mathcal{D}$に記録する
1. $end$ $for$

### ダイナミクスのフィッティング
アルゴリズムのstep 3. では、仮定したダイナミクスのモデルをサンプルにフィッティングする。
モデルとしては、以下の線形ガウスモデルや、

\begin{equation}
\begin{aligned}
p(s\_{t+1} | x_t, a_t) =
\mathcal{N} (f(s_t, a_t), \Sigma) \\\\\
f(s_t, a_t ) = \frac{df}{d s_t} s_t + \frac{df}{d a_t} a_t
\end{aligned}
\end{equation}

ガウス混合モデル（Gaussian mixture model, GMM) や、ニューラルネットワークを用いることができる。

ロボットの関節が多くモデルが複雑で、高次元のシステムの場合、時変のモデルをフィッティングさせるために非常に長い訓練時間が必要になってしまう。
しかし、時間ステップ的に近いサンプルや、前イテレーションのサンプルも現在のイテレーションのある時間と強く関連していると想定できるならば、サンプル数をかさ増しし、訓練時間を短縮させることができる。
Levineら[^2]は方策更新ステップでのKLダイバージェンスの制約をコストに応じて適応的に変化させることで、現在時刻の周辺のサンプルと前イテレーションの同一時刻のサンプルを利用できるようにし、学習のイテレーション数を抑えている。

### パラメタライズされた任意の方策の利用
学習時に扱う方策$p(a_t | s_t)$とは別に実際に実行する方策として任意の関数$\pi\_{\theta}(a_t | s_t)$(例えばニューラルネットワーク)を用意し、フィッティングするようにパラメタを調整する方法がguided policy searchであった。

方策を近似する仮定を入れることで、もとの価値最適化問題は

\begin{equation}
\underset{\theta, p(\tau)}{\text{minimize }}
E\_{p(\tau)} [ l(\tau) ]
\text{ s.t. } D\_{KL}(p(s_t) \pi\_{\theta}(a_t | s_t) || p(a_t, s_t)) = 0
\end{equation}

を解く問題となる。
ここで、$\tau=(s_0, a_0, \dots, s_H, a_H)$であり、$l(\tau)$は軌道におけるコストを表す。(ここでは報酬の最大化ではなくコスト最小化問題として扱っているが、報酬の最大化と同一の問題である)
また、$p(\tau)$は$s, a$で表現される軌道における存在確率を表す。

これをラグランジュの未定乗数法で変形すると、以下の$\mathcal{L} (\theta, p, \lambda)\_{GPS}$の最小化問題となる。
\begin{equation}
\mathcal{L} (\theta, p, \lambda)\_{GPS}
= E\_{p(\tau)} [ l(\tau) ]
+ \sum\_{t=0}^T \lambda_t D\_{KL}(p(s_t) \pi\_{\theta}(a_t | s_t) || p(a_t, s_t))
\end{equation}

アルゴリズムのstep 4. では収集したサンプル$\mathcal{D}=\left\\{ (s, a, s' )\_i \right\\}$を用いて、各時間ステップ$t$について以下を最小化する
\begin{equation}
\sum\_{t=0}^T \lambda_t D\_{KL}(p(s_t) \pi\_{\theta}(a_t | s_t) || p(a_t, s_t))
\end{equation}

### 方策の更新
状態遷移確率のモデルを
\begin{equation}
\begin{aligned}
p(s\_{t+1} | x_t, a_t) =
\mathcal{N} (f(s_t, a_t), F_t) \\\\\
f(s_t, a_t ) = \frac{df}{d s_t} s_t + \frac{df}{d a_t} a_t
\end{aligned}
\end{equation}

と仮定し、
アルゴリズムのstep 5. ではiLQR(iterative Linear Quadratic Regulator)を使って最適な方策を求める。
詳細は省略するが、最適な方策$p^{\*} (a_t | s_t)$を状態フィードバックの形で書き下すことができる。

# おまけ
Levineら[^1]の方策関数としてニューラルネットワークを使ったときの動画がYoutubeに上がっていたので、貼っておく。

ペグの挿入、歩行

<iframe width="560" height="315" src="https://www.youtube.com/embed/jyjhkJ8oDU4" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>

歩行

<iframe width="560" height="315" src="https://www.youtube.com/embed/Htw5kNb8W4w" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>

なんか難しそうなタスク

<iframe width="560" height="315" src="https://www.youtube.com/embed/KphCbR5w4rQ" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>

以上、モデルベース強化学習についてまとめたが、カメラ画像などのより高次元な観測情報$o_t$がある場合にどのようにモデルベース学習するかについては触れられなかった。
そのあたりは[Deep RL Bootcamp]
(https://sites.google.com/view/deep-rl-bootcamp/lectures)
のModel-Based Deep RLで解説されている。
