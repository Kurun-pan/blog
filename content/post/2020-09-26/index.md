---
title: "RISC-V向けブートローダおよびLinuxカーネルのビルド手順"
date: 2020-09-26T00:00:00+09:00
bigimg: [{src: "/img/sphere.jpg", desc: ""}]
draft: false
comments: true
tags: ["Linux", "RISC-V", "BBL", "OpenSBI"]
---

RISC-V向けLinuxカーネルとブートローダのビルド手順についてまとめています。
※本記事ではLinuxのrootfs (BusyBoxなど) のビルドは対象外です。

<!--more-->

## 1. 環境準備
-------

macOS環境だと上手く行きませんでした。そのため、Linux (Ubuntu 20.04) を利用して下さい。

まず、ビルドに必要なパッケージをインストールします。

```
sudo apt install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev \
                 gawk build-essential bison flex texinfo gperf libtool patchutils bc \
                 zlib1g-dev libexpat-dev git libncurses-dev
```

### ツールチェーン

ソースコードを取得して自分でビルドしても良いのですが、時間がかかるため今回は事前にビルドされたバイナリを
インストール方法を紹介します。

https://toolchains.bootlin.com/ からarch = `riscv64`, libc = `glibc` を洗濯してダウンロードします。
ダウンロードしたデータは解凍し、`/opt`以下などの適当な場所に置いてパスを通して下さい。

パスが通っていれば以下のコマンドが実行できるはずです。

```
$ riscv64-linux-gcc -v
```

### 作業スペース

今回は`riscv64-linux`ディレクトリ以下で作業をします。

```
$ mkdir riscv64-linux
$ cd riscv64-linux
```

<br>


## 2. Linux カーネルビルド
-------

### ソースコード取得

```
$ git clone https://github.com/torvalds/linux -b v5.8
```

### ビルド

`menuconfig`では、GUIで特に何か変更は不要です。

```
# cd linux
$ make ARCH=riscv CROSS_COMPILE=riscv64-linux- defconfig
$ make ARCH=riscv CROSS_COMPILE=riscv64-linux- menuconfig
$ make ARCH=riscv CROSS_COMPILE=riscv64-linux- all
```

ビルドが成功すると`arch/riscv/boot/Image`が生成されているはずです。

<br>


## 3. RISC-V 向け Linux ブートローダについて
-------

RISC-VのLinux向けブートローダは`BBL`と`OpenSBI`の2つが存在する様子です。
`OpenSBI`の方が将来の本命な様子のため、基本的には`OpenSBI`を利用しますが、ここでは`BBL`のビルド手順も紹介しておきます。

<br>


## 4. BBL ブートローダのビルド
-------

BBL単体のビルドだけではなく、Linuxカーネルイメージを含んだデータを生成します。

```
$ git clone https://github.com/riscv/riscv-pk.git
$ cd riscv-pk
$ mkdir build
$ cd build
$ ../configure --enable-logo --host=riscv64-linux --with-payload=../../linux/vmlinux
```

ビルドが成功するとカレントディレクトリに`bbl`ファイルが生成されているはずです。
これがブートローダ + Linuxカーネルのイメージファイルです。

<br>


## 5. OpenSBI ブートローダのビルド
-------

```
$ git clone https://github.com/riscv/opensbi.git -b v0.5
$ cd opensbi
```

### QEMU (virt) 向けにビルドする場合

```
$ make CROSS_COMPILE=riscv64-linux- PLATFORM=qemu/virt \
       FW_PAYLOAD_PATH=../linux/arch/riscv/boot/Image
```

ビルドが成功すると`build/platform/qemu/virt/firmware/fw_payload.elf`が生成されているはずです。

### SiFive FU540 SoC 向けにビルドする場合

```
$ make CROSS_COMPILE=riscv64-linux- PLATFORM=sifive/fu540 \
       FW_PAYLOAD_PATH=../linux/arch/riscv/boot/Image
```

ビルドが成功すると`build/platform/sifive/fu540/firmware/fw_payload.elf`が生成されているはずです。

<br>


## 6. QEMU で実行
-------

事前にQEMUのビルドやインストールが必要です！また、今回はrootfsを用意していないので、rootfsをマウントするところでエラーになります（意図通り）。

