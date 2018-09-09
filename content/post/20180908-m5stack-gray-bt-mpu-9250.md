---
title: M5stack Gray で MPU9250 で取得した角度を Bluetooth SPPで送る
date: 2018-09-08T15:26:57+09:00
lastmod: 2018-09-08T15:26:57+09:00
cover: "/images/2018/09/m5stack.jpg"
draft: false
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
* https://github.com/m5stack/M5Stack/blob/master/examples/Modules/MPU9250/MPU9250BasicAHRS/MPU9250BasicAHRS.ino
* https://github.com/espressif/arduino-esp32/blob/master/libraries/BluetoothSerial/examples/SerialToSerialBT/SerialToSerialBT.ino

以上のコードを組み合わせれば簡単に姿勢をBluetoothで送れる。
Mahony filterというのを使ってクオータニオンを算出し、Yaw-Pitch-Rollに変換しているようだ。

元のコードそのままだとかなり煩雑になっているので、整理した。
https://github.com/hackathon201808/hackathon201808/blob/master/arduino/m5stackgray/imu_bt/imu_bt.ino
これで姿勢について様々な処理を追加する見通しが立てやすくなったと思う。

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
受け取ったデータをcsvとかに書き出せると便利だと思うので、pythonで受け取ったデータをcsvに保存するやつ書きました。
モーションの解析とかできそう。

```python
import time
import datetime

import serial
import pandas as pd


class IMUBTRecorder:
    def __init__(self, port, baudrate):
        self._port = port
        self._baudrate = baudrate
        self._columns=['ax', 'ay', 'az', 'gx', 'gy', 'gz', 'yaw', 'pitch', 'roll']
        self._data_frame = pd.DataFrame(columns=self._columns)
        self._start_at = None
    
    def start(self):
        with serial.Serial(self._port, timeout=0.1, baudrate=self._baudrate) as se:
            while True:
                line = se.readline()
                # remove return and line feed
                str_data = str(line).split('\\r\\n')[:-1]
                if not str_data:
                    continue
                # split by tag
                data = str_data[0].split('\\t')
                if len(data) < 10:
                    continue
                # remove data name column
                data = data[1:]
                now = time.time()
                if not self._start_at:
                    self._start_at = now
                elapsed = now - self._start_at
                parsed_data = {k: float(v) for k, v in zip(self._columns, data)}
                parsed_data['time'] = elapsed
                self._on_data_recieved(parsed_data)
                if elapsed > 2.0:
                    self._save()
                    break

    def _on_data_recieved(self, data):
        # print(data)
        self._make_log(data)

    def _make_log(self, data):
        s = pd.Series(data)
        self._data_frame = self._data_frame.append(s, ignore_index=True)

    def _save(self):
        self._data_frame.to_csv("analytics/log/log-{0:%Y-%m-%d-%H-%M-%S}.csv".format(datetime.datetime.now()))


if __name__ == '__main__':
    recorder = IMUBTRecorder('/dev/rfcomm0', 115200)
    recorder.start()
```

# まとめ
M5Stack、あらゆるプロトタイピングの場面で活躍しそう。
特にいわゆるIoTのものを作るのに取っ掛かりとして最高だと思う。
願わくばお値段が5000円くらいするのを2000円にしてくれたら泣いて喜ぶ。