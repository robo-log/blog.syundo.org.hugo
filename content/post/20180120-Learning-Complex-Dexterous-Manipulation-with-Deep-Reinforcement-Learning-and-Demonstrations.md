---
title: "Learning Complex Dexterous Manipulation With Deep Reinforcement Learning and Demonstrations"
date: 2018-01-20T20:15:22+09:00
draft: true
---

[Learning Complex Dexterous Manipulation With Deep Reinforcement Learning and Demonstrations](https://sites.google.com/view/deeprl-dexterous-manipulation)
[arxiv](https://arxiv.org/abs/1709.10087)を読む。

以下の動画のようにハンドを使った複雑なタスクの学習を少ない学習ステップ数で実現している。
人間のデモンストレーションを学習に用いることで、他手法と比べて人間らしい動作になっている。
例えば変に指先が曲がったりしていない。

<iframe width="560" height="315" src="https://www.youtube.com/embed/jJtBll8l_OM" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

この論文では、
1. 自然方策勾配法を使って、方策の学習をしている
2. 方策の初期値を人間によるデモを使って学習している
3. 方策勾配に人間によるデモを利用した