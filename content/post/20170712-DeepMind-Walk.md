+++
date = "2017-07-12T01:35:21+09:00"
draft = true
title = "DeepMindのあれについて"

+++

DeepMindが公開した
[ブログ記事はこちら](https://deepmind.com/blog/producing-flexible-behaviours-simulated-environments/)

強化学習を用いて手法について
https://arxiv.org/abs/1707.02286

モーションキャプチャーを元にして学習する
https://arxiv.org/abs/1707.02201

UNREALはこれ
https://deepmind.com/blog/reinforcement-learning-unsupervised-auxiliary-tasks/

<iframe width="560" height="315" src="https://www.youtube.com/embed/hx_bgoTF7bs" frameborder="0" allowfullscreen></iframe>


In order to learn effectively in these rich and challenging domains, it is necessary to have a reliable and
scalable reinforcement learning algorithm. We leverage components from several recent approaches
to deep reinforcement learning. First, we build upon robust policy gradient algorithms, such as trust
region policy optimization (TRPO) and proximal policy optimization (PPO) [7, 8], which bound
parameter updates to a trust region to ensure stability. Second, like the widely used A3C algorithm
[2] and related approaches [3] we distribute the computation over many parallel instances of agent
and environment. Our distributed implementation of PPO improves over TRPO in terms of wall clock
time with little difference in robustness, and also improves over our existing implementation of A3C
with continuous actions when the same number of workers is used.

# PPO
https://arxiv.org/pdf/1707.06347.pdf

OpenAIはデフォルトのアルゴリズムをPPOにする
https://blog.openai.com/openai-baselines-ppo/

発表時の動画
https://channel9.msdn.com/Events/Neural-Information-Processing-Systems-Conference/Neural-Information-Processing-Systems-Conference-NIPS-2016/Deep-Reinforcement-Learning-Through-Policy-Optimization
スライド
https://people.eecs.berkeley.edu/~pabbeel/nips-tutorial-policy-optimization-Schulman-Abbeel.pdf
