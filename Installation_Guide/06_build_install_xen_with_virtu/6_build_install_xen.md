
# Get the source

```
cd $WORK_DIR
wget -c https://downloads.xenproject.org/release/xen/4.18.3/xen-4.18.3.tar.gz
tar xf xen-4.18.3.tar.gz
```

# Build XEN on ARM ARCH CPU
```
cd $WORK_DIR/xen-4.18.3
make dist-xen XEN_TARGET_ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
```

# Copy to the build dir.
```
cp xen/xen $BUILD_DIR/busybox_arm64/
```

# generate a device tree blob
```
cd $BUILD_DIR/busybox_arm64

qemu-system-aarch64 \
    -machine virt,gic_version=3 \
    -machine virtualization=true \
    -cpu cortex-a57 \
    -machine type=virt \
    -m 4096 \
    -smp 4 \
    -display none \
    -machine dumpdtb=virt-gicv3.dtb
```

# Get the files size in Hex 
```
printf "0x%x\n" $(stat -c %s Image.gz)
0xbe1581

printf "0x%x\n" $(stat -c %s rootfs.img.gz)
0x122a54
```

# Next step
Now if run the Qeme with Xen, and start the Linux as dom0, XEN will stuck on loading Linux. caused by the Advanced Microcontroller Bus Architecture (AMBA) compliant controller.  We will handle that in the next sector.


Stuck output
```
$ qemu-system-aarch64  -machine virt,gic_version=3 -machine virtualization=true -cpu cortex-a57 -machine type=virt -m 4096 -smp 4 -bios u-boot.bin -device loader,file=xen,force-raw=on,addr=0x49000000 -device loader,file=Image.gz,addr=0x47000000 -device loader,file=virt-gicv3.dtb,addr=0x44000000 -device loader,file=rootfs.img.gz,addr=0x42000000 -nographic -no-reboot -chardev socket,id=qemu-monitor,host=localhost,port=7777,server,nowait,telnet -mon qemu-monitor,mode=readline


U-Boot 2024.07 (Sep 26 2024 - 21:54:01 -0400)

DRAM:  4 GiB
Core:  51 devices, 14 uclasses, devicetree: board
Flash: 64 MiB
Loading Environment from Flash... *** Warning - bad CRC, using default environment
...
...
...
=> fdt addr 0x44000000
Working FDT set to 44000000
=> fdt resize
=> fdt set /chosen \#address-cells <1>
=> fdt set /chosen \#size-cells <1>
=> fdt mknod /chosen module@0
=> fdt set /chosen/module@0 compatible "xen,linux-zimage" "xen,multiboot-module"
=> fdt set /chosen/module@0 reg <0x47000000 0xbe1581>
=> fdt set /chosen/module@0 bootargs "earlyprintk=serial,ttyAMA0
> console=ttyAMA0,115200n8 earlycon=xenboot"
libfdt fdt_setprop(): FDT_ERR_NOSPACE
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
(XEN) Processor: 00000000411fd070: "ARM Limited", variant: 0x1, part 0xd07,rev 0x0
(XEN) 64-bit Execution:
(XEN)   Processor Features: 0000000001000222 0000000000000000
(XEN)     Exception Levels: EL3:No EL2:64+32 EL1:64+32 EL0:64+32
(XEN)     Extensions: FloatingPoint AdvancedSIMD GICv3-SysReg
...
...
...
(XEN) ***************************************************
(XEN) PLEASE SPECIFY dom0_mem PARAMETER - USING 512M FOR NOW
(XEN) ***************************************************
(XEN) 3... 2... 1... 
(XEN) *** Serial input to DOM0 (type 'CTRL-a' three times to switch input)
(XEN) Freed 364kB init memory.
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER4
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER8
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER12
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER16
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER20
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER24
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER28
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER32
(XEN) d0v0: vGICR: SGI: unhandled word write 0x000000ffffffff to ICACTIVER0
(XEN) d0v1: vGICR: SGI: unhandled word write 0x000000ffffffff to ICACTIVER0  <<< stuck here. Have to abort by `pkill -f qemu-system-aarch64`
```
