---
title: A survey on policy search algorithms for learning robot controllers in a handful of trials
date: 2018-10-22T20:40:31+09:00
lastmod: 2018-10-22T20:40:31+09:00
draft: true
categories: ["category1"]
tags: ["tag1", "tag2"]
description: 
---

# Using priors on the policy parameters/representation
## Hand-designed policies
* [Peggy Fidelman and Peter Stone. Learning ball acquisi- tion on a physical robot. In International Symposium on Robotics and Automation (ISRA), 2004.](http://www.cs.utexas.edu/users/ai-lab/?fidelman:isra04)

## Policies as function approximators
* [F. Guenter, M. Hersch, S. Calinon, and A. Billard. Rein- forcement learning for imitating constrained reaching move- ments. Advanced Robotics, Special Issue on Imitative Robots, 21(13):1521–1544, 2007.](https://infoscience.epfl.ch/record/114046)

## Dynamical Movement Primitives
[FreekStulp,EvangelosTheodorou,andStefanSchaal.Re- inforcement learning with sequences of motion primitives for robust manipulation. IEEE Transactions on Robotics, 28(6):1360–1370, 2012.
](https://ieeexplore.ieee.org/document/6295672)

### Hierarical Policy
[C. Daniel, G. Neumann, O. Kroemer, and J. Peters. Hier- archical relative entropy policy search. Journal of Machine Learning Research (JMLR), pages 1–50, 2016.](http://jmlr.org/papers/v17/15-188.html)

## Learning the controller
[M. Kalakrishnan, L. Righetti, P. Pastor, and S. Schaal. Learning force control policies for compliant manipulation. In IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS 2011), 2011.](https://ieeexplore.ieee.org/abstract/document/6095096)

<iframe width="560" height="315" src="https://www.youtube.com/embed/LkwQJ9_i6vQ" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## Learning the policy representation
NeaTの話

## Initialization with demonstrations / imitation learning
imitation learningの話

# Learning models of the expected return
## Bayesian optimization
[日本語スライド](https://www.slideshare.net/issei_sato/bayesian-optimization)

[EricBrochu,VladM.Cora,andNandoDeFreitas.Atutorial on Bayesian optimization of expensive cost functions, with application to active user modeling and hierarchical rein- forcement learning. arXiv preprint arXiv:1012.2599, 2010.](https://arxiv.org/abs/1012.2599)

## Bayesian optimization with priors


# Model-based Policy Search
## Model Learning

解説
* PILCO
[MarcPDeisenrothandCarlEdwardRasmussen.PILCO:A model-based and data-efficient approach to policy search. In ICML, 2011.](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.231.4371)
[PILCO web site](http://mlg.eng.cam.ac.uk/pilco/)

<iframe width="560" height="315" src="https://www.youtube.com/embed/XiigTGKZfks" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>


* BLACK-DROP
[Konstantinos Chatzilygeroudis, Roberto Rama, Rituraj Kaushik, Dorian Goepp, Vassilis Vassiliades, and Jean- Baptiste Mouret. Black-Box Data-efficient Policy Search for Robotics. In International Conference on Intelligent Robots and Systems (IROS). IEEE, 2017.](https://arxiv.org/abs/1703.07261)

<iframe width="560" height="315" src="https://www.youtube.com/embed/kTEyYiIFGPM" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

* M-PGPE
[Voot Tangkaratt, Syogo Mori, Tingting Zhao, Jun Morimoto, and Masashi Sugiyama. Model-based policy gradients with parameter-based exploration by least-squares conditional density estimation. Neural Networks, 57:128–140, 2014.](https://arxiv.org/abs/1307.5118)

### local model (Linearlizationしたもの)を使う
* [Vikash Kumar, Emanuel Todorov, and Sergey Levine. Op- timal control with learned local models: Application to dex- terous manipulation. In Robotics and Automation (ICRA), 2016 IEEE International Conference on, pages 378–383. IEEE, 2016.](https://www.semanticscholar.org/paper/Optimal-control-with-learned-local-models%3A-to-Kumar-Todorov/3a003991d7500e2ddb7a32d46db65c522d2174f5)

<iframe width="560" height="315" src="https://www.youtube.com/embed/bD5z1I1TU3w" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

TODO: local modelについて学ぶ
PILCOが何をしているのか学ぶ
