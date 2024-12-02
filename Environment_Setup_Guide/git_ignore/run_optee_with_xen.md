
## 10/18
```
ryan@ryan-Precision-7510:~/Projects/xenonarm/optee2/build$ make XEN_BOOT=y QEMU_VIRTFS_ENABLE=y QEMU_USERNET_ENABLE=y run-only

cd /home/ryan/Projects/xenonarm/optee2/build/../out/bin && /home/ryan/Projects/xenonarm/optee2/build/../qemu/build/aarch64-softmmu/qemu-system-aarch64 \
	-nographic -smp 4 -cpu max,sme=off,pauth-impdef=on -d unimp -semihosting-config enable=on,target=native -m 3072 -bios bl1.bin -initrd rootfs.cpio.gz -kernel Image -append 'console=ttyAMA0,38400 keep_bootcon root=/dev/vda2 ' -drive if=none,file=/home/ryan/Projects/xenonarm/optee2/build/../out/bin/xen.ext4,format=raw,id=hd1 -device virtio-blk-device,drive=hd1 -object rng-random,filename=/dev/urandom,id=rng0 -device virtio-rng-pci,rng=rng0,max-bytes=1024,period=1000 -netdev user,id=vmnic -device virtio-net-device,netdev=vmnic -machine virt,acpi=off,secure=on,mte=off,gic-version=3,virtualization=true  -fsdev local,id=fsdev0,path=/home/ryan/Projects/xenonarm/optee2/build/..,security_model=none -device virtio-9p-device,fsdev=fsdev0,mount_tag=host -s -S -serial tcp:127.0.0.1:54320 -serial tcp:127.0.0.1:54321 

```

```
U-Boot 2023.07.02 (Oct 17 2024 - 21:26:26 -0400)

DRAM:  3 GiB
Core:  51 devices, 14 uclasses, devicetree: board
Flash: 32 MiB


Booting /xen.efi
Xen 4.18.0 (c/s Thu Nov 16 21:44:21 2023 +0000 git:d75f1e9) EFI loader
Using configuration file 'xen.cfg'
Image: 0x00000000faf68000-0x00000000fd732a00
rootfs.cpio.gz: 0x00000000f89c0000-0x00000000faf672e5
Using bootargs from Xen configuration file.
 Xen 4.18.0
(XEN) Xen version 4.18.0 (ryan@) (aarch64-linux-gnu-gcc (Arm GNU Toolchain 11.3.Rel1) 11.3.1 20220712) debug=n Thu Oct 17 23:44:38 EDT 2024
(XEN) Latest ChangeSet: Thu Nov 16 21:44:21 2023 +0000 git:d75f1e9
(XEN) build-id: b274cfec1e2d2a59f41c9fdcda7576bcbded782c
(XEN) Processor: 00000000000f0510: "Unknown", variant: 0x0, part 0x051,rev 0x0
(XEN) 64-bit Execution:
(XEN)   Processor Features: 1201001121112222 0000000000000021
(XEN)     Exception Levels: EL3:64+32 EL2:64+32 EL1:64+32 EL0:64+32
(XEN)     Extensions: FloatingPoint AdvancedSIMD GICv3-SysReg
(XEN)   Debug Features: 0000000010305609 0000000000000000
(XEN)   Auxiliary Features: 0000000000000000 0000000000000000
(XEN)   Memory Model Features: 0100032310201126 0000011010311122
(XEN)   ISA Features:  1221111110212120 0011111110211102
(XEN) 32-bit Execution:
(XEN)   Processor Features: 0000000011020131:0000000010011011
(XEN)     Instruction Sets: AArch32 A32 Thumb Thumb-2 Jazelle
(XEN)     Extensions: GenericTimer Security
(XEN)   Debug Features: 0000000006000099
(XEN)   Auxiliary Features: 0000000000000000
(XEN)   Memory Model Features: 0000000010101105 0000000040000000
(XEN)                          0000000001260000 0000000002122211
(XEN)   ISA Features: 0000000002101110 0000000013112111 0000000021232042
(XEN)                 0000000001112131 0000000000011142 0000000011011121
(XEN) Using SMC Calling Convention v1.4


(XEN) OP-TEE supports 2 simultaneous threads per guest.
(XEN) Using TEE mediator for OP-TEE


(XEN) *** LOADING DOMAIN 0 ***
(XEN) Loading d0 kernel from boot module @ 00000000faf68000
(XEN) Loading ramdisk from boot module @ 00000000f89c0000

(XEN) *** Serial input to DOM0 (type 'CTRL-a' three times to switch input)
(XEN) Freed 352kB init memory.
[    0.000000] Booting Linux on physical CPU 0x0000000000 [0x000f0510]
[    0.000000] Linux version 6.6.0-ga24278ea74b2 (ryan@ryan-Precision-7510) (aarch64-linux-gnu-gcc (Arm GNU Toolchain 11.3.Rel1) 11.3.1 20220712, GNU ld (Arm GNU Toolchain 11.3.Rel1) 2.38.20220708) #1 SMP PREEMPT Thu Oct 17 23:21:08 EDT 2024

[    0.249842] CPU: All CPU(s) started at EL1

[    5.204041] optee: probing for conduit method.
[    5.208268] optee: revision 4.3
[    5.219981] optee: dynamic shared memory is enabled
[    5.316835] optee: initialized driver
[    5.320544] Driver 'optee' was unable to register with bus_type 'arm_ffa' because the bus was not initialized.

[    5.204041] optee: probing for conduit method.
[    5.208268] optee: revision 4.3
[    5.219981] optee: dynamic shared memory is enabled
[    5.316835] optee: initialized driver
[    5.320544] Driver 'optee' was unable to register with bus_type 'arm_ffa' because the bus was not initialized.


Starting network: OK
Starting network (udhcpc): OK
Starting domain watchdog daemon: xenwatchdogd startup

Starting /usr/sbin/xenstored...
Setting domain 0 name, domid and JSON config...
Done setting up Dom0
Starting xenconsoled...
Starting QEMU as disk backend for dom0
[    8.735964] qemu-system-i38[182]: memfd_create() called without MFD_EXEC or MFD_NOEXEC_SEAL set
qemu-system-i386: -xen-domid 0: Option not supported for this target
  [done] 

Welcome to Buildroot, type root or test to login
buildroot login: root
# 
```


