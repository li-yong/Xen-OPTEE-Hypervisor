

## Install pre-request
```
DEBIAN_FRONTEND=noninteractive
apt update && apt upgrade -y
apt install -y \
    adb \
    acpica-tools \
    autoconf \
    automake \
    bc \
    bison \
    build-essential \
    ccache \
    cpio \
    cscope \
    curl \
    device-tree-compiler \
    e2tools \
    expect \
    fastboot \
    flex \
    ftp-upload \
    gdisk \
    git \
    libattr1-dev \
    libcap-ng-dev \
    libfdt-dev \
    libftdi-dev \
    libglib2.0-dev \
    libgmp3-dev \
    libhidapi-dev \
    libmpc-dev \
    libncurses5-dev \
    libpixman-1-dev \
    libslirp-dev \
    libssl-dev \
    libtool \
    libusb-1.0-0-dev \
    make \
    mtools \
    netcat \
    ninja-build \
    python3-cryptography \
    python3-pip \
    python3-pyelftools \
    python3-serial \
    python-is-python3 \
    rsync \
    swig \
    unzip \
    uuid-dev \
    wget \
    xdg-utils \
    xterm \
    xz-utils \
    zlib1g-dev
```

## Get 
curl https://storage.googleapis.com/git-repo-downloads/repo > /bin/repo && chmod a+x /bin/repo

## Fetch qemu_v8 manifest
```
mkdir /optee
cd /optee
repo init -u https://github.com/OP-TEE/manifest.git -m qemu_v8.xml && repo sync -j10
```

## Build toolchains.  
Notice besides Qemu, u-boot ... will be fetched by make jobs too.
```
cd /optee/build
make -j2 toolchains
make -j `nproc` check
```

## Build install
```
make -j `nproc`
```

## Run
```
make run
```

Then you will get a QEMU console with two UART consoles point to secure world and normal world.




## Output

```
mkdir ~/Projects/xenonarm/optee

curl https://storage.googleapis.com/git-repo-downloads/repo > repo


repo init -u https://github.com/OP-TEE/manifest.git -m qemu_v8.xml && repo sync -j1

cd /optee/build

make -j2 toolchains


source /home/Projects/xenonarm/optee/build/../toolchains/rust/.cargo/env

Verify carga is in the path
cargo -h

make -j4 check

    LD      vmlinux
    NM      System.map
    SORTTAB vmlinux
    OBJCOPY arch/arm64/boot/Image
    make[1]: Leaving directory '~/Projects/xenonarm/optee/linux'

make -j4
    nonarm/optee/build/../out/bin/rootfs.cpio.uboot
    Image Name:   Root file system
    Created:      Thu Oct 10 21:56:25 2024
    Image Type:   AArch64 Linux RAMDisk Image (gzip compressed)
    Data Size:    14744587 Bytes = 14399.01 KiB = 14.06 MiB
    Load Address: 45000000
    Entry Point:  45000000
    ~/Projects/xenonarm/optee

make run
    cd /home/ryan/Projects/xenonarm/optee/build/../out/bin && /home/ryan/Projects/xenonarm/optee/build/../qemu/build/aarch64-softmmu/qemu-system-aarch64 \
	-nographic -smp 2 -cpu max,sme=on,pauth-impdef=on -d unimp -semihosting-config enable=on,target=native -m 1057 -bios bl1.bin -initrd rootfs.cpio.gz -kernel Image -append 'console=ttyAMA0,38400 keep_bootcon root=/dev/vda2 '  -object rng-random,filename=/dev/urandom,id=rng0 -device virtio-rng-pci,rng=rng0,max-bytes=1024,period=1000 -netdev user,id=vmnic -device virtio-net-device,netdev=vmnic -machine virt,acpi=off,secure=on,mte=off,gic-version=3,virtualization=false   -s -S -serial tcp:127.0.0.1:54320 -serial tcp:127.0.0.1:54321 
QEMU 8.1.2 monitor - type 'help' for more information


```
cd /home/ryan/Projects/xenonarm/optee/build/../out/bin && /home/ryan/Projects/xenonarm/optee/build/../qemu/build/aarch64-softmmu/qemu-system-aarch64 \
	-nographic -smp 2 -cpu max,sme=on,pauth-impdef=on -d unimp -semihosting-config enable=on,target=native -m 1057 \
    -bios bl1.bin \
    -initrd rootfs.cpio.gz -kernel Image \
    -append 'console=ttyAMA0,38400 keep_bootcon root=/dev/vda2 '  -object rng-random,filename=/dev/urandom,id=rng0 -device virtio-rng-pci,rng=rng0,max-bytes=1024,period=1000 -netdev user,id=vmnic -device virtio-net-device,netdev=vmnic -machine virt,acpi=off,secure=on,mte=off,gic-version=3,virtualization=false   -s -S -serial tcp:127.0.0.1:54320 -serial tcp:127.0.0.1:54321 



