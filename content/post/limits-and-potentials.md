---
title: "Limits and Potentials"
date: 2018-04-25T21:06:45+09:00
draft: true
---

The Limits and Potentials of Deep Learning for Robotic

# Learning Challenges
## Uncertainty Estimation
NNの出力は出力段にsoftmax関数を使って[0, 1]にしているけど、それはそのまま確率論の中で使えるような値ではないため、ベイジアンの既存手法と組み合わせることができない。

## Identify Unknown
実世界では見分けたい対象以外のデータを扱う、いわゆるopen-setの問題を扱う必要があり、そういうときには間違った判定をするのでなく、UNKNOWNと判定する必要がある。

## Incremental Learning
動作させながら学習する必要がある。

## Class-Incremental Learning
クラス数を増やしながら、しかも既存の学習結果を活かしながら学習できたほうがよい。でも難しい。
教師なしでクラスを増やしたいけどむずいよということを言っているのかな？

## Active Learning
データを集めるときにより情報量の多いデータを選んでいく必要がある。
SLAMのことを言っているのかな？

# Embodiment Challenges