```
# uname -a   
Linux buildroot 6.6.0-ga24278ea74b2 #1 SMP PREEMPT Thu Oct 17 23:21:08 EDT 2024 aarch64 GNU/Linux



```

```
# xl vm-list 
UUID                                  ID    name
00000000-0000-0000-0000-000000000000  0    Domain-0 
```


```
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast qlen 1000
    link/ether 52:54:00:12:34:56 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global eth0
       valid_lft forever preferred_lft forever
```


```
# xl 
Usage xl [-vfNtT] <subcommand> [args]

xl full list of subcommands:

 create              Create a domain from config file <filename>
 config-update       Update a running domain's saved configuration, used when rebuilding the domain after reboot.
WARNING: xl now has better capability to manage domain configuration, avoid using this command when possible
 list                List information about all/some domains
 destroy             Terminate a domain immediately
 shutdown            Issue a shutdown signal to a domain
 reboot              Issue a reboot signal to a domain
 pci-attach          Insert a new pass-through pci device
 pci-detach          Remove a domain's pass-through pci device
 pci-list            List pass-through pci devices for a domain
 pci-assignable-add  Make a device assignable for pci-passthru
 pci-assignable-remove 
```


```
# mount 
rootfs on / type rootfs (rw)
devtmpfs on /dev type devtmpfs (rw,relatime,size=421756k,nr_inodes=105439,mode=755)
proc on /proc type proc (rw,relatime)
devpts on /dev/pts type devpts (rw,relatime,gid=5,mode=620,ptmxmode=666)
tmpfs on /dev/shm type tmpfs (rw,relatime)
tmpfs on /tmp type tmpfs (rw,relatime)
tmpfs on /run type tmpfs (rw,nosuid,nodev,relatime,mode=755)
sysfs on /sys type sysfs (rw,relatime)
host on /mnt/host type 9p (rw,relatime,access=client,msize=65536,trans=virtio)
xenfs on /proc/xen type xenfs (rw,relatime)


# ls /mnt/host/
SCP-firmware        mbedtls             out-br
build               optee_client        qemu
buildroot           optee_examples      toolchains
d.make.toolchains   optee_os            trusted-firmware-a
d.make_run          optee_rust          u-boot
hafnium             optee_test          xen
linux               out
#              


# ls /mnt/host/optee_examples/
.git/               LICENSE             hotp/
.github/            Makefile            plugins/
.gitignore          README.md           random/
Android.mk          acipher/            secure_storage/
CMakeLists.txt      aes/                
CMakeToolchain.txt  hello_world/  

# ls -lh /usr/bin/qemu-system-i386
-rwxr-xr-x    1 root     root       20.9M Oct 18 02:09 /usr/bin/qemu-system-i386

```
