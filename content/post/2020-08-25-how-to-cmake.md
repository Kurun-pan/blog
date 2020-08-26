---
title: "CMakeの基本的な使い方"
date: 2020-08-25T00:00:00+09:00
bigimg: [{src: "/img/sphere.jpg", desc: ""}]
draft: false
comments: true
tags: ["CMake"]
---

## 1. はじめに
-------

自分の備忘録として、CMakeの基本的な使い方をまとめておきます。
普段利用するものの頻繁に書くものではないため、ゼロから書こうとするとあれ？って忘れている事が多いので。

<!--more-->

作業スペースのカレントディレクトリに`CMakeLists.txt`を作成します。
ソースコードは`src`ディレクトリ以下に配置しているものとします。

```
├── CMakeLists.txt
└── src/main.c
└── src/hoge.cc
```

まず、ごく基本的な例を以下に示します。

```
cmake_minimum_required(VERSION 3.10)
project(Project_name_free)
set(CMAKE_CXX_STANDARD 14)

find_package(PkgConfig)
pkg_check_modules(EGL REQUIRED egl)

add_executable(target_name src/main.c src/*.cc)
target_include_directories(target_name PRIVATE ${EGL_INCLUDE_DIRS})
target_link_libraries(target_name PRIVATE ${EGL_LIBRARIES})
target_compile_options(target_name PUBLIC ${EGL_CFLAGS} "-Wall")
```

<br>


## 2. プロジェクト設定
-------

### [cmake_minimum_required](https://cmake.org/cmake/help/latest/command/cmake_minimum_required.html) コマンド
ビルドを実行するHost PC環境で必要になるCMakeの最低バージョンを指定します。

```
cmake_minimum_required(VERSION 3.10)
```

最新のCMakeの文法/コマンド等を利用していなければ、バージョン制約はほぼないと思います。
例えばUbuntu18.04のCMakeのバージョンが3.10のため、この辺のバージョンを指定しておけば良いと思います。

<br>

### [project](https://cmake.org/cmake/help/latest/command/project.html) コマンド

CMakeプロジェクト名を指定するために利用します。

```
project(Project_name_free)
```

指定した文字列が`PROJECT_NAME`にセットされます。
オプションとして、`VERSION`や`LANGUAGES`がありますが単なる付加情報のため、
指定してもしなくても良いです。

<br>

### [CMAKE_CXX_STANDARD](https://cmake.org/cmake/help/git-master/variable/CMAKE_CXX_STANDARD.html) プロパティ

C++の場合、以下の様にビルド時に指定するC++標準バージョンを指定する事が出来ます。

```
set(CMAKE_CXX_STANDARD 14)
```

<br>


## 3. PkgConfigの利用
-------

ビルド時のヘッダーファイルのインクルードやライブラリリンクオプションを簡易化するために、
`pkg-config`はほぼ必須だと思います。これをCMakeで利用する方法について説明します。

<br>

### [find_package](https://cmake.org/cmake/help/latest/command/find_package.html) コマンド

`find_package`コマンドを利用すると、利用したいライブラリ (パッケージ) のライブラリパスや
インクルードファイルパスなどを取得する事が可能です。

下記を実行すると、`GTK2_INCLUDE_DIRS`にインクルードファイルのパス、`GTK2_LIBRARIES`に
ライブラリパスが自動で設定されます。

```
find_package(GTK2)
```

<br>

利用可能なモジュールは、`cmake --help-module-list`コマンドで表示される一覧で確認が可能です。
ただ、find_packageをサポートしているパッケージがまあまあない事があるので、その場合には後述の
`pkg_check_modules`コマンドを利用します。

なので、よくあるパターンとしては、以下の例の様に`find_package`コマンドはPkgConfigモジュールを
指定してそれが提供する`pkg_check_modules`コマンドを利用するというやり方です。

```
find_package(PkgConfig)
```

<br>

### [pkg_check_modules](https://cmake.org/cmake/help/latest/module/FindPkgConfig.html) コマンド

実際にはこのコマンドを多用すると思います。上記の様に、`find_package(PkgConfig)`を実行しておけば、
以下の様に`pkg-config`コマンドで取得できる情報を取得可能です。

第一引数は、適当なプレフィックス (XPREFIX) します。
第二引数は、`REQUIRED`か`QUIET`を指定し、パッケージが必須の場合には`REQUIRED`を指定し、
見つからなかった場合にエラーで止まる様にします。
第三引数は、利用したいpkg-configで指定するパッケージ名です。

```
pkg_check_modules(EGL REQUIRED egl)
```

<br>

コマンド実行で設定される変数は以下です。

```
<XPREFIX>_FOUND          ... set to 1 if module(s) exist
<XPREFIX>_LIBRARIES      ... only the libraries (w/o the '-l')
<XPREFIX>_LIBRARY_DIRS   ... the paths of the libraries (w/o the '-L')
<XPREFIX>_LDFLAGS        ... all required linker flags
<XPREFIX>_LDFLAGS_OTHER  ... all other linker flags
<XPREFIX>_INCLUDE_DIRS   ... the '-I' preprocessor flags (w/o the '-I')
<XPREFIX>_CFLAGS         ... all required cflags
<XPREFIX>_CFLAGS_OTHER   ... the other compiler flags
```

<br>

## 4. ターゲットの指定
-------

### ソースコード

[`add_executable`](https://cmake.org/cmake/help/latest/command/add_executable.html)コマンドを利用します。

```
add_executable(target_name src/main.c src/*.cc)
```

<br>

第一引数は、実行バイナリの名前を指定します。
第二引数以降に、ソースコードを指定します。ワイルドカード (*) も利用可能です。
なお、ソースコードのパスはCMakeLists.txtファイルが置いてあるパスからの相対パスとなります。

<br>

### インクルードファイル

[`target_include_directories`](https://cmake.org/cmake/help/latest/command/target_include_directories.html)コマンドを利用します。
コンパイラに指定する`-I`に繋がるコマンドです。

第一引数は、実行バイナリの名前を指定します。
第二引数は、PRIVATE/PUBLIC/INTERFACEのいずれかを指定します。
第三引数は、対象のインクルードファイルのパスを指定します。

```
target_include_directories(target_name PRIVATE ${EGL_INCLUDE_DIRS})
```

<br>

ところで、第二引数に指定している`PRIVATE`, `PUBLIC`, `INTERFACE`は何でしょうか？
非常に分かり辛いですが、以下の様な感じで基本的に多いのはPRIVATEだと思います。

<br>

#### PRIVATE

ターゲットのみが必要とする場合に指定

#### PUBLIC

ターゲットとターゲットが依存するライブラリが必要とする場合に指定

#### INTERFACE

ターゲットが依存するライブラリのみ必要とする場合に指定

<br>

### ライブラリ

[`target_link_libraries`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html)コマンドを利用します。
コンパイラに指定する`-l`に繋がるコマンドです。

```
target_link_libraries(target_name PRIVATE ${EGL_LIBRARIES})
```

<br>

## 5. ビルドオプションの指定
-------

コンパイラに渡すオプションを[target_compile_options](https://cmake.org/cmake/help/latest/command/target_compile_options.html)を利用して指定する事が出来ます。


```
target_compile_options(target_name PUBLIC ${EGL_CFLAGS} "-Wall")
```

<br>


## 6. リンク
-------

 - https://qiita.com/shohirose/items/5b406f060cd5557814e9
 - https://qiita.com/tchofu/items/69dacfb93908525e5b0b
 - https://gitlab.kitware.com/cmake/cmake
   - CMakeはgitlabで管理されており、C++で書かれている様子です。
