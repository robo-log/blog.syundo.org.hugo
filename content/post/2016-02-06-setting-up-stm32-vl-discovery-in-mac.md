+++
date = "2016-02-07T00:57:41+09:00"
draft = true
title = "STM32 VL Discovery の開発環境を整える (Mac)"

+++

Macでも開発できるか試してみた。

# launchpad project の gcc-arm-none-eabi をダウンロード
[ここ](https://launchpad.net/gcc-arm-embedded) から Mac用のバイナリをダウンロードする。
`/Applications`とかそこら辺に展開しておいて、
`.bash_profile`などに
```
export PATH=$PATH:/Applications/gcc-arm-none-eabi-5_2-2015q4/bin
```
などと書いておく。そして
```
$ source .bash_profile
```
しておく。

# OpenOCDのインストール
homebrewにビルド済みの最新バージョンがある
```
$ brew install openocd
```

# OpenOCDのビルド
[ここ](http://sourceforge.net/projects/openocd/)からソースコードをダウンロード。
ビルドしようとしても、
```
sim3x.c:867:20: error: address of array 'sim3x_info->device_package' will always evaluate to 'true' .
```
などと出るバグが[あるらしい](https://community.particle.io/t/issues-getting-started-with-the-programmer-shield/16419/14)ので、それの修正のパッチも当てている。
wgetがmacに入っていなかったら
```
$ brew install wget
$ cd openocd-0.9.0
$ wget http://openocd.zylin.com/changes/2838/revisions/169db31ae06627a073c8179dc33d1bce1e88f4d6/patch\?zip -O 169db31.diff.zip; unzip 169db31.diff.zip; patch -u src/flash/nor/sim3x.c 169db31a.diff
$ ./configure --enable-stlink
$ make
$ sudo make install
```

