---
title: "Amazon FreeRTOSのRISC-V向けビルド方法とエミュレータ上での動かし方"
date: 2020-09-06T00:00:00+09:00
bigimg: [{src: "/img/sphere.jpg", desc: ""}]
draft: false
comments: true
tags: ["FreeRTOS"]
---

Amazonが管理するFreeRTOSをRISC-V向けにビルドし、QEMU上で動作させるための手順をまとめています。
ターゲットHost環境は、Linux (Ubuntu18.04で確認) しています。macOSでも試してみましたが、後述の
`Freedom Studio`でRISC-Vクロスコンパラのパスを上手く認識してくれなかったため諦めました。

<!--more-->

## 1. What is FreeRTOS?

[FreeRTOS](https://www.freertos.org/index.html)とは、現在はAmazonが管理している組み込みデバイス向けの小さなリアルタイムOSです。ソースコードは以下で管理されています。
https://github.com/aws/amazon-freertos

<br>

## 2. Getting FreeRTOS source code
-------

シリアルに何らか文字が出るサンプルプログラムが欲しかったため、以下からダウンロードします。
https://www.freertos.org/a00104.html

私が確認したバージョンは、v10.3.1です。
https://github.com/FreeRTOS/FreeRTOS/releases/download/V10.3.1/FreeRTOSv10.3.1.zip

ダウンロード後、zipファイルを解凍して適当な場所に置いておきます。

<br>

## 3. Installing RISC-V toolchain
-------

[ここ](https://kurun-pan.github.io/post/2020-08-16-riscv-toolchain/)の手順に従い、
RISC-Vのtoolchainをインストールします。

<br>

## 4. Installing Freedom Studio
-------

https://www.sifive.com/software から`Freedom Studio`をダウンロード, インストールします。
Ubuntuの場合、ダウンロードデータを解凍し、適当な場所にインストールしてパスを通しておきます。

<br>

## 5. Importing the project
-------

Freeedom Studioを起動します。

```
$ FreedomStudio
```

<br>

メニューの`File` > `Import...`とクリックし、`General` > `Existing Projects intto Workspace`を
選択肢て`Next>`をクリックします。
{{< figure src="./FreedomStudio-00.png" title="" >}}

次の画面で、取得したFreeRTOSソースコードの`FreeTROSv10.3.1/FreeRTOS/Demo/RISC-V-Qemu-sifive_e-Eclipe-GCC`を選択し、Finishをクリックします。
{{< figure src="./FreedomStudio-01.png" title="" >}}

<br>

## 6. Building
-------

メニューの`Project` > `Build All`をクリックし、ビルドします。
しかし、以下のエラーが発生してビルド出来ません。ファイル拡張子が.exeになっている。。

```
/bin/sh: riscv64-unknown-elf-gcc.exe: command not found
```

<br>

Makefile等も自動生成の様子でどこにこのコマンドを指定しているのか分からなかったため、
以下のようにコピーなりシンボリックリンクで対応しました。

```
$ sudo ln -sf /opt/riscv/bin/riscv64-unknown-elf-gcc /opt/riscv/bin/riscv64-unknown-elf-gcc.exe
```

<br>

これでビルドが通るようになります。成功すると`Debug/RTOSDemo.elf`が生成されているはずです。

<br>

## 7. Installing QEMU
-------

QEMUのRISC版を未インストールの場合は以下を実行して下さい。

```
$ git clone https://github.com/qemu/qemu
$ cd qemu/
$ ./configure --target-list=riscv32-softmmu
$ make
$ sudo make install
```

<br>

## 8. Executing on QEMU
-------

以下のコマンドでビルドした`RTOSDemo.elf`を実行します。

```
$ qemu-system-riscv32- kernel Debug/RTOSDemo.elf -machine sifive_e -nographic
StartingBlink
Blink
Blink
...
```

Ctrl+a, xで終了する事が出来ます。


メモ↓。命令トレースなどの情報をファイル出力する方法

```
$ qemu-system-riscv32- kernel Debug/RTOSDemo.elf -machine sifive_e -nographic ¥
  -d int,in_asm -D trace.log
```

<br>

## 9. References
-------

- https://www.freertos.org/Using-FreeRTOS-on-RISC-V.html
- https://www.freertos.org/RTOS-RISC-V-FreedomStudio-QMEU.html

