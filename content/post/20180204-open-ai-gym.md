---
title: "Open AI Gym & MuJoCo を使う"
date: 2018-02-04T20:21:14+09:00
draft: false
---

[OpenAI Gym](https://gym.openai.com/docs/)
は強化学習のアルゴリズムを開発するためのツールキットである。
これを用いると、強化学習を行うときに一番手間が掛かる環境を製作する部分は飛ばして、アルゴリズムの開発に集中できる。
また、皆が同じ問題を扱うことで強化学習のアルゴリズムの性能を比較しやすくなるという利点もある。
実際に、近年は強化学習のベンチマーク環境としてOpenAI Gymを使った論文は多数出てきている。
OpenAI gymは強化学習における環境と報酬、可視化の部分のみを担うので、利用するアルゴリズムについての前提は無く、強化学習アルゴリズムの実装も含まれていない。

# インストール
pipでインストールできる。
```
pip install gym
```

# 使ってみる
とりあえずこれで、シミュレーションを回せる。

```
import gym
env = gym.make('CartPole-v0')
env.reset()
for _ in range(1000):
    env.render()
    env.step(env.action_space.sample()) # take a random action
```

![cart-pole-v0](/images/2018/02/cart-pole-v0.gif)


# メソッドなどの使い方
## env
envは環境を表していて、MDPに関するメソッドやフィールドを色々持っている。
```
import gym
env = gym.make('CartPole-v0')
print(env.action_space)
#> Discrete(2)
print(env.observation_space)
#> Box(4,)
```
`env.action_space`は行動の空間を表している。
`CartPole-v0`の場合は、`action_space`の型は`Discrete`になっている。
`Discrete(n)`は非負のn個の離散値`[0, ..., n-1]` を表すので、`CartPole-v0`では0か1のどちらかが左右どちらかへの移動を表す行動を取れるということになる。

また、`env.observation_space`は観測の空間を表している。
`CartPole-v0`の場合は、`observation_space`の型は`Box`で、4次元の連続値になっている。

### Spaceのメソッドとプロパティ
`Space`はOpenAPI Gymの空間を表す抽象クラス(インターフェース)である。
さきほどの`Discrete`や`Box`はこの`Space`を継承している。

`Space`が持ってるメソッドには以下のものがある

#### sample()
`sample`は`Space`が取り得る値をランダムに生成する。
行動の初期値を作るのに使える。
```
import gym
env = gym.make('CartPole-v0')
action = env.action_space.sample()
```

#### contains(x)
`contains(x)`は`x`が`Space`の取り得る値かを真偽値で返す。
具体的には上限、下限を越えていたりすると`false`になる。

### Spaceの子クラス
Spaceを継承して`Box`や`Discrete`などの実際に使うクラスが作られている。
主なクラスとそのメソッド、プロパティは以下のとおり。

#### Box(low, high, shape)
n次元の連続値を表す

* コンストラクタの引数
  * low, high
      * 連続値の上限と下限を表す
      * `Box(np.array([-1.0,-2.0]), np.array([2.0,4.0]))` のように使う
  * shape
      * 空間の形を表すタプル。`(n,m)`の場合n*mの空間になる。省略した場合、low, highに合わせて決定される。
* プロパティ low, high, shape
  * 連続地の上限、下限、空間の次元が格納されている。

```
import gym
env = gym.make('CartPole-v0')
print(env.observation_space.high)
#> [-4.80000000e+00 -3.40282347e+38 -4.18879020e-01 -3.40282347e+38]
print(env.observation_space.low)
#>[4.80000000e+00 3.40282347e+38 4.18879020e-01 3.40282347e+38]
```

#### Discrete(n)
`Discrete(n)`は非負のn個の離散値`[0, ..., n-1]` を表す。

* コンストラクタの引数
  * `n`
      * 次元
  * プロパティ `n`
      * 次元を格納している

## env.reset()
シミュレーションを始めるときに最初に呼ぶメソッド。
イテレーションの開始ごとに呼ぶのが通常の使い方だろう。
```
observation = env.reset()
```
戻り値としては、初期の観測(状態)を返す。

### env.step(action)
`env.step(action)`はシミュレーションにおける1時間ステップの進行を行う。
引数として、現在の行動を渡す。
```
observation, reward, done, info = env.step(action)
```
戻り値は `observation, reward, done, info`で、

* `observation`
  * 状態を表す(Space)
* `reward`
  * MRPにおける報酬を表す float値
* `done`
  * エピソードの終了を表す。
  環境ごとにエピソードをどこで区切るべきか設定されているので、これに従うのが良い。
  例えば倒立振子なら、ある角度以上に倒れてしまったらエピソード終了となる。
  * よって、これが`True`になったとき、`env.reset()`を呼んでシミュレーションをリセットする。
* `info`
  * シミュレーションに関する情報。
  ログを出したりするのに便利だが、この情報を使って学習した結果は強化学習のベンチマークとしては使えない。

### env.render()
シミュレーションの状態を画面に表示する。

## envs.registry
以下で利用できる環境の一覧が得られる。
```
from gym import envs
print(envs.registry.all())
#> [EnvSpec(DoubleDunk-v0), EnvSpec(InvertedDoublePendulum-v0), EnvSpec(BeamRider-v0), EnvSpec(Phoenix-ram-v0), EnvSpec(Asterix-v0), EnvSpec(TimePilot-v0), EnvSpec(Alien-v0), EnvSpec(Robotank-ram-v0), EnvSpec(CartPole-v0), EnvSpec(Berzerk-v0), EnvSpec(Berzerk-ram-v0), EnvSpec(Gopher-ram-v0), ...
```

以前は、公式サイトの環境一覧[https://gym.openai.com/envs/](https://gym.openai.com/envs/)でサムネイルが見られたのだが、今はメンテナンスされていないように見える。
[https://github.com/openai/gym/wiki/Table-of-environments](https://github.com/openai/gym/wiki/Table-of-environments)には環境の一覧とか行動、観測の型が書いてあるので、便利だと思う。

## 動画保存
wrapperを使って動画をmp4形式で録画できる。
```
from gym import wrappers
env = gym.make('CartPole-v0') # 環境作成
# 動画をpath-to-videoに保存する。force=Trueの場合は上書き保存する
env = wrappers.Monitor(env, 'path-to-video', force=True)
```

# OpenAI gymの基本的構成
以上をまとめると基本は以下の形式に従うことになる。
```
import gym
env = gym.make('CartPole-v0')
for i_episode in range(20):
    observation = env.reset()
    for t in range(100):
        env.render()
        print(observation)
        # actionを決定するアルゴリズムを作ることが求められている
        action = env.action_space.sample()
        observation, reward, done, info = env.step(action)
        if done:
            print("Episode finished after {} timesteps".format(t+1))
            break
```

# MuJoCoを使う
CartPole-v0のような2次元のシミュレーションでなく、3次元のシミュレーションをしたいなら、[MuJoCo](http://www.MuJoCo.org/)(Multi-Joint dynamics with Contact)のセットアップが必要になる。
MuJoCoのライセンスは有料なので、試したいときは30日無料の試用ライセンスを使うことになる。

## セットアップ
インストールとライセンスファイルの配置は[MuJoCo-py](https://github.com/openai/MuJoCo-py#install-MuJoCo)の手順に従えば良いのだが、OSXでは、最新のMuJoCoを使おうとすると以下のエラーが出る問題があるようだ。(2018年1月現在)

```
raise error.DependencyNotInstalled("{}. (HINT: you need to install MuJoCo_py, and also perform the setup instructions here: https://github.com/openai/MuJoCo-py/.)".format(e))
gym.error.DependencyNotInstalled: No module named 'MuJoCo_py'. (HINT: you need to install MuJoCo_py, and also perform the setup instructions here: https://github.com/openai/MuJoCo-py/.)
```

そこで、MuJoCo-pyはversion 0.57をインストールする。
```
pip install MuJoCo-py==0.5.7
```
このMuJoCo-pyはMuJoCo1.31にしか対応していないので、[https://www.roboti.us/download/mjpro131_osx.zip](https://www.roboti.us/download/mjpro131_osx.zip)から古いバージョンをダウンロードする。

バージョン以外は公式ページ通りに作業を進めて、最後に動作するか試してみる。

```
import gym
from gym import wrappers
env = gym.make('Humanoid-v1')
env.reset()
for _ in range(1000):
    env.render()
    env.step(env.action_space.sample()) # take a random action
```

![humanoid-v1](/images/2018/02/humanoid-v1.gif)

# まとめ
OpenAI gymとMuJoCoを使う場合について使い方をまとめた。
env一覧が公式ページで見えなくなっているので、見れるサイトを用意したいなぁと思った。