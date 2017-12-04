+++
date = "2017-11-21T19:00:45+09:00"
draft = true
title = "Google Homeを使いたおす！ DialogflowとAoG Meetup 参加してきた"

+++

[イベントページ](https://gcpug-tokyo.connpass.com/event/71279/)

# スケジュール
20:15-20:30	休憩
22:20-22:30	Coming Soon

# セッション
## Dialogflowで作るアシスタントアプリ
@ymotongpoo 山口よしふみさん

Developer Advocate, Google Japan
Acions on Google, AMP/PWA, Android, Tango, Go

### 音声対話型アプリに期待されること
Voice User Interface(VUI) とGUIの違いを考えないといけない

* 音声入力デバイスの特徴
	* 部屋の中に置かれている
	* スクリーンがない
		* カルーセル
		* suggestion
		* list
		* 音声では結果を確認するのに時間がかかってしまう
			* eg. コールセンター
* どうするか
	* ステップを減らす事が重要
* 人間との対話を想定
	* 怒ってしまう

## Actions on Google
GoogleHome ... Google Assistantを搭載したスマートスピーカー
GoogleAssistant
Actions on Google ... 外からGoogleアシスタントを拡張するしくみ

Actions on Googleの特徴
呼び出すデバイス本体へのThirdPartyアプリのインストールの必要はない
音声認識、音声合成はGoogle Assistantが担当
RPC形式でJSOn over Http
Dialogflowなどを利用することで会話をGUIで作成可能
9 aleに対応

* 音声認識(STT) speech to text
* 音声合成(TTS)

JSONでやりとりする
GoogleAssistantは音声認識、音声合成のみを担当する

## DialogFlow
* 入力とIntentの対応をGUIで設定することができる
* 形態素解析もサービス内エンジンで処理
* 入力とIntentの割り振られ方の履歴から再割り振りを行い、Agentの学習を継続的に行うことができる。間違えても学習してくれる
*

### dialogflow使い方
intent name: やりとりには使われない。なんでも良い。
Action name: webhookに渡すための名前。GoogleAssistantの文脈のActionとは違うものなので
context: 状態を管理するために使う
Entitiy: user saysの数が爆発するのを防ぐ

# 19:45-20:15	Dialogflow + 赤外線で家電管理
fishさん @SENSY

Smart家電を追加で買いたくない
=> 赤外線をなんとかすれば家電操作できる

構成
* Google Home
* DialogFlow
* Backend(Firebase Cloud Functions)
* IRKit


* テスト状態でGoogleHomeを利用できる

# RasPi + Assistant SDK + AoGで3Dプリンター管理~Dialogflowで非同期同期やれるかな~

かぶぶ

Slack BotからActions on Googleに対応

Q.
* どうやって進捗率を取得している?
*

* できるんじゃないかと思って自然言語で話しかけられる

Cloud Functions => Cloud PubSub

Cloud PubSub : Message Queue

非同期の処理を扱えない

無理やり同期的にする: DialogFlowの中でCloudPubSub
Webhookのtimeoutは5秒なので、結構失敗する

## 困ったこと
### 定義済みのEntityは増やせないし減らせない: 例えば金色はない

赤くして => 赤色
文字列でしか帰ってこない。colorcodeに変換したい

### 長文は鬱陶しい
はなしい

### 文字列と話し言葉の印象が違う
やわらかい印象にしづら

### 抜けられないとイライラする
抜けるためのパターンをとにかく沢山用意する

## AIY
Voice KITの紹介

* ボタンを押して話しかける
* GoogleHomeっぽくする必要はない

Google Assistant API


# SUUMOとホットペッパーグルメにAoGを使ってローンチしてみた　〜デモもあるよ！〜
## SUUMOの事例
DialogFlow => Lambda
Lambdaを使ったのはGCPのCloudFuncitonがJP Regionに無かったため

JSONの形式を間違えるとつらいので、actions on goole のnpmを使うべき

### 苦しんだこと
* documentによってjson形式が違うことがある。
* jsonのどの
* dialogFlowのcontextｗ使って検索条件を保存している
* GUIで操作するのつらい

### 感想
* Dialogflowのおかげでそれなりにさわれる
* IFTTTとかをつかう

## ホットペッパーグルメにAoG
画面ありデバイスとなしデバイスでは挙動が全く違う
=> Conversatoin branching
=> 画面の有り無しで動作を切り替えることになる。

### 気をつけた部分
* Chat Bubbleで切る
* 認識した言葉を返している

### 気づき
* STTが意図しない言葉を拾ってくる
* 長い文章はとても良く拾ってくれる
	* ユーザーに長く話してもらうようにした
* TTSをちゃんと調教しないといけない
	* displayTextと話す言葉は別に指定しないといけない
		* 11日(火) : speechは11日火曜日
* ngrok最強
* brand verificationをする
	* 有名な
	* 歯車 -> Project settings -> BRAND VERIFICATIONタブ
* ログ監視する
	* 何を失敗したかフィードバックしたほうがいい

### 今後
* Handoff機能
	* 画面を持っているデバイスに

# Dialogflow + MAGELLAN BLOCKSで商品のロケーション案内するよ
Groupnoutes のYoshimura Satoshiさん

## MAGELLAN BLOCKSとは
ML用のモジュール

## 事例
ホームセンターでの案内
pepper => speech api => dialogflow => MLBlock

速く返すための

* referenceに答え、entityのvalueに候補を置く。DialogFlowだけで処理をする

# 行き先 確認エージェントを最小手で作る
@a2c 中村厚 さん

学び
* ヘルプ
	* 失敗したらHelp
	* 聞かれたらHelp

はじめまして。昨日のDialogflowとAoG Meetup、参加させていただきました。AoGを触り始めていたところ、大変参考になりました。
いくつか疑問点があります
1. GoogleAssistant
