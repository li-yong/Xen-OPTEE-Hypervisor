
# Get the U-Boot source
```
cd $WORK_DIR
wget https://ftp.denx.de/pub/u-boot/u-boot-2024.07.tar.bz2
tar xf u-boot-2024.07.tar.bz2
```

## Make config
```
cd $WORK_DIR/u-boot-2024.07
make CROSS_COMPILE=aarch64-linux-gnu- qemu_arm64_defconfig
```

## Edit build config
```
CONFIG_ARCH_QEMU=y
CONFIG_TARGET_QEMU_ARM_64BIT=y
```

# Build u-boot
```
make CROSS_COMPILE=aarch64-linux-gnu- -j4
```


## Copy Binary
```
cp u-boot.bin $BUILD_DIR/busybox_arm64/
cd $BUILD_DIR/busybox_arm64/
```

## Verify run u-boot with Linux
```
gunzip Image.gz

qemu-system-aarch64 \
    -machine virt,gic_version=3 \
    -machine virtualization=true \
    -cpu cortex-a57 \
    -machine type=virt \
    -m 512M \
    -bios u-boot.bin \
    -device loader,file=Image,addr=0x45000000 \
    -nographic \
    -no-reboot \
    -chardev socket,id=qemu-monitor,host=localhost,port=7777,server,nowait,telnet \
    -mon qemu-monitor,mode=readline

```

Console will output something as illustrated and stop at the U-boot prompt.
```
U-Boot 2024.07 (Sep 26 2024 - 21:54:01 -0400)

DRAM:  512 MiB
Core:  51 devices, 14 uclasses, devicetree: board
...
...
...

No more bootdevs
---  -----------  ------  --------  ----  ------------------------  ----------------
(0 bootflows, 0 valid)
=> 
```

## Load Linux by U-Boot.
```
booti 0x45000000 - 0x40000000
```

Output. Note the kernel panic on mounting root fs because no root fs was specified this time.
```
=> booti 0x45000000 - 0x40000000
## Flattened Device Tree blob at 40000000
   Booting using the fdt blob at 0x40000000
Working FDT set to 40000000
   Loading Device Tree to 000000005d45e000, end 000000005d560fff ... OK
Working FDT set to 5d45e000

Starting kernel ...

[    0.000000] Booting Linux on physical CPU 0x0000000000 [0x411fd070]
[    0.000000] Linux version 6.1.18 (ryan@ryan-Precision-7510) (aarch64-linux-gnu-gcc (Ubuntu 13.2.0-23ubuntu4) 13.2.0, GNU ld (GNU Binutils for Ubuntu) 2.42) #2 SMP PREEMPT Thu Sep 26 23:39:05 EDT 2024
...
...
...
[    0.851399] Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
[    0.852250] CPU: 0 PID: 1 Comm: swapper/0 Not tainted 6.1.18 #2
[    0.852681] Hardware name: linux,dummy-virt (DT)
[    0.853214] Call trace:
[    0.853541]  dump_backtrace.part.0+0xdc/0xf0
[    0.853959]  show_stack+0x18/0x30
[    0.854300]  dump_stack_lvl+0x64/0x80
[    0.854535]  dump_stack+0x18/0x34
[    0.854764]  panic+0x188/0x344
[    0.854953]  mount_block_root+0x188/0x238
[    0.855166]  mount_root+0x214/0x254
[    0.855401]  prepare_namespace+0x12c/0x16c
[    0.855657]  kernel_init_freeable+0x258/0x284
[    0.855921]  kernel_init+0x24/0x12c
[    0.856176]  ret_from_fork+0x10/0x20
[    0.856873] Kernel Offset: 0x49bc48c00000 from 0xffff800008000000
[    0.857212] PHYS_OFFSET: 0xffffd2a980000000
[    0.857469] CPU features: 0x22000,2033c084,0000421b
[    0.857965] Memory Limit: none
[    0.858567] ---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0) ]---

```