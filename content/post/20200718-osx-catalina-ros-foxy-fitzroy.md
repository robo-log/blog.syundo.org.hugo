---
title: macOS 10.15 Catalina に ROS2 Foxy Fitzroyをインストールする
date: 2020-07-18T23:48:14+09:00
lastmod: 2020-07-18T23:48:14+09:00
cover: "/images/default1.jpg"
draft: false
categories: ["ROS"]
tags: ["ros2", "Foxy Fitzroy"]
description: 
---

OSX 10.15 Catalina に ROS2 Foxy Fitzroyをインストールしてサンプルプログラムのビルドまで行います。
ROS2 Foxy FitzroyはmacOS Mojave (10.14)までの対応となっていますが、Catalinaでも動作させることができます。

# ROS2 セットアップ
## 基本手順
ROS2のビルド済みバイナリを用いるセットアップをまず行います。
以下のサイトの手順通りで問題ありませんでした。
https://index.ros.org/doc/ros2/Installation/Foxy/macOS-Install-Binary/

## ビルドすると起こる問題の対処
### Qt5について
Qt用のcmake fileが見つからない以下のエラーが出ます。
```
--- stderr: turtlesim                         
CMake Error at CMakeLists.txt:15 (find_package):
  By not providing "FindQt5.cmake" in CMAKE_MODULE_PATH this project has
  asked CMake to find a package configuration file provided by "Qt5", but
  CMake did not find one.

  Could not find a package configuration file provided by "Qt5" with any of
  the following names:

    Qt5Config.cmake
    qt5-config.cmake

  Add the installation prefix of "Qt5" to CMAKE_PREFIX_PATH or set "Qt5_DIR"
  to a directory containing one of the above files.  If "Qt5" provides a
  separate development package or SDK, be sure it has been installed.


---
```

`~/.bash_profile` に以下を追記してcmakeにQtの場所を明示すれば問題なくなります。

```
echo "export Qt5_DIR=$(brew --prefix qt5)/lib/cmake/Qt5" >> ~/.bash_profile
source ~/.bash_profile
```

### openSSL について
ROSの通信の暗号化に使われるファイルが見つからないエラーが出ます。
```
--- stderr: turtlesim                               
ld: cannot link directly with dylib/framework, your binary is not an allowed client of /usr/lib/libcrypto.dylib for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make[2]: *** [libturtlesim__rosidl_typesupport_fastrtps_cpp.dylib] Error 1
make[1]: *** [CMakeFiles/turtlesim__rosidl_typesupport_fastrtps_cpp.dir/all] Error 2
make: *** [all] Error 2
make: INTERNAL: Exiting with 5 jobserver tokens available; should be 4!
---
```

そこで、brewでinstallしたopenSSLから静的リンクを貼ってやって見つかるようにします。

まず既存のファイルが壊れてしまわないように念の為バックアップを作ります。
そのためには "System Integrity Protection" の解除が必要です。
ROS2のinstall手順で必要なのですでにやっているはずですが、やっていない場合は
https://developer.apple.com/library/archive/documentation/Security/Conceptual/System_Integrity_Protection_Guide/ConfiguringSystemIntegrityProtection/ConfiguringSystemIntegrityProtection.html
の手順に従って解除します。

次にroot directoryをreadonlyでない状態でマウントします。
これをやっておかないと `Read-only file system` と言われます。
```
sudo mount -r -w /
```
以下の2ファイルをバックアップしておきます

```
sudo mv /usr/lib/libssl.dylib /usr/lib/libssl.dylib.bak
sudo mv /usr/lib/libcrypto.dylib /usr/lib/libcrypto.dylib.bak
```

次にhome brewでinstallしたopenSSLのdlibから静的リンクを張ります。
home brewでinstallしたdlibファイル一覧を確認します。

```
$ brew list openssl | grep lib
/usr/local/Cellar/openssl@1.1/1.1.1g/lib/libcrypto.a
/usr/local/Cellar/openssl@1.1/1.1.1g/lib/pkgconfig/openssl.pc
/usr/local/Cellar/openssl@1.1/1.1.1g/lib/pkgconfig/libssl.pc
/usr/local/Cellar/openssl@1.1/1.1.1g/lib/pkgconfig/libcrypto.pc
/usr/local/Cellar/openssl@1.1/1.1.1g/lib/libssl.dylib
/usr/local/Cellar/openssl@1.1/1.1.1g/lib/libssl.1.1.dylib
/usr/local/Cellar/openssl@1.1/1.1.1g/lib/libcrypto.dylib
/usr/local/Cellar/openssl@1.1/1.1.1g/lib/libssl.a
/usr/local/Cellar/openssl@1.1/1.1.1g/lib/engines-1.1/padlock.dylib
/usr/local/Cellar/openssl@1.1/1.1.1g/lib/engines-1.1/capi.dylib
/usr/local/Cellar/openssl@1.1/1.1.1g/lib/libcrypto.1.1.dylib
/usr/local/Cellar/openssl@1.1/1.1.1g/share/man/man3/ERR_get_next_error_library.3ssl
/usr/local/Cellar/openssl@1.1/1.1.1g/share/man/man3/ERR_lib_error_string.3ssl
/usr/local/Cellar/openssl@1.1/1.1.1g/share/man/man3/SSL_library_init.3ssl
/usr/local/Cellar/openssl@1.1/1.1.1g/share/doc/openssl/html/man3/SSL_library_init.html
/usr/local/Cellar/openssl@1.1/1.1.1g/share/doc/openssl/html/man3/ERR_lib_error_string.html
/usr/local/Cellar/openssl@1.1/1.1.1g/share/doc/openssl/html/man3/ERR_get_next_error_library.html
```

以下の2つのファイルを使う必要があります。
```
/usr/local/Cellar/openssl@1.1/1.1.1g/lib/libssl.1.1.dylib
/usr/local/Cellar/openssl@1.1/1.1.1g/lib/libcrypto.1.1.dylib
```

静的リンクを張ります。
```
sudo ln -s /usr/local/Cellar/openssl@1.1/1.1.1g/lib/libssl.1.1.dylib /usr/lib/libssl.dylib
sudo ln -s /usr/local/Cellar/openssl@1.1/1.1.1g/lib/libcrypto.1.1.dylib /usr/lib/libcrypto.dylib
```

# サンプルのビルド
https://index.ros.org/doc/ros2/Tutorials/Workspace/Creating-A-Workspace/
の手順のとおりにサンプルプロジェクトがビルドできるか確認します。
colconが必要ですので入れます。
```
python3 -m pip install colcon-common-extensions
```
ビルドします。
```
mkdir -p ~/ros2_example_ws/src
cd ~/ros2_example_ws/src
git clone https://github.com/ros/ros_tutorials.git -b foxy-devel
cd ..
colcon build
```

ビルドが成功しました。
```
$ colcon build
Starting >>> turtlesim
[Processing: turtlesim]                             
[Processing: turtlesim]                                     
Finished <<< turtlesim [1min 30s]                              

Summary: 1 package finished [1min 30s]
```

別の端末を開いてビルドしたturtle_simを実行すると動作します。

```
. install/local_setup.bash
ros2 run turtlesim turtlesim_node
```

<img src="/images/2020/07/my_turtle_sim.png" width="400px">