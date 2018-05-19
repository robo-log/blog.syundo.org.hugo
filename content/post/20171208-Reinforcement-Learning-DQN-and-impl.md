---
title: "強化学習についてまとめる(7) DQNとDDQN"
date: 2017-12-28T20:37:58+09:00
draft: false
---
今回は[Q-Learningの回](../20171110-reinforcement-value-policy-iteration/)で取り上げた手法をニューラルネットを使って近似するように発展させたもの(DQN)についてまとめる。

DQN(Deep Q-Network)は行動価値関数$Q(s,a)$を深層ニューラルネットワークを用いて推定し、Q-Learningを行う手法である。
DQNでは行動と状態の組$(s,a)$に対してスカラー値$Q^{\*}(s,a)$を割り当てるのでは**なく**、
状態$s$において行動$a\_1, \dots, a\_N$を採用したときの値$Q^{\*}(s,a_1), \dots, Q^{\*}(s,a_N)$を予測している。
つまり、入力が状態、出力が各行動を採用した時のQ値となっているニューラルネットワークを使ってQ関数の代わりとする。

この工夫によって、例えば、状態としてニューラルに多次元の画像をそのまま入力することができるようになり、ゲーム画面を入力としてそのままゲームを学習することができるようになった。
Atariのいくつかのゲームにおいて人間より良いスコアを記録するなど目覚しい成果を上げた。

関数近似を用いたQ-Learningのアルゴリズムは以下になる。

\begin{equation}
\begin{aligned}
\delta_t = R\_{t+1} + \gamma \max\_{a^{\prime} \in A} Q\_{\theta_t}(s\_{t+1}, a^{\prime}) - Q\_{\theta_t}(s_t, a_t) \\\\\
\theta\_{t+1} = \theta_t + \alpha_t \delta\_{t+1} \nabla\_{\theta} Q\_{\theta_t} (s_t, a_t)
\end{aligned}
\end{equation}

関数近似を用いた場合、つまりDQNのような場合でもQ-Learningのアルゴリズムはほぼ同じだが、Q関数を近似しているために、最適行動価値関数への収束は保証されない。

## DQN
そこで、DQNでは学習が収束しない問題に対処するためや、学習されたネットワークを他のタスクに転用しやすくするためにいくつかの工夫をしている。

* Experience Replayの利用
  * 観測したサンプル $(s_t, s\_{t+1}, a_t, R\_{t+1})$ をリプレイバッファに蓄積し、それをランダムに並べ替えてからバッチ的に学習する。
  * これによって、サンプルの系列において時間的相関があると確率的勾配法がうまく働かなくなる問題を緩和している。
* Target Networkの利用
  * Q-Learningの更新則における目標値の計算において、学習中のネットワーク$Q\_{\theta\_t}$の代わりに、パラメータが古いもので固定されたネットワーク$Q\_{\theta\_t^{-}}$を利用する。
  * これによって、Q-Learningの目標値$R\_{t+1}+\gamma \max\_{a^{\prime} \in A} Q\_{\theta_t}(s\_{t+1}, a^{\prime}) - Q\_{\theta_t}(s_t, a_t)$と現在推定している行動価値関数$Q\_{\theta\_t}$の間の相関があるために、学習が発散、振動しやすくなる問題を緩和している。
* 報酬のクリッピング
  * 得られる報酬を$[-1, 1]$の範囲にしている。
  * これによって、復数のゲーム間で同じネットワークを使って学習をすることができる。

結局、DQNのTD誤差は以下のようになる。
\begin{equation}
\delta_t = R\_{t+1} + \gamma \max\_{a^{\prime} \in A} Q\_{\theta_t^{-}}(s\_{t+1}, a^{\prime}) - Q\_{\theta_t}(s_t, a_t)
\end{equation}

試しに、Open AIのCartpoleを使ったサンプルを[実装してみた]。(https://github.com/syundo0730/RL-samples/blob/master/dqn/main.py)
Qネットワークの目標値の計算は[L59](https://github.com/syundo0730/RL-samples/blob/master/dqn/main.py#L59)で行っている。
```
target = self.mainQ.model.predict(state)
target[0][action] = reward if done else reward + self.gamma * np.max(self.subQ.model.predict(next_state)[0])
```
ここで、`subQ`と名付けているのが、Target netrowkである。Target networkの同期は2エピソードに1回にしてみた。
```
if e % 2 == 0:
    self.mainQ = self.subQ
```

## DDQN
DQNでは目標値の計算のなかにmaxを取る操作があるが、
$\max\_{a^{\prime} \in A} Q\_{\theta_t}(s\_{t+1}, a^{\prime})$で、最大のQ値を実現するという$a^{\prime}$は、推定中のQ関数がたまたまその$a^{\prime}$について最大値を取っているだけかもしれない。
Q関数が推定中で、特定の$a^{\prime}$について推定誤差がある場合にこういうことが起こるが、これは行動空間が大きくなるほど起こりやすくなる。

これを防ぐため、DDQN(Double DQN)では、目標値を以下の$y_t^{\text{DoubleDQN}}$と置き換える。
\begin{equation}
y_t^{\text{DoubleDQN}}
= R\_{t+1} + \gamma Q\_{\theta_t^{-}}(s\_{t+1}, \underset{a^{\prime} \in A}{\text{argmax }} Q\_{\theta_t}(s\_{t+1}, a^{\prime}))
\end{equation}
こうすると、仮に$Q\_{\theta_t}(s\_{t+1}, a^{\prime})$の推定誤差によって、$a^{\prime}$が選ばれても、$Q\_{\theta\_t^{-}}$にも推定誤差があるために、過度に目標値が大きく見積もられることを抑制できるという。

DDQNの場合のQネットワークの目標値の計算は[L59](https://github.com/syundo0730/RL-samples/blob/master/dqn/main.py#L65)のようになる。
```
target = self.mainQ.model.predict(state)
next_action = np.argmax(target[0])
target[0][action] = reward if done else reward + self.gamma * self.subQ.model.predict(next_state)[0][next_action]
```