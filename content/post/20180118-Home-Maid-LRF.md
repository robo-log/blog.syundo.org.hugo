---
title: "TOF型距離センサVL53L0Xを使って測域センサを作った"
date: 2018-01-18T22:48:02+09:00
draft: false
---

第2回ROBO-ONE Autoに出す(出そうとした)ために作ったヴァニラスケルトンだが、その目玉は360度全周囲計測可能な測距センサを搭載していることであった。
(ヴァニラスケルトンと名付けているが、フレームは大半をヴァニラから流用したもの。見た目があまりにも醜くなってしまったので、別プロジェクトだと明言することによって、誤魔化そうとしたのであった。)

TOF(Time Of Flight)型の測距センサVL53L0Xを使った[測距モジュール](https://www.switch-science.com/catalog/2869/)を4個取り付けてあり、サーボモータを使ってセンシング部を90度ずつ左右に反復回転させることで、360度の範囲の計測をできるようにしている。

2足ロボの頭に搭載してROBO-ONEの狭いリングをセンシングするには、ちょっと下向きを見たいので、
センサは水平から60度下向きに取り付けてある。
市販のLRFだと水平にしか計測できないけど、自作するとこういうことができるのが良い点だと思う。

<iframe width="560" height="315" src="https://www.youtube.com/embed/Yf5KiL0H3xg" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

一方で、動画を見ていただけるとわかるが、サーボモータの回転速度が遅いのでセンシング周期はとても遅い。
解像度?も10度ずつしか計測していないので低くて、とても荒い状態しか取れない(ROBO-ONEにはそれでも十分そうだが)。

とりあえず作ってみることを優先して指定の角度まで動かす=>計測=>動かすという実装になっているので、まだまだ改善はできると思う。
Raspberry Piごと回しているのもすごく無駄がある。
これは実は最初は回すつもりはなくて4方向だけ取れればいいかと思っていたのが、作ってみたらやっぱり回してみたくなったのと、ヴァニラの骨格をベースに作っているのが原因。

とりあえずRaspberry Pi Zeroで直接センサを読んでいる。
![Home maid LRF raspi](/images/2018/01/board_close_up.jpg)

計測できるデータは36次元で、プロットしてみると以下のようになる。

これは周りに何も置いていないとき。
<iframe width="660" height="415" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vSdUjyqCYW1DlT4eCL6nJ63yC6BSE-VuNcm1CCI5Vi1cDfEf3ibGNSSkZy8OfgfUw9LAUQutqWPh_hD/pubchart?oid=521443742&amp;format=image"></iframe>

これは左に箱を置いたとき。
<iframe  width="660" height="415" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vTN0nOHDY_WP1bkwXZJ1any1O2kw6NMn-IwGtUbG-xsLO7g53W50gCInmYiXYRXfBhbIZecDzyo-GLc/pubchart?oid=652289347&amp;format=image"></iframe>

横軸はデータのindexで、縦軸は距離(mm)である。
ほぼ全域に外れ値があるが、物を置いたほうの外れ値じゃない部分はなんとなく距離が近くなっているのがわかる...辛うじて。

外れ値の原因は、

* 計測不能な遠距離を計測すると 8000mm以上の値が返ってくること
* 計測完了したタイミングで値を取得していない

ことにあるようだ。
恐らく、2番目の理由が致命的で、このセンサモジュールは計測完了時に割り込みピンをraise upさせるのだが、その割り込みをキャプチャせずに適当なタイミングで計測している。
これも締め切り間近になって手を抜いた結果なので、割り込みキャプチャによって改善ができるか試したいと思っている。

ただ、外れ値を除いてざっくり8方向に平均化すれば、なんとなく使えそうなデータが出てくる。

これは前に箱を置いた場合。
![front](/images/2018/01/measureing_front.jpg)
<iframe width="660" height="415" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vTmlQ--wzVazXq4y03thtX8cBddNnt3e2dThicC4SsEe7PyeF__gAo02iAJaFe_24w2M_pj3V26bg0i/pubchart?oid=1926959665&amp;format=image"></iframe>

左に箱を置いた場合。
![left](/images/2018/01/measureing_left.jpg)
<iframe width="660" height="415" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vQnsels2-90FdNdC8beApuVriIDFTZRFOwnN7SsKqOBAUbnoUxtbqnCIymfkZmepOSt1rMt3rod_jIb/pubchart?oid=1930359957&amp;format=image"></iframe>

# まとめ
LRFみたいなやつを作ってみたけど、まだ理想の形にはなっていない。
なんとなく8方向の距離は計測できた。
今後は、センサを左右に振らないで回転させ続けられないか試す(配線が捻れないように無線で飛ばす)、割り込みピンを使う、計測したデータを使って適当にロボットの動作を決めたときにうまく動くか試す、などをやりたい。

あるいは[最近、RPLIDARを買った](../20171116-rplidar)ので、そもそも自作した意味無くなってくるが、それもうまく2足に積んで使えないかやってみたい。
反射材を使って曲げる的なことできないかという感じのことを考えている。
