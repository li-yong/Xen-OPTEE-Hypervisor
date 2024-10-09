

# generate a device tree blob
```
cd $BUILD_DIR/busybox_arm64
```

# Get the files size in Hex 
```
printf "0x%x\n" $(stat -c %s Image.gz)
0xbe1581

printf "0x%x\n" $(stat -c %s rootfs.img.gz)
0x122a54
```

# Lunch Xen+Linux(Dom0) under Qemu
```
$ qemu-system-aarch64 \
    -machine virt,gic_version=3 \
    -machine virtualization=true \
    -cpu cortex-a57 \
    -machine type=virt \
    -m 4096 \
    -smp 4 \
    -bios u-boot.bin \
    -device loader,file=xen,force-raw=on,addr=0x49000000 \
    -device loader,file=Image.gz,addr=0x47000000 \
    -device loader,file=virt-gicv3.dtb,addr=0x44000000 \
    -device loader,file=rootfs.img.gz,addr=0x42000000 \
    -nographic \
    -no-reboot \
    -chardev socket,id=qemu-monitor,host=localhost,port=7777,server,nowait,telnet \
    -mon qemu-monitor,mode=readline
```

Then in U-Boot prompt, 
```
fdt addr 0x44000000
fdt resize
fdt set /chosen \#address-cells <1>
fdt set /chosen \#size-cells <1>
fdt mknod /chosen module@0
fdt set /chosen/module@0 compatible "xen,linux-zimage" "xen,multiboot-module"
fdt set /chosen/module@0 reg <0x47000000 0xbe1581>
fdt set /chosen/module@0 bootargs "earlyprintk=serial,ttyAMA0
console=ttyAMA0,115200n8 earlycon=xenboot"
booti 0x49000000 - 0x44000000
```

## Output Review

Starting XEN
```
=> booti 0x49000000 - 0x44000000
## Flattened Device Tree blob at 44000000
   Booting using the fdt blob at 0x44000000
Working FDT set to 44000000
   Loading Device Tree to 00000000ffffa000, end 00000000ffffefff ... OK
Working FDT set to ffffa000

Starting kernel ...

 Xen 4.18.3
(XEN) Xen version 4.18.3 (ryan@) (aarch64-linux-gnu-gcc (Ubuntu 13.2.0-23ubuntu4) 13.2.0) debug=n Fri Sep 27 22:52:12 EDT 2024
(XEN) Latest ChangeSet: 
(XEN) build-id: 8cd9a148444063cc4d6a62169a0866cf91705f4d

```

XEN loading Linux
```
(XEN) ***************************************************
(XEN) 3... 2... 1... 
(XEN) *** Serial input to DOM0 (type 'CTRL-a' three times to switch input)
(XEN) Freed 364kB init memory.
[    0.000000] Booting Linux on physical CPU 0x0000000000 [0x411fd070]
[    0.000000] Linux version 6.1.18 (ryan@ryan-Precision-7510) (aarch64-linux-gnu-gcc (Ubuntu 13.2.0-23ubuntu4) 13.2.0, GNU ld (GNU Binutils for Ubuntu) 2.42) #2 SMP PREEMPT Thu Sep 26 23:39:05 EDT 2024
```

Again there is Kernel panic as no root-fs was given this time.
```
[    1.456485] Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
[    1.457831] CPU: 3 PID: 1 Comm: swapper/0 Not tainted 6.1.18 #2
[    1.458997] Hardware name: linux,dummy-virt (DT)
[    1.460332] Call trace:
[    1.460884]  dump_backtrace.part.0+0xdc/0xf0
[    1.461441]  show_stack+0x18/0x30
[    1.461823]  dump_stack_lvl+0x64/0x80
[    1.462228]  dump_stack+0x18/0x34
[    1.462583]  panic+0x188/0x344
[    1.462924]  mount_block_root+0x188/0x238
[    1.463399]  mount_root+0x214/0x254
[    1.463756]  prepare_namespace+0x12c/0x16c
[    1.464200]  kernel_init_freeable+0x258/0x284
[    1.464931]  kernel_init+0x24/0x12c
[    1.465316]  ret_from_fork+0x10/0x20
[    1.466221] SMP: stopping secondary CPUs
[    1.467493] Kernel Offset: disabled
[    1.467945] CPU features: 0x22000,2023c084,0000421b
[    1.468743] Memory Limit: none
[    1.469460] ---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0) ]---
```