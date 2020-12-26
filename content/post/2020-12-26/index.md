---
title: "Linux Device Treeは難しい"
date: 2020-12-26T00:00:00+09:00
bigimg: [{src: "/img/sphere.jpg", desc: ""}]
draft: false
comments: true
tags: ["Linux", "RISC-V", "Device Tree"]
---

Linuxのデバイスツリーは難しいというお話。

<!--more-->

## Device Treeは謎が多い
-------

[RISC-Vエミュ](https://github.com/Kurun-pan/riscv-emu) でLinuxカーネルを動かすためにデバイスツリーを作成しているのですが、
UARTの割り込みIDがデバイスツリーで指定した値にならなくてハマりました。

問題がないデバイスツリーのファイルは[githubにコミットしているファイル](https://github.com/Kurun-pan/riscv-emu/blob/master/artifacts/linux/dtb/qemu_virtio.dts)で、以下の通りです。

#### 正しい内容

```
/dts-v1/;

/ {
    #address-cells = <2>;
    #size-cells = <2>;
    compatible = "riscv-virtio";
    model = "RISCV Qemu Virtio";

    chosen {
        bootargs = "root=/dev/vda rw ttyS0";
        stdout-path = "/uart@10000000";
    };

    uart@10000000 {
        compatible = "ns16550a";
        reg = <0x0 0x10000000 0x0 0x100>;
        clock-frequency = <0x384000>;
        interrupts = <10>;
        interrupt-parent = <&intc>;
    };

    virtio_mmio@10001000 {
        compatible = "virtio,mmio";
        reg = <0x0 0x10001000 0x0 0x1000>;
        interrupts = <1>;
        interrupt-parent = <&intc>;
    };

    cpus {
        #address-cells = <1>;
        #size-cells = <0>;
        timebase-frequency = <1000000>;
        cpu@0 {
            device_type = "cpu";
            compatible = "riscv";
			riscv,isa = "rv64abcdfimnsu";
            mmu-type = "riscv,sv39";
            reg = <0>;
			clock-frequency = <0>;
            status = "okay";
            vic: interrupt-controller {
                compatible = "riscv,cpu-intc";
                #interrupt-cells = <1>;
                interrupt-controller;
            };
        };
    };

    intc: intc@c000000 {
        compatible = "riscv,plic0";
        #interrupt-cells = <1>;
        interrupt-controller;
        reg = <0x0 0xc000000 0x0 0x4000000>;
        interrupts-extended = <&vic 11 &vic 9>;
        riscv,ndev = <0x35>;
    };

    memory@80000000 {
        device_type = "memory";
        reg = <0x0 0x80000000 0x0 0x8000000>;
    };

    clint@2000000 {
        compatible = "riscv,clint0";
        reg = <0x0 0x2000000 0x0 0x10000>;
        interrupts-extended = <&vic 3 &vic 7>;
    };
};
```

<br>

#### 正しくない内容（割り込み番号が指定した値にならない）

上記を以下のように、`uart@10000000`と`virtio_mmio@10001000`の位置をファイルの下の方に移動させると、
`interrupts`で指定した割り込み番号が設定値通りになりませんでした。
dtsのコンパイル時には何もワーニング等は何も出てきませんが、記述の順序にルールがあるのかなぁ。。

```
/dts-v1/;

/ {
    #address-cells = <2>;
    #size-cells = <2>;
    compatible = "riscv-virtio";
    model = "RISCV Qemu Virtio";

    chosen {
        bootargs = "root=/dev/vda rw ttyS0";
        stdout-path = "/uart@10000000";
    };

    cpus {
        #address-cells = <1>;
        #size-cells = <0>;
        timebase-frequency = <1000000>;
        cpu@0 {
            device_type = "cpu";
            compatible = "riscv";
			riscv,isa = "rv64abcdfimnsu";
            mmu-type = "riscv,sv39";
            reg = <0>;
			clock-frequency = <0>;
            status = "okay";
            vic: interrupt-controller {
                compatible = "riscv,cpu-intc";
                #interrupt-cells = <1>;
                interrupt-controller;
            };
        };
    };

    intc: intc@c000000 {
        compatible = "riscv,plic0";
        #interrupt-cells = <1>;
        interrupt-controller;
        reg = <0x0 0xc000000 0x0 0x4000000>;
        interrupts-extended = <&vic 11 &vic 9>;
        riscv,ndev = <0x35>;
    };

    memory@80000000 {
        device_type = "memory";
        reg = <0x0 0x80000000 0x0 0x8000000>;
    };

    clint@2000000 {
        compatible = "riscv,clint0";
        reg = <0x0 0x2000000 0x0 0x10000>;
        interrupts-extended = <&vic 3 &vic 7>;
    };

    uart@10000000 {
        compatible = "ns16550a";
        reg = <0x0 0x10000000 0x0 0x100>;
        clock-frequency = <0x384000>;
        interrupts = <10>;
        interrupt-parent = <&intc>;
    };

    virtio_mmio@10001000 {
        compatible = "virtio,mmio";
        reg = <0x0 0x10001000 0x0 0x1000>;
        interrupts = <1>;
        interrupt-parent = <&intc>;
    };
};
```

<br>

## まとめ
-------

デバイスツリーの記述はよく分からん。
