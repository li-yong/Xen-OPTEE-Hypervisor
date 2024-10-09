# Running Dom0 and DomU simultaneously on Qemu-arm64


## Get the size in hex of Image and rootfs.
```
printf "0x%x\n" $(stat -c %s Image)
0x237ea00

printf "0x%x\n" $(stat -c %s rootfs.img.gz)
0x122a54
```

## Lunch qeum with xen and two Linux
```
qemu-system-aarch64 \
    -machine virt,gic_version=3 \
    -machine virtualization=true \
    -cpu cortex-a57 \
    -machine type=virt \
    -m 4096 \
    -smp 4 \
    -bios u-boot.bin \
    -device loader,file=xen,force-raw=on,addr=0x50000000 \
    -device loader,file=Image,addr=0x47000000 \
    -device loader,file=Image,addr=0x53000000 \
    -device loader,file=virt-gicv3.dtb,addr=0x44000000 \
    -device loader,file=rootfs.img.gz,addr=0x42000000 \
    -device loader,file=rootfs.img.gz,addr=0x58000000 \
    -nographic \
    -no-reboot \
    -chardev socket,id=qemu-monitor,host=localhost,port=7777,server,nowait,telnet \
    -mon qemu-monitor,mode=readline
```

## Call U-Boot
```
setenv xen_bootargs 'dom0_mem=512M'
fdt addr 0x44000000
fdt resize
fdt set /chosen \#address-cells <1>
fdt set /chosen \#size-cells <1>
fdt set /chosen xen,xen-bootargs \"$xen_bootargs\"
fdt mknod /chosen module@0
fdt set /chosen/module@0 compatible "xen,linux-zimage" "xen,multiboot-module"
fdt set /chosen/module@0 reg <0x47000000 0x237ea00>
fdt set /chosen/module@0 bootargs "rw root=/dev/ram rdinit=/sbin/init   earlyprintk=serial,ttyAMA0 console=hvc0 earlycon=xenboot"
fdt mknod /chosen module@1
fdt set /chosen/module@1 compatible "xen,linux-initrd" "xen,multiboot-module"
fdt set /chosen/module@1 reg <0x42000000 0x122a54>
fdt mknod /chosen domU1
fdt set /chosen/domU1 compatible "xen,domain"
fdt set /chosen/domU1 \#address-cells <1>
fdt set /chosen/domU1 \#size-cells <1>
fdt set /chosen/domU1 \cpus <1>
fdt set /chosen/domU1 \memory <0 548576>
fdt set /chosen/domU1 vpl011
fdt mknod /chosen/domU1 module@0
fdt set /chosen/domU1/module@0 compatible "multiboot,kernel" "multiboot,module"
fdt set /chosen/domU1/module@0 reg <0x53000000 0x237ea00>
fdt set /chosen/domU1/module@0 bootargs "rw root=/dev/ram rdinit=/sbin/init console=ttyAMA0"
fdt mknod /chosen/domU1 module@1
fdt set /chosen/domU1/module@1 compatible "multiboot,ramdisk" "multiboot,module"
fdt set /chosen/domU1/module@1 reg <0x58000000 0x122a54>
booti 0x50000000 - 0x44000000
```

## Switch Console between XEN/DOM0/DOM1
```
# (XEN) *** Serial input to DOM1 (type 'CTRL-a' three times to switch input)
(XEN) *** Serial input to Xen (type 'CTRL-a' three times to switch input)
(XEN) *** Serial input to DOM0 (type 'CTRL-a' three times to switch input)
```

## Output Review
XEN start
```
Starting kernel ...

 Xen 4.18.3
(XEN) Xen version 4.18.3 (ryan@) (aarch64-linux-gnu-gcc (Ubuntu 13.2.0-23ubuntu4) 13.2.0) debug=n Fri Sep 27 22:52:12 EDT 2024
```

Load Domain 0 (DOM0) and Domain 1(DOM1)
```
(XEN) *** LOADING DOMAIN 0 ***
(XEN) Loading d0 kernel from boot module @ 0000000047000000
(XEN) Loading ramdisk from boot module @ 0000000042000000
...
(XEN) *** LOADING DOMU cpus=1 memory=0x85ee0KB ***
(XEN) Loading d1 kernel from boot module @ 0000000053000000
(XEN) Allocating mappings totalling 535MB for d1:
(XEN) d1 BANK[0] 0x00000040000000-0x000000617b8000 (535MB)
(XEN) Loading zImage from 0000000053000000 to 0000000040000000-000000004237ea00
(XEN) Loading d1 DTB to 0x0000000048000000-0x0000000048000479
```



DOM0 Start
```
(XEN) *** Serial input to DOM0 (type 'CTRL-a' three times to switch input)

[    0.000000] Booting Linux on physical CPU 0x0000000000 [0x411fd070]
[    0.000000] Linux version 6.1.18 (ryan@ryan-Precision-7510) (aarch64-linux-gnu-gcc (Ubuntu 13.2.0-23ubuntu4) 13.2.0, GNU ld (GNU Binutils for Ubuntu) 2.42) #2 SMP PREEMPT Thu Sep 26 23:39:05 EDT 2024

```

DOM1 start
```
(XEN) DOM1: [    0.000000] Booting Linux on physical CPU 0x0000000000 [0x411fd070]
(XEN) DOM1: [    0.000000] Linux version 6.1.18 (ryan@ryan-Precision-7510) (aarch64-linux-gnu-gcc (Ubuntu 13.2.0-23ubuntu4) 13.2.0, GNU ld
(XEN) DOM1:  (GNU Binutils for Ubuntu) 2.42) #2 SMP PREEMPT Thu Sep 26 23:39:05 EDT 2024
```