
# Run with root-fs 
```
printf "0x%x\n" $(stat -c %s Image.gz)
0xbe1581

printf "0x%x\n" $(stat -c %s rootfs.img.gz)
0x122a54


compile xen
make dist-xen XEN_TARGET_ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j8

make dist-xen XEN_TARGET_ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-


./test/livepatch/Makefile:ifeq ($(XEN_TARGET_ARCH),arm64)






cd /home/ryan/Projects/xenonarm/optee/build/../out/bin && /home/ryan/Projects/xenonarm/optee/build/../qemu/build/aarch64-softmmu/qemu-system-aarch64         -nographic -smp 2 -cpu max,sme=on,pauth-impdef=on -d unimp -semihosting-config enable=on,target=native -m 1057 -bios bl1.bin -initrd rootfs.cpio.gz -kernel Image -append 'console=ttyAMA0,38400 keep_bootcon root=/dev/vda2 '  -object rng-random,filename=/dev/urandom,id=rng0 -device virtio-rng-pci,rng=rng0,max-bytes=1024,period=1000 -netdev user,id=vmnic -device virtio-net-device,netdev=vmnic -machine virt,acpi=off,secure=on,mte=off,gic-version=3,virtualization=false   -s -S -serial tcp:127.0.0.1:54320 -serial tcp:127.0.0.1:54321




qemu-system-aarch64 \
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
    -chardev socket,id=qemu-monitor,host=localhost,port=7777,server=on,wait=off,telnet=on \
    -device e1000,netdev=network0 -netdev tap,id=network0,ifname=tap299,script=no,downscript=no \
    -s -S -serial tcp:127.0.0.1:54320 -serial tcp:127.0.0.1:54321 \
    -mon qemu-monitor,mode=readline

 


fdt addr 0x44000000
fdt resize
fdt set /chosen \#address-cells <1>
fdt set /chosen \#size-cells <1>
fdt mknod /chosen module@0
fdt set /chosen/module@0 compatible "xen,linux-zimage" "xen,multiboot-module"
fdt set /chosen/module@0 reg <0x47000000 0xbe1581>
fdt set /chosen/module@0 bootargs "rw root=/dev/ram rdinit=/sbin/init   earlyprintk=serial,ttyAMA0 console=hvc0 earlycon=xenboot"
fdt mknod /chosen module@1
fdt set /chosen/module@1 compatible "xen,linux-initrd" "xen,multiboot-module"
fdt set /chosen/module@1 reg <0x42000000 0x122a54>
booti 0x49000000 - 0x44000000


```


This time, Linux no longer panic, Xen give you option to activate the Linux console.
```
Please press Enter to activate this console.
```

## Output Review

U-Boot boot the xen
```
Xen 4.18.3
(XEN) Xen version 4.18.3 (ryan@) (aarch64-linux-gnu-gcc (Ubuntu 13.2.0-23ubuntu4) 13.2.0) debug=n Fri Sep 27 22:52:12 EDT 2024
```

XEN start DOM0
```
(XEN) ***************************************************
(XEN) PLEASE SPECIFY dom0_mem PARAMETER - USING 512M FOR NOW
(XEN) ***************************************************
(XEN) 3... 2... 1... 
(XEN) *** Serial input to DOM0 (type 'CTRL-a' three times to switch input)
(XEN) Freed 364kB init memory.
[    0.000000] Booting Linux on physical CPU 0x0000000000 [0x411fd070]
[    0.000000] Linux version 6.1.18 (ryan@ryan-Precision-7510) (aarch64-linux-gnu-gcc (Ubuntu 13.2.0-23ubuntu4) 13.2.0, GNU ld (GNU Binutils for Ubuntu) 2.42) #2 SMP PREEMPT Thu Sep 26 23:39:05 EDT 2024
```

The linux booted with the root-fs previously built in Busybox
```
Please press Enter to activate this console. [   11.544030] platform gpio-keys: deferred probe pending
  
~ # uname -a
Linux (none) 6.1.18 #2 SMP PREEMPT Thu Sep 26 23:39:05 EDT 2024 aarch64 GNU/Linux
~ # 
~ # ls /
bin      etc      proc     sbin     usr
dev      linuxrc  root     sys

~ # which grep
/bin/grep

~ # ls -l /bin/grep
lrwxrwxrwx    1 1000     1000             7 Sep 27 01:29 /bin/grep -> busybox
```


