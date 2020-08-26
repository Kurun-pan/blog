---
title: "RISC-V Toolchainのインストール手順"
date: 2020-08-16T17:47:01+09:00
bigimg: [{src: "/img/sphere.jpg", desc: ""}]
draft: false
comments: true
tags: ["RISC-V"]
---

{{< figure src="/images/riscv.png" title="" >}}

## 1. はじめに
------

RISC-Vプロセッサ向けのToolchain (クロスコンパイラ) のインストール手順を解説します。
対象OSは`macOS`ですが、基本的にLinux (Ubuntuなど) でも同じ手順です。

<!--more-->

<br>

## 2. インストール手順
------

### 必要なパッケージのインストール

```
$ brew install python3 gawk gnu-sed gmp mpfr libmpc isl zlib expat
```

### Toolchainのビルドとインストール

makeコマンドを実行することで、インストールまで実行されます。
また、git cloneにも結構な時間を要します。ビルドも同様です。。

`configure`の`prefix`オプションには、インストール先のパスを指定します。

```
$ git clone --recursive https://github.com/riscv/riscv-gnu-toolchain
$ cd riscv-gnu-toolchain
$ ./configure --prefix=/opt/riscv
$ sudo make
```

### PATH追加

```
$export PATH=$PATH:/opt/riscv/bin
```

<br>

# 3. 動作確認
------

```
$ /opt/riscv/bin/riscv64-unknown-elf-gcc --version
riscv64-unknown-elf-gcc (GCC) 10.1.0
Copyright (C) 2020 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

<br>

# 4. 参考文献
------

 - [riscv-gnu-toolchain](https://github.com/riscv/riscv-gnu-toolchain)

