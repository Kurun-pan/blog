---
title: "RISC-V テストベクター (検証用プログラム) "
date: 2020-08-17T00:00:01+09:00
bigimg: [{src: "/img/sphere.jpg", desc: ""}]
draft: false
comments: true
tags: ["RISC-V"]
---

{{< figure src="/images/riscv.png" title="" >}}

## 1. はじめに
------

RISC-Vの検証用テストプログラムである[`riscv-tests`](https://github.com/riscv/riscv-tests)について情報をまとめました。
RISC-Vエミュレータ開発においても、このテストプログラムがPASSするようにテスト駆動型開発 (TDD) を基本に進めました。

<br>

## 2. riscv-tests ディレクトリ構造
---------

riscv-testsには、`isa`と`benchmarks`の2つが用意されています。`isa`が各命令のテストプログラムで、
`benchmarks`が`dhrystone`やクイックソートなどのベンチマーク用のプログラムです。

[`isa`](https://github.com/riscv/riscv-tests/tree/master/isa)ディレクトリ以下には
以下のようにテスト内容毎にディレクトリが分けられています。

  ディレクトリ名   | Bits | Privilege | 命令の種類
-------|--------|----------|---------
  rv32mi | 32 | Machine | RV32I (基本命令系)
  rv32si | 32 | Supervisor | RV32I (基本命令系)
  rv32ua | 32 | User | RV32A (アトミック命令系)
  rv32uc | 32 | User | RV32C (圧縮命令系)
  rv32ud | 32 | User | RV32D (倍精度 浮動小数点演算命令系)
  rv32uf | 32 | User | RV32F (単精度 浮動小数点演算命令系)
  rv32ui | 32 | User | RV32I (基本命令系)
  rv32um | 32 | User | RV32M (乗除命令系)
  rv64mi | 64 | Machine | RV64I (基本命令系)
  rv64si | 64 | Supervisor | RV64I (基本命令系)
  rv64ua | 64 | User | RV64A (アトミック命令系)
  rv64uc | 64 | User | RV64C (圧縮命令系)
  rv64ud | 64 | User | RV64D (倍精度 浮動小数点演算命令系)
  rv64uf | 64 | User | RV64F (単精度 浮動小数点演算命令系)
  rv64ui | 64 | User | RV64I (基本命令系)
  rv64um | 64 | User | RV64M (乗除命令系)

<br>

## 3. OK/NGの判定方法
---------

`Spike` というRISC-V公式のRISC-V向けの命令セットシミュレータ(ISS)において、
 `Host Target Interface (HTIF) `というハードウェアテスト用の通信仕様が定められています。

risc-testsのテストプログラムは、このHTIFに基づいて、ユーザプログラム終了時に`tohost`という特定の
番地に値を書き込むことで、OSなりHost側に通知 (通信) します。

ベンチマーク用のソースコードだと、benchmarks/common/syscalls.cで以下のように利用されています。

```
static uintptr_t syscall(uintptr_t which, uint64_t arg0, uint64_t arg1, uint64_t arg2)
{
  volatile uint64_t magic_mem[8] __attribute__((aligned(64)));
  magic_mem[0] = which;
  magic_mem[1] = arg0;
  magic_mem[2] = arg1;
  magic_mem[3] = arg2;
  __sync_synchronize();

  tohost = (uintptr_t)magic_mem;
  while (fromhost == 0)
    ;
  fromhost = 0;

  __sync_synchronize();
  return magic_mem[0];
}
```

```
void __attribute__((noreturn)) tohost_exit(uintptr_t code)
{
  tohost = (code << 1) | 1;
  while (1);
}
```

benchmarks/common/crt.S

```
.section ".tohost","aw",@progbits
.align 6
.globl tohost
tohost: .dword 0
.align 6
.globl fromhost
fromhost: .dword 0
```

テストプログラムの終了に関しては、テストプログラムがtohostに値を書き込んで無限ループに入っているだけで、
割り込み等も利用しておらず、Host側でポーリングするしか手段がないように思います。

ということで、RISC-Vエミュレータのテスト用に利用する場合、`.tohost`のセクションを見つけて、そこのアドレスをポーリングすれば良さそうですね。

<br>

# 4. riscv-testsのビルド
---------

ここからはriscv-testsのビルド手順についてです。

なお、当然ですがRISC-Vプロセッサ向けにクロスビルドする場合は、事前に[RISC-V Toolchainのインストール手順](https://kurun-pan.github.io/post/2020-08-16-riscv-toolchain/)の手順に従って、RISC-VのToolchainをインストールしておく必要があります。

カレントディレクトリにビルド結果さえあればいいので、今回はインストールは不要なので行いません。

```
$ git clone https://github.com/riscv/riscv-tests
$ cd riscv-tests
$ git submodule update --init --recursive
$ ./configure
$ make
```

ビルドは比較的すぐに終了し、成功すれば`isa`ディレクトリ直下に`rv32ui-p-add`や、
`benchmarks`ディレクトリ直下に`dhrystone.riscv`などのバイナリが生成されています。

```
$ file benchmarks/dhrystone.riscv
benchmarks/dhrystone.riscv: ELF 64-bit LSB executable, UCB RISC-V, 
version 1 (SYSV), statically linked, not stripped
```

### isaのテストプログラム

rv32ui-`p`-add、rv32ui-`v`-addのように、ハイフン間の識別子は、`p`が物理アドレシング (仮想アドレス無効)、
`v`が仮想アドレス有効なテストであることを示しています。

このようにCPUビットモード、Privilegeモード、アドレッシングモード (仮想アドレスのON/OFF)、命令の種類の組み合わせ
のテストパターンとなります。

<br>

## 5. 参考文献

------

 - https://github.com/riscv/riscv-tests
 - [自作CPU & 自作OSをやっていく (3) - riscv/riscv-tests の挙動を追う](https://laysakura.github.io/2020/02/09/handcraft-cpu-os-3/)
 - [RISC-Vの命令セットテストベンチはどのようになっているのか](https://msyksphinz.hatenablog.com/entry/2015/05/20/020000)
 - [Structure of the RISC-V Software Stack](https://riscv.org/wp-content/uploads/2015/01/riscv-software-stack-bootcamp-jan2015.pdf)
