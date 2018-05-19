---
title: "20180121 Policy Gradient Methods"
date: 2018-01-21T18:18:13+09:00
draft: true
---

# 導関数を用いない手法
## CEM (Cross-Entoropy-method)

Full	episode	evaluaMon,	parameter	perturbaMon	
n Simple	
n Main	caveat:	best	when	number	of	parameters	is	relaMvely	small	
n i.e.,	number	of	populaMon	members	comparable	to	or	larger	than	number	of	
(effecMve)	parameters
à	in	pracMce	OK	if	low-dimensional	θ	and	willing	to	do	do	many	runs	
à	Easy-to-implement	baseline,	great	for	comparisons!


https://www.roboti.us/download/mjpro131_osx.zip
pip install mujoco-py==0.5.7



--- mujoco error
raise error.DependencyNotInstalled("{}. (HINT: you need to install mujoco_py, and also perform the setup instructions here: https://github.com/openai/mujoco-py/.)".format(e))
gym.error.DependencyNotInstalled: No module named 'mujoco_py'. (HINT: you need to install mujoco_py, and also perform the setup instructions here: https://github.com/openai/mujoco-py/.)