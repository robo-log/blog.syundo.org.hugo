+++
date = "2017-06-06T22:24:18+09:00"
draft = true
title = "CMakeの使い方"

+++

## ファイル一覧を自動で取ってくる?
https://stackoverflow.com/questions/1027247/specify-source-files-globally-with-glob

こうするとファイルの追加をcmakeが検知しない。build => make

`touch CMakeLists.txt`と更新してやれば対応できる。

`find src | grep .cpp`とかやれば簡単にリストは取ってこれる。
`find src -type d` とかでディレクトリは一覧にできる。

CMakeでは、大文字、小文字を区別しない。

# 各種設定まとめ
```
project (Tutorial)
```
tutorial projectと名付ける。

```
add_executable(hoge hoge.cpp)
```
hogeという実行形式を生成すること、hoge.cppが必要なことを示している。
```

```
add_libarary(hoge static hoge.cpp)
```
生成するライブラリファイルを指定する。

```
target_link_libraries(Main hoge)
```
mainにリンクさせる。

# 疑問
INCLUDE_DIRECTORIES("src/")

ADD_LIBRARY(hoge STATIC ${SOURCES})


# link_directories

```
CMake Warning (dev) at CMakeLists.txt:47 (link_directories):
  This command specifies the relative path

    ../thirdparty/fruit/stage/lib

  as a link directory.

  Policy CMP0015 is not set: link_directories() treats paths relative to the
  source dir.  Run "cmake --help-policy CMP0015" for policy details.  Use the
  cmake_policy command to set the policy and suppress this warning.
This warning is for project developers.  Use -Wno-dev to suppress it.
```