### OpenSBI の場合

```
$ cd opensbi
$ qemu-system-riscv64 -M virt -m 256M -nographic -bios build/platform/qemu/virt/firmware/fw_payload.bin
OpenSBI v0.5 (Sep 22 2020 04:15:03)
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|____/_____|
        | |
        |_|

Platform Name          : QEMU Virt Machine
Platform HART Features : RV64ACDFIMSU
Platform Max HARTs     : 8
Current Hart           : 0
Firmware Base          : 0x80000000
Firmware Size          : 116 KB
Runtime SBI Version    : 0.2

PMP0: 0x0000000080000000-0x000000008001ffff (A)
PMP1: 0x0000000000000000-0xffffffffffffffff (A,R,W,X)
[    0.000000] OF: fdt: Ignoring memory range 0x80000000 - 0x80200000
[    0.000000] Linux version 5.8.0 (hide@ubuntu) (riscv64-linux-gcc.br_real (Buildroot 2020.02-00011-g7ea8a52) 9.3.0, GNU ld (GNU Binutils) 2.33.1) #2 SMP Sat Sep 26 02:20:01 PDT 2020
[    0.000000] Zone ranges:
[    0.000000]   DMA32    [mem 0x0000000080200000-0x000000008fffffff]
[    0.000000]   Normal   empty
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x0000000080200000-0x000000008fffffff]
[    0.000000] Initmem setup node 0 [mem 0x0000000080200000-0x000000008fffffff]
[    0.000000] software IO TLB: mapped [mem 0x8bc7a000-0x8fc7a000] (64MB)
[    0.000000] SBI specification v0.2 detected
[    0.000000] SBI implementation ID=0x1 Version=0x5
[    0.000000] riscv: ISA extensions acdfimsu
[    0.000000] riscv: ELF capabilities acdfim
[    0.000000] percpu: Embedded 17 pages/cpu s31976 r8192 d29464 u69632
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 64135
[    0.000000] Kernel command line: 
[    0.000000] Dentry cache hash table entries: 32768 (order: 6, 262144 bytes, linear)
[    0.000000] Inode-cache hash table entries: 16384 (order: 5, 131072 bytes, linear)
[    0.000000] Sorting __ex_table...
[    0.000000] mem auto-init: stack:off, heap alloc:off, heap free:off
[    0.000000] Memory: 172956K/260096K available (6419K kernel code, 4267K rwdata, 4096K rodata, 235K init, 317K bss, 87140K reserved, 0K cma-reserved)
[    0.000000] Virtual kernel memory layout:
[    0.000000]       fixmap : 0xffffffcefee00000 - 0xffffffceff000000   (2048 kB)
[    0.000000]       pci io : 0xffffffceff000000 - 0xffffffcf00000000   (  16 MB)
[    0.000000]      vmemmap : 0xffffffcf00000000 - 0xffffffcfffffffff   (4095 MB)
[    0.000000]      vmalloc : 0xffffffd000000000 - 0xffffffdfffffffff   (65535 MB)
[    0.000000]       lowmem : 0xffffffe000000000 - 0xffffffe00fe00000   ( 254 MB)
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
[    0.000000] rcu: Hierarchical RCU implementation.
[    0.000000] rcu: 	RCU restricting CPUs from NR_CPUS=8 to nr_cpu_ids=1.
[    0.000000] rcu: 	RCU debug extended QS entry/exit.
[    0.000000] rcu: RCU calculated value of scheduler-enlistment delay is 25 jiffies.
[    0.000000] rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=1
[    0.000000] NR_IRQS: 64, nr_irqs: 64, preallocated irqs: 0
[    0.000000] riscv-intc: 64 local interrupts mapped
[    0.000000] plic: interrupt-controller@c000000: mapped 53 interrupts with 1 handlers for 2 contexts.
[    0.000000] riscv_timer_init_dt: Registering clocksource cpuid [0] hartid [0]
[    0.000000] clocksource: riscv_clocksource: mask: 0xffffffffffffffff max_cycles: 0x24e6a1710, max_idle_ns: 440795202120 ns
[    0.000155] sched_clock: 64 bits at 10MHz, resolution 100ns, wraps every 4398046511100ns
[    0.003476] Console: colour dummy device 80x25
[    0.005342] printk: console [tty0] enabled
[    0.009004] Calibrating delay loop (skipped), value calculated using timer frequency.. 20.00 BogoMIPS (lpj=40000)
[    0.009199] pid_max: default: 32768 minimum: 301
[    0.010564] Mount-cache hash table entries: 512 (order: 0, 4096 bytes, linear)
[    0.010693] Mountpoint-cache hash table entries: 512 (order: 0, 4096 bytes, linear)
[    0.050774] rcu: Hierarchical SRCU implementation.
[    0.054706] smp: Bringing up secondary CPUs ...
[    0.054835] smp: Brought up 1 node, 1 CPU
[    0.080141] devtmpfs: initialized
[    0.086231] random: get_random_u32 called from bucket_table_alloc.isra.0+0x4e/0x154 with crng_init=0
[    0.088885] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
[    0.089190] futex hash table entries: 256 (order: 2, 16384 bytes, linear)
[    0.094861] NET: Registered protocol family 16
[    0.168359] vgaarb: loaded
[    0.169839] SCSI subsystem initialized
[    0.172513] usbcore: registered new interface driver usbfs
[    0.172916] usbcore: registered new interface driver hub
[    0.173127] usbcore: registered new device driver usb
[    0.183662] clocksource: Switched to clocksource riscv_clocksource
[    0.205743] NET: Registered protocol family 2
[    0.210103] tcp_listen_portaddr_hash hash table entries: 128 (order: 0, 5120 bytes, linear)
[    0.210285] TCP established hash table entries: 2048 (order: 2, 16384 bytes, linear)
[    0.210552] TCP bind hash table entries: 2048 (order: 4, 65536 bytes, linear)
[    0.210770] TCP: Hash tables configured (established 2048 bind 2048)
[    0.228852] UDP hash table entries: 256 (order: 2, 24576 bytes, linear)
[    0.244265] UDP-Lite hash table entries: 256 (order: 2, 24576 bytes, linear)
[    0.245937] NET: Registered protocol family 1
[    0.249126] RPC: Registered named UNIX socket transport module.
[    0.249214] RPC: Registered udp transport module.
[    0.249259] RPC: Registered tcp transport module.
[    0.249304] RPC: Registered tcp NFSv4.1 backchannel transport module.
[    0.249434] PCI: CLS 0 bytes, default 64
[    0.256501] workingset: timestamp_bits=62 max_order=16 bucket_order=0
[    0.279852] NFS: Registering the id_resolver key type
[    0.280889] Key type id_resolver registered
[    0.280964] Key type id_legacy registered
[    0.281120] nfs4filelayout_init: NFSv4 File Layout Driver Registering...
[    0.281986] 9p: Installing v9fs 9p2000 file system support
[    0.283780] NET: Registered protocol family 38
[    0.284118] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
[    0.284279] io scheduler mq-deadline registered
[    0.284407] io scheduler kyber registered
[    0.297250] pci-host-generic 30000000.pci: host bridge /soc/pci@30000000 ranges:
[    0.297977] pci-host-generic 30000000.pci:       IO 0x0003000000..0x000300ffff -> 0x0000000000
[    0.298407] pci-host-generic 30000000.pci:      MEM 0x0040000000..0x007fffffff -> 0x0040000000
[    0.300672] pci-host-generic 30000000.pci: ECAM at [mem 0x30000000-0x3fffffff] for [bus 00-ff]
[    0.301735] pci-host-generic 30000000.pci: PCI host bridge to bus 0000:00
[    0.301960] pci_bus 0000:00: root bus resource [bus 00-ff]
[    0.302091] pci_bus 0000:00: root bus resource [io  0x0000-0xffff]
[    0.302142] pci_bus 0000:00: root bus resource [mem 0x40000000-0x7fffffff]
[    0.303051] pci 0000:00:00.0: [1b36:0008] type 00 class 0x060000
[    0.429734] Serial: 8250/16550 driver, 4 ports, IRQ sharing disabled
[    0.452983] 10000000.uart: ttyS0 at MMIO 0x10000000 (irq = 2, base_baud = 230400) is a 16550A
[    0.542118] printk: console [ttyS0] enabled
[    0.545614] [drm] radeon kernel modesetting enabled.
[    0.585441] loop: module loaded
[    0.589492] libphy: Fixed MDIO Bus: probed
[    0.591904] e1000e: Intel(R) PRO/1000 Network Driver - 3.2.6-k
[    0.592556] e1000e: Copyright(c) 1999 - 2015 Intel Corporation.
[    0.593457] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
[    0.593989] ehci-pci: EHCI PCI platform driver
[    0.594774] ehci-platform: EHCI generic platform driver
[    0.595815] ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
[    0.596669] ohci-pci: OHCI PCI platform driver
[    0.597384] ohci-platform: OHCI generic platform driver
[    0.599288] usbcore: registered new interface driver uas
[    0.600669] usbcore: registered new interface driver usb-storage
[    0.602612] mousedev: PS/2 mouse device common for all mice
[    0.607115] goldfish_rtc 101000.rtc: registered as rtc0
[    0.608599] goldfish_rtc 101000.rtc: setting system clock to 2020-09-26T14:44:40 UTC (1601131480)
[    0.612081] syscon-poweroff poweroff: pm_power_off already claimed (____ptrval____) sbi_shutdown
[    0.613206] syscon-poweroff: probe of poweroff failed with error -16
[    0.615734] usbcore: registered new interface driver usbhid
[    0.616330] usbhid: USB HID core driver
[    0.618869] NET: Registered protocol family 10
[    0.628796] Segment Routing with IPv6
[    0.629603] sit: IPv6, IPv4 and MPLS over IPv4 tunneling driver
[    0.633730] NET: Registered protocol family 17
[    0.635933] 9pnet: Installing 9P2000 support
[    0.636798] Key type dns_resolver registered
[    0.666749] VFS: Cannot open root device "(null)" or unknown-block(0,0): error -6
[    0.667965] Please append a correct "root=" boot option; here are the available partitions:
[    0.668465] DEBUG_BLOCK_EXT_DEVT is enabled, you need to specify explicit textual name for "root=" boot option.
[    0.669085] Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
[    0.669682] CPU: 0 PID: 1 Comm: swapper/0 Not tainted 5.8.0 #2
[    0.670101] Call Trace:
[    0.670441] [<ffffffe0002025c0>] walk_stackframe+0x0/0xaa
[    0.670802] [<ffffffe0002027ae>] show_stack+0x2e/0x3a
[    0.671120] [<ffffffe000466e24>] dump_stack+0x74/0x8e
[    0.671560] [<ffffffe0002082a0>] panic+0xf0/0x276
[    0.672025] [<ffffffe000002126>] mount_block_root+0x1ac/0x22a
[    0.672408] [<ffffffe0000022c6>] mount_root+0x122/0x138
[    0.672771] [<ffffffe000002418>] prepare_namespace+0x13c/0x17a
[    0.673191] [<ffffffe000001d04>] kernel_init_freeable+0x1ba/0x1d6
[    0.673975] [<ffffffe00083edae>] kernel_init+0x12/0x120
[    0.674410] [<ffffffe00020131e>] ret_from_exception+0x0/0xc
[    0.675205] ---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0) ]---
```

