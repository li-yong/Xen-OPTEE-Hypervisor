
Now verify the Qemu is able to run the Linux (Arm Arch) with the rootfs provided by busybox.

```
cd $BUILD_DIR/busybox_arm64
```
```
/home/xenonarm/build/busybox_arm64$ ls
Image.gz  rootfs.img.gz
```

## Run
```
qemu-system-aarch64 \
    -machine virt,gic_version=3 \
    -machine virtualization=true \
    -cpu cortex-a57 \
    -machine type=virt \
    -m 4096 \
    -smp 4 \
    -kernel Image.gz \
    -nographic \
    -no-reboot \
    -initrd rootfs.img.gz \
    -append "rw root=/dev/ram rdinit=/sbin/init earlyprintk=serial,ttyAMA0 console=ttyAMA0" 
```


## Output Review
```
[    0.000000] Booting Linux on physical CPU 0x0000000000 [0x411fd070]
[    0.000000] Linux version 6.1.18 (ryan@ryan-Precision-7510) (aarch64-linux-gnu-gcc (Ubuntu 13.2.0-23ubuntu4) 13.2.0, GNU ld (GNU Binutils for Ubuntu) 2.42) #2 SMP PREEMPT Thu Sep 26 23:39:05 EDT 2024
```

ARM CPU detected
```
[    0.000000] CPU features: detected: GIC system register CPU interface
[    0.000000] CPU features: detected: Spectre-v2
[    0.000000] CPU features: detected: Spectre-v3a
[    0.000000] CPU features: detected: Spectre-v4
[    0.000000] CPU features: detected: Spectre-BHB
[    0.000000] CPU features: kernel page table isolation forced ON by KASLR
[    0.000000] CPU features: detected: Kernel page table isolation (KPTI)
[    0.000000] CPU features: detected: ARM erratum 834220
[    0.000000] CPU features: detected: ARM erratum 1742098
[    0.000000] CPU features: detected: ARM erratum 832075
[    0.000000] CPU features: detected: ARM errata 1165522, 1319367, or 1530923
```

```
[    0.085211] smp: Brought up 1 node, 4 CPUs
[    0.085243] SMP: Total of 4 processors activated.
```

```
[    0.089721] CPU: All CPU(s) started at EL2

```

```
...
[    1.113126] uart-pl011 9000000.pl011: no DMA platform data
[    1.220320] Freeing unused kernel memory: 7552K
[    1.222026] Run /sbin/init as init process

Please press Enter to activate this console.
```

## Enter to active the linux console.
Now you have a run a ARM arch Linux on the Qemu emulation.
```
Please press Enter to activate this console. 
~ # uname -a
Linux (none) 6.1.18 #2 SMP PREEMPT Thu Sep 26 23:39:05 EDT 2024 aarch64 GNU/Linux
~ # 
```