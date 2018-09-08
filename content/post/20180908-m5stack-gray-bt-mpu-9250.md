---
title: M5stack Gray で MPU9250 で取得した角度を Bluetooth SPPで送る
date: 2018-09-08T15:26:57+09:00
lastmod: 2018-09-08T15:26:57+09:00
cover: "/images/2018/09/m5stack.jpg"
draft: true
categories: ["電子工作"]
tags: ["IMU", "MPU-9250", "Bluetooth", "M5Stack", "M5Stack Gray", "Arduino", "ESP32"]
description: 
---

# M5Stack Gray
[M5Stack Gray](https://www.switch-science.com/catalog/3648/)を買ったので使ってみている。
M5Stack GrapyはESP32にLCDとスピーカーとバッテリー、スイッチ、microSDスロットを付けてコンパクトに纏めたM5Stackに、MPU-9250 9軸IMUを搭載したモジュールである。
もちろん、ESP32が搭載されているのでBluetoothもWifiも使える。
また、ESP32向けに用意されているArduino拡張を流用してM5Stak向けにもArduino開発環境が用意されている。
Arduinoでなく、C/C++, micropythonを利用して開発する環境もある。
GPIOなどにアクセスするためのピンも本体パッケージ側面や、本体下部のアダプターに用意されているため、拡張性も高い。
M5Stackは色々なやってみたいことをサクッと実装するために、非常に強力な道具になってくれるのではないかと思う。

今回は、IMUで本体の姿勢を取得して、Bluetooth (SPP)で送信するというのをやってみた。

![M5StackGray](/images/2018/09/m5stack.jpg)

# 開発環境の準備
基本的に[Getting Started](http://m5stack.com/assets/docs/index.html)に書いてあることに従って作業すれば良いのでそんなに難しいことはない。
初心者向けにArduinoに標準対応してくれればもっと敷居が低くなるのになぁとは思った。

# M5Stack Gray で無線IMU
## 接続先PCがUbuntu16.04の場合
Bluetooth接続をシリアルポートを介した通信のように扱うSPP(Serial Protocol Profile)を使うために、Ubuntu側で準備が必要になる。

以下でBluetoothアドレスとChannelを調べる。
```
sdptool browse
```
そして以下でrfcomm0に対応付ける。この場合はアドレスが84:0D:8E:20:49:BE、チャンネルが2であった。
```
sudo rfcomm bind 0 84:0D:8E:20:49:BE 2
```
ちゃんと送られているか、試しにscreenで見てみる。
```
screen /dev/rfcomm0
```
catとechoを使っても送受信できる。
```
cat /dev/rfcomm0
```
```
echo "Hello World" > /dev/rfcomm0
```

## pythonで受信してみる

# まとめ
M5Stack、あらゆるプロトタイピングの場面で活躍しそう。
特にいわゆるIoTのものを作るのに取っ掛かりとして最高だと思う。
願わくばお値段が5000円くらいするのを2000円にしてくれたら泣いて喜ぶ。