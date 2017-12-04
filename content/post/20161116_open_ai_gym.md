+++
date = "2016-11-16T19:32:08+09:00"
draft = true
title = "Pythonではじめる強化学習 OpenAI Gym 体験ハンズオンに行ってきた"

+++

イベントのurlはこれ
http://techcircle.connpass.com/event/43844/

## 講義編
http://www.slideshare.net/takahirokubo7792/techcircle-18-python-openai-gym

* 行動評価はDeepNNで行う
* DeepQNNで学習を行わせるためのトリック
* 報酬のclipping
 * 報酬は -1, 0, 1にclipping
 * ゲーム依存になってしまうので
* Exprence replay
 * シャッフルして学習データにする
 * 学習データ間の相関を除去するため
* Fixed Target Q-Network
 * 報酬を計算するネットワークはしばらく重みを固定する

* 学習にCPUだと3日、GPUだと1日

* DQNの実装に必要な行数
 * 100行くらい
 * でもプログラムが正しい
 * OpenAIGymを使えばGPUインスタンスを使える

* Q & A
 * システムの安定性はどうなのか?
  * 補足
	 * 同じ状態において
	 * 可安定とか
	  * つまり経済だと
 * 画像の入力がstate?
  * 5stateくらい記録して
 * 連続地はOK?
 * 皆が納得する
  * openAI gymがopenな実験環境を提供できている
* policy network
