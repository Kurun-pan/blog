---
title: "Zephyr OSを自作RISC-Vで動かした話"
date: 2020-09-22T00:00:00+09:00
bigimg: [{src: "/img/sphere.jpg", desc: ""}]
draft: false
comments: true
tags: ["Zephyr", "RISC-V", "Rust"]
---

Zephyr OSをRustで書いた[自作RISC-Vエミュレータ](https://github.com/Kurun-pan/riscv-emu)で
動かすことに成功したのでその内容をまとめています。
基本的には何か新しいことはしておらず、NuttX OSを動かした時の実装そのままで動きました。

<!--more-->

{{< figure src="./zephyr-logo.png" title="" >}}

## Zephyr OSのビルド
-------

[ここ](https://kurun-pan.github.io/post/2020-09-13/)にまとめている手順でZephyr OSをビルドし、`zephyr.elf`を生成します。

<br>

### Zephyr OSのターゲットハードウェア

SiFive Freedom E310 (FE310) がターゲットハードウェアです。[ここ](https://sifive.cdn.prismic.io/sifive/4d063bf8-3ae6-4db6-9843-ee9076ebadf7_fe310-g000.pdf)にハードウェア仕様があり、自作エミュレータのメモリマップや
ペリフェラルなど、この仕様書をベースに作成しました。

PWMやSPIなどはOSが起動するためには不要であるため、未実装です。GPIOやPLLはブート時にアクセスしていたため、
レジスタだけ用意している感じです。

<br>

## 自作エミュレータで動かしてみる
-------

LinuxやNuttXのようなシェルはないので、地味ですが。

```
$ git clone https://github.com/Kurun-pan/riscv-emu.git
$ cd riscv-emu
$ cargo run --release
$ ./target/release/riscv-emu -k ./artifacts/zephyr/zephyr.elf -m SiFive_e
```

{{< figure src="./zephyr.gif" title="" >}}

<br>