qemu-system-aarch64 \
    -machine virt,gic_version=3 \
    -machine virtualization=true \
    -cpu cortex-a57 \
    -machine type=virt \
    -m 4096 \
    -smp 4 \
    -bios u-boot.bin \
    -device loader,file=xen_optee,force-raw=on,addr=0x50000000 \
    -device loader,file=Linux_optee_Image,addr=0x47000000 \
    -device loader,file=Linux_optee_Image,addr=0x53000000 \
    -device loader,file=virt-gicv3.dtb,addr=0x44000000 \
    -device loader,file=rootfs_cpio.optee.gz,addr=0x42000000 \
    -device loader,file=rootfs_cpio.optee.gz,addr=0x58000000 \
    -nographic \
    -no-reboot \
    -chardev socket,id=qemu-monitor,host=localhost,port=7777,server,nowait,telnet \
    -mon qemu-monitor,mode=readline






ryan@ryan-Precision-7510:~/Projects/xenonarm/build/busybox_arm64$ printf "0x%x\n" $(stat -c %s Linux_optee_Image)
0x27caa00
ryan@ryan-Precision-7510:~/Projects/xenonarm/build/busybox_arm64$ printf "0x%x\n" $(stat -c %s rootfs_cpio.optee.gz)
0xe0fccd



setenv xen_bootargs 'dom0_mem=512M'
fdt addr 0x44000000
fdt resize
fdt set /chosen \#address-cells <1>
fdt set /chosen \#size-cells <1>
fdt set /chosen xen,xen-bootargs \"$xen_bootargs\"
fdt mknod /chosen module@0
fdt set /chosen/module@0 compatible "xen,linux-zimage" "xen,multiboot-module"
fdt set /chosen/module@0 reg <0x47000000 0x27caa00>
fdt set /chosen/module@0 bootargs "rw root=/dev/ram rdinit=/sbin/init   earlyprintk=serial, console=hvc0 earlycon=xenboot"
fdt mknod /chosen module@1
fdt set /chosen/module@1 compatible "xen,linux-initrd" "xen,multiboot-module"
fdt set /chosen/module@1 reg <0x42000000 0xe0fccd>
fdt mknod /chosen domU1
fdt set /chosen/domU1 compatible "xen,domain"
fdt set /chosen/domU1 \#address-cells <1>
fdt set /chosen/domU1 \#size-cells <1>
fdt set /chosen/domU1 \cpus <1>
fdt set /chosen/domU1 \memory <0 548576>
fdt set /chosen/domU1 vpl011
fdt mknod /chosen/domU1 module@0
fdt set /chosen/domU1/module@0 compatible "multiboot,kernel" "multiboot,module"
fdt set /chosen/domU1/module@0 reg <0x53000000 0x27caa00>
fdt set /chosen/domU1/module@0 bootargs "rw root=/dev/ram rdinit=/sbin/init console=ttyAMA0,38400 keep_bootcon root=/dev/vda2 "

fdt resize
fdt mknod /chosen/domU1 module@1
fdt set /chosen/domU1/module@1 compatible "multiboot,ramdisk" "multiboot,module"
fdt set /chosen/domU1/module@1 reg <0x58000000 0xe0fccd>
booti 0x50000000 - 0x44000000




OK
(XEN) DOM1: Starting klogd: OK
Set permissions on /dev/tee*: chown: /dev/teepriv0: No such file or directory
FAIL
(XEN) DOM1: Running sysctl: /etc/init.d/S02sysctl: line 37: can't create /dev/null: Is a directory
Starting network: (XEN) DOM1: OK
(XEN) DOM1: Set permissions on /dev/tee*: chown: /dev/teepriv0: No such file or directory
(XEN) DOM1: FAIL