```
$ cd opensbi
$ qemu-system-riscv64 -M sifive_u -m 256M -nographic -bios build/platform/sifive/fu540/firmware/fw_payload.bin
```

### BBL の場合

```
$ cd riscv-pk
$ qemu-system-riscv64 -M virt -m 256M -nographic -bios bbl
bbl loader
              vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv
                  vvvvvvvvvvvvvvvvvvvvvvvvvvvv
rrrrrrrrrrrrr       vvvvvvvvvvvvvvvvvvvvvvvvvv
rrrrrrrrrrrrrrrr      vvvvvvvvvvvvvvvvvvvvvvvv
rrrrrrrrrrrrrrrrrr    vvvvvvvvvvvvvvvvvvvvvvvv
rrrrrrrrrrrrrrrrrr    vvvvvvvvvvvvvvvvvvvvvvvv
rrrrrrrrrrrrrrrrrr    vvvvvvvvvvvvvvvvvvvvvvvv
rrrrrrrrrrrrrrrr      vvvvvvvvvvvvvvvvvvvvvv  
rrrrrrrrrrrrr       vvvvvvvvvvvvvvvvvvvvvv    
rr                vvvvvvvvvvvvvvvvvvvvvv      
rr            vvvvvvvvvvvvvvvvvvvvvvvv      rr
rrrr      vvvvvvvvvvvvvvvvvvvvvvvvvv      rrrr
rrrrrr      vvvvvvvvvvvvvvvvvvvvvv      rrrrrr
rrrrrrrr      vvvvvvvvvvvvvvvvvv      rrrrrrrr
rrrrrrrrrr      vvvvvvvvvvvvvv      rrrrrrrrrr
rrrrrrrrrrrr      vvvvvvvvvv      rrrrrrrrrrrr
rrrrrrrrrrrrrr      vvvvvv      rrrrrrrrrrrrrr
rrrrrrrrrrrrrrrr      vv      rrrrrrrrrrrrrrrr
rrrrrrrrrrrrrrrrrr          rrrrrrrrrrrrrrrrrr
rrrrrrrrrrrrrrrrrrrr      rrrrrrrrrrrrrrrrrrrr
rrrrrrrrrrrrrrrrrrrrrr  rrrrrrrrrrrrrrrrrrrrrr

       INSTRUCTION SETS WANT TO BE FREE
[    0.000000] OF: fdt: Ignoring memory range 0x80000000 - 0x80200000
[    0.000000] Linux version 5.8.0 (hide@ubuntu) (riscv64-linux-gcc.br_real (Buildroot 2020.02-00011-g7ea8a52) 9.3.0, GNU ld (GNU Binutils) 2.33.1) #2 SMP Sat Sep 26 02:20:01 PDT 2020
[    0.000000] Zone ranges:
[    0.000000]   DMA32    [mem 0x0000000080200000-0x000000008fffffff]
[    0.000000]   Normal   empty
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x0000000080200000-0x000000008fffffff]
[    0.000000] Initmem setup node 0 [mem 0x0000000080200000-0x000000008fffffff]
[    0.000000] software IO TLB: mapped [mem 0x8bc7a000-0x8fc7a000] (64MB)
[    0.000000] SBI specification v0.1 detected
[    0.000000] riscv: ISA extensions acdfimsu
[    0.000000] riscv: ELF capabilities acdfim
[    0.000000] percpu: Embedded 17 pages/cpu s31976 r8192 d29464 u69632
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 64135
[    0.000000] Kernel command line: 
[    0.000000] Dentry cache hash table entries: 32768 (order: 6, 262144 bytes, linear)
[    0.000000] Inode-cache hash table entries: 16384 (order: 5, 131072 bytes, linear)
[    0.000000] Sorting __ex_table...
[    0.000000] mem auto-init: stack:off, heap alloc:off, heap free:off
[    0.000000] Memory: 172956K/260096K available (6419K kernel code, 4267K rwdata, 4096K rodata, 235K init, 317K bss, 87140K reserved, 0K cma-reserved)
[    0.000000] Virtual kernel memory layout:
[    0.000000]       fixmap : 0xffffffcefee00000 - 0xffffffceff000000   (2048 kB)
[    0.000000]       pci io : 0xffffffceff000000 - 0xffffffcf00000000   (  16 MB)
[    0.000000]      vmemmap : 0xffffffcf00000000 - 0xffffffcfffffffff   (4095 MB)
[    0.000000]      vmalloc : 0xffffffd000000000 - 0xffffffdfffffffff   (65535 MB)
[    0.000000]       lowmem : 0xffffffe000000000 - 0xffffffe00fe00000   ( 254 MB)
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
[    0.000000] rcu: Hierarchical RCU implementation.
[    0.000000] rcu: 	RCU restricting CPUs from NR_CPUS=8 to nr_cpu_ids=1.
[    0.000000] rcu: 	RCU debug extended QS entry/exit.
[    0.000000] rcu: RCU calculated value of scheduler-enlistment delay is 25 jiffies.
[    0.000000] rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=1
[    0.000000] NR_IRQS: 64, nr_irqs: 64, preallocated irqs: 0
[    0.000000] riscv-intc: 64 local interrupts mapped
[    0.000000] plic: interrupt-controller@c000000: mapped 53 interrupts with 1 handlers for 2 contexts.
[    0.000000] riscv_timer_init_dt: Registering clocksource cpuid [0] hartid [0]
[    0.000000] clocksource: riscv_clocksource: mask: 0xffffffffffffffff max_cycles: 0x24e6a1710, max_idle_ns: 440795202120 ns
[    0.000221] sched_clock: 64 bits at 10MHz, resolution 100ns, wraps every 4398046511100ns
[    0.003576] Console: colour dummy device 80x25
[    0.005421] printk: console [tty0] enabled
[    0.009015] Calibrating delay loop (skipped), value calculated using timer frequency.. 20.00 BogoMIPS (lpj=40000)
[    0.009253] pid_max: default: 32768 minimum: 301
[    0.010701] Mount-cache hash table entries: 512 (order: 0, 4096 bytes, linear)
[    0.010793] Mountpoint-cache hash table entries: 512 (order: 0, 4096 bytes, linear)
[    0.036955] rcu: Hierarchical SRCU implementation.
[    0.040934] smp: Bringing up secondary CPUs ...
[    0.041082] smp: Brought up 1 node, 1 CPU
[    0.050040] devtmpfs: initialized
[    0.056179] random: get_random_u32 called from bucket_table_alloc.isra.0+0x4e/0x154 with crng_init=0
[    0.058833] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
[    0.059152] futex hash table entries: 256 (order: 2, 16384 bytes, linear)
[    0.064733] NET: Registered protocol family 16
[    0.123552] vgaarb: loaded
[    0.124980] SCSI subsystem initialized
[    0.127460] usbcore: registered new interface driver usbfs
[    0.127821] usbcore: registered new interface driver hub
[    0.128078] usbcore: registered new device driver usb
[    0.138868] clocksource: Switched to clocksource riscv_clocksource
[    0.160708] NET: Registered protocol family 2
[    0.164900] tcp_listen_portaddr_hash hash table entries: 128 (order: 0, 5120 bytes, linear)
[    0.165137] TCP established hash table entries: 2048 (order: 2, 16384 bytes, linear)
[    0.165384] TCP bind hash table entries: 2048 (order: 4, 65536 bytes, linear)
[    0.165585] TCP: Hash tables configured (established 2048 bind 2048)
[    0.166599] UDP hash table entries: 256 (order: 2, 24576 bytes, linear)
[    0.184552] UDP-Lite hash table entries: 256 (order: 2, 24576 bytes, linear)
[    0.201699] NET: Registered protocol family 1
[    0.204921] RPC: Registered named UNIX socket transport module.
[    0.205005] RPC: Registered udp transport module.
[    0.205090] RPC: Registered tcp transport module.
[    0.205136] RPC: Registered tcp NFSv4.1 backchannel transport module.
[    0.205283] PCI: CLS 0 bytes, default 64
[    0.212247] workingset: timestamp_bits=62 max_order=16 bucket_order=0
[    0.232139] NFS: Registering the id_resolver key type
[    0.233803] Key type id_resolver registered
[    0.233875] Key type id_legacy registered
[    0.234057] nfs4filelayout_init: NFSv4 File Layout Driver Registering...
[    0.234772] 9p: Installing v9fs 9p2000 file system support
[    0.236435] NET: Registered protocol family 38
[    0.236749] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
[    0.236898] io scheduler mq-deadline registered
[    0.237054] io scheduler kyber registered
[    0.244990] pci-host-generic 30000000.pci: host bridge /soc/pci@30000000 ranges:
[    0.245671] pci-host-generic 30000000.pci:       IO 0x0003000000..0x000300ffff -> 0x0000000000
[    0.246127] pci-host-generic 30000000.pci:      MEM 0x0040000000..0x007fffffff -> 0x0040000000
[    0.247932] pci-host-generic 30000000.pci: ECAM at [mem 0x30000000-0x3fffffff] for [bus 00-ff]
[    0.248859] pci-host-generic 30000000.pci: PCI host bridge to bus 0000:00
[    0.249103] pci_bus 0000:00: root bus resource [bus 00-ff]
[    0.249224] pci_bus 0000:00: root bus resource [io  0x0000-0xffff]
[    0.249272] pci_bus 0000:00: root bus resource [mem 0x40000000-0x7fffffff]
[    0.250118] pci 0000:00:00.0: [1b36:0008] type 00 class 0x060000
[    0.370967] Serial: 8250/16550 driver, 4 ports, IRQ sharing disabled
[    0.394217] 10000000.uart: ttyS0 at MMIO 0x10000000 (irq = 2, base_baud = 230400) is a 16550A
[    0.480523] printk: console [ttyS0] enabled
[    0.483689] [drm] radeon kernel modesetting enabled.
[    0.527410] loop: module loaded
[    0.531287] libphy: Fixed MDIO Bus: probed
[    0.533463] e1000e: Intel(R) PRO/1000 Network Driver - 3.2.6-k
[    0.533983] e1000e: Copyright(c) 1999 - 2015 Intel Corporation.
[    0.535241] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
[    0.535830] ehci-pci: EHCI PCI platform driver
[    0.536627] ehci-platform: EHCI generic platform driver
[    0.537257] ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
[    0.537801] ohci-pci: OHCI PCI platform driver
[    0.538360] ohci-platform: OHCI generic platform driver
[    0.541217] usbcore: registered new interface driver uas
[    0.541918] usbcore: registered new interface driver usb-storage
[    0.543451] mousedev: PS/2 mouse device common for all mice
[    0.547875] goldfish_rtc 101000.rtc: registered as rtc0
[    0.549087] goldfish_rtc 101000.rtc: setting system clock to 2020-09-26T14:45:50 UTC (1601131550)
[    0.552660] syscon-poweroff poweroff: pm_power_off already claimed (____ptrval____) sbi_shutdown
[    0.553512] syscon-poweroff: probe of poweroff failed with error -16
[    0.556474] usbcore: registered new interface driver usbhid
[    0.557038] usbhid: USB HID core driver
[    0.559709] NET: Registered protocol family 10
[    0.569948] Segment Routing with IPv6
[    0.570710] sit: IPv6, IPv4 and MPLS over IPv4 tunneling driver
[    0.574384] NET: Registered protocol family 17
[    0.576211] 9pnet: Installing 9P2000 support
[    0.576946] Key type dns_resolver registered
[    0.607161] VFS: Cannot open root device "(null)" or unknown-block(0,0): error -6
[    0.608052] Please append a correct "root=" boot option; here are the available partitions:
[    0.609049] DEBUG_BLOCK_EXT_DEVT is enabled, you need to specify explicit textual name for "root=" boot option.
[    0.609958] Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
[    0.610862] CPU: 0 PID: 1 Comm: swapper/0 Not tainted 5.8.0 #2
[    0.611425] Call Trace:
[    0.612170] [<ffffffe0002025c0>] walk_stackframe+0x0/0xaa
[    0.612687] [<ffffffe0002027ae>] show_stack+0x2e/0x3a
[    0.613206] [<ffffffe000466e24>] dump_stack+0x74/0x8e
[    0.613666] [<ffffffe0002082a0>] panic+0xf0/0x276
[    0.614154] [<ffffffe000002126>] mount_block_root+0x1ac/0x22a
[    0.614884] [<ffffffe0000022c6>] mount_root+0x122/0x138
[    0.615310] [<ffffffe000002418>] prepare_namespace+0x13c/0x17a
[    0.615874] [<ffffffe000001d04>] kernel_init_freeable+0x1ba/0x1d6
[    0.616377] [<ffffffe00083edae>] kernel_init+0x12/0x120
[    0.616801] [<ffffffe00020131e>] ret_from_exception+0x0/0xc
[    0.618354] ---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0) ]---
```

<br>

## 7. References
-------

- https://risc-v-getting-started-guide.readthedocs.io/en/latest/linux-introduction.html
- https://github.com/riscv/opensbi/blob/master/docs/platform/sifive_fu540.md
- https://msyksphinz.hatenablog.com/entry/2020/05/11/040000
- https://msyksphinz.hatenablog.com/entry/2018/05/08/040000
- https://cstmize.hatenablog.jp/entry/2019/10/14/_QEMU%2BOpenSBI%28boot_loader%29%E3%81%A7linux_kernel%E3%81%AE%E8%B5%B7%E5%8B%95
