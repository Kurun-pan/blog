---
title: "Zephyr OSをRISC-V向けにビルドして動かしてみる"
date: 2020-09-13T00:00:00+09:00
bigimg: [{src: "/img/sphere.jpg", desc: ""}]
draft: false
comments: true
tags: ["Zephyr", "RISC-V"]
---

Zephyr OSをRISC-V向けにビルドする手順をまとめています。作業環境はLinux Ubuntu 20.04です。
macOSではビルドエラーになることと、CMakeのバージョンが割と新しいのを求められることからUbuntu 20.04で確認しました。

<!--more-->

## 1. What is Zephyr?
-------

{{< figure src="./Zephyr-logo.png" title="" >}}

[Zephyr](https://www.zephyrproject.org/)とは、VxWorksからforkして2015年から開発されているRTOSの一つらしいです。
Linux Foundationのプロジェクトの一つとのことですが、全く知らなかった…。

<br>

## 2. Installing
-------

以下のパッケージをインストールしておきます。

```
$ sudo apt install --no-install-recommends git cmake ninja-build gperf \
  ccache dfu-util device-tree-compiler wget python3-pip python3-setuptools \
  python3-wheel xz-utils file make gcc gcc-multilib \
  python3-dev libglib2.0-dev libpixman-1-dev
```

<br>

## 3. Getting Zephyr source code
-------

```
$ git clone https://github.com/zephyrproject-rtos/zephyr
```

依存パッケージをインストールします。

```
$ cd zephyr
$ pip3 install --user -r scripts/requirements.txt
```

環境設定を行います。

```
$ export ZEPHYR_TOOLCHAIN_VARIANT=zephyr
$ export ZEPHYR_SDK_INSTALL_DIR="/opt/zephyr-sdk/"
$ . ./zephyr-env.sh
```

<br>

## 4. Install Zephyr SDK
-------

SDKのダウンロードが結構遅くて時間がかかります…。

```
$ wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.11.3/zephyr-sdk-0.11.3-setup.run
$ sudo sh zephyr-sdk-0.11.3-setup.run -- -d $ZEPHYR_SDK_INSTALL_DIR
```

<br>


## 5. Building sample project
-------

```
$ mkdir build-example
$ cd build-example
$ cmake -DBOARD=qemu_riscv32 $ZEPHYR_BASE/samples/hello_world
$ make -j 4
```

ビルドが成功すると`zephyr/zephyr.elf`が生成されているはずです。

<br>

## 6. Executing
-------

```
$ make run
[  3%] Built target parse_syscalls_target
[  4%] Built target kobj_types_h_target
[  7%] Built target syscall_list_h_target
[ 10%] Built target driver_validation_h_target
[ 10%] Built target offsets
[ 11%] Built target offsets_h
[ 11%] Built target zephyr_generated_headers
[ 33%] Built target kernel
[ 34%] Built target app
[ 61%] Built target zephyr
[ 62%] Built target linker_script_target
[ 64%] Built target isr_tables
[ 66%] Built target arch__common
[ 73%] Built target arch__riscv__core
[ 87%] Built target lib__libc__minimal
[ 89%] Built target lib__posix
[ 91%] Built target drivers__gpio
[ 92%] Built target drivers__serial
[ 94%] Built target zephyr_prebuilt
[ 95%] Built target linker_pass_final_script_target
[ 98%] Built target zephyr_final
Scanning dependencies of target run
[100%] 
To exit from QEMU enter: 'CTRL+a, x'
[QEMU] CPU: riscv32
*** Booting Zephyr OS build v2.4.0-rc1-87-g61819f23f97c  ***
Hello World! qemu_riscv32
QEMU: Terminated
[100%] Built target run
```

<br>

## 7. References
-------

- https://risc-v-getting-started-guide.readthedocs.io/en/latest/zephyr-introduction.html

