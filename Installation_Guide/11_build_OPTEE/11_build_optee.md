
In this section, we build OP-TEE for QEMU v8 (Armv8-A). OP-TEE repository and make file have build 

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

## Install Git Repo
The command installs the `repo` tool by downloading it and making it executable.
curl https://storage.googleapis.com/git-repo-downloads/repo > /bin/repo && chmod a+x /bin/repo

## Fetch qemu_v8 manifest
Initializes a repo environment by setting up the OP-TEE repository's configuration.After running this command, the environment is prepared for syncing all necessary OP-TEE repositories using `repo sync`.

```
mkdir /optee
cd /optee
repo init -u https://github.com/OP-TEE/manifest.git -m qemu_v8.xml && repo sync -j10
```

## Build toolchains
In this setup, running `make toolchains` is essential because the development machine (x86 architecture) cannot directly compile code for the target ARM architecture without using specific cross-compilation toolchains. This command will download and build the `aarch32`,`aarch64` and `rust` toolchains that enable the x86 host system to produce ARM-compatible binaries, which are required for OP-TEE. Even though QEMU will emulate the ARM CPU, the binaries themselves must still be ARM-compatible to function correctly within that emulated environment. By running `make toolchains`, we ensure compatibility with the ARM architecture and prepare the environment for a seamless emulation experience.

```
cd /optee/build
make -j `nproc` toolchains
```


Runs the `make` command to check the build environment, utilizing all available CPU cores. The `$(nproc)` command dynamically calculates the number of available cores, optimizing the check process for performance.

Together, these commands set up and verify the OP-TEE build environment.
```
make -j `nproc` check
```

## Build Everything
This will spend significant time comparing to the previous steps, as it is actually build and install the `all` target. QEMU, ARM-TF (ARM TrustFirmware), Linux, OPTEE-OS, U-Boot will be compiled and installed in this step.
```
make -j `nproc`
```

### Performance Benchmark consideration
OP-TEE is being a solution spanning over several architectural layers. OP-TEE [Benchmark-framework](https://optee.readthedocs.io/en/3.21.0/debug/benchmark.html#benchmark-framework) enables performance latency observation over difference layers, it provides detailed and precise profiling information for each layer. 

If you intend to use Benchmark framework, OP-TEE should be rebuilt with the CFG_TEE_BENCHMARK flag enabled so that the benchmark framework will be enabled in all architectural layers. 
```
make CFG_TEE_BENCHMARK=y -j `nproc`
```

 
## Run the Application 
By this step, all the binaries have been built. Next we are going to invoke QEMU to boot XEN, Linux along with OP-TEE.  We also need to transfer files from the guest VM to the host, hence `QEMU_VIRTFS_ENABLE` and `QEMU_USERNET_ENABLE` flags are specified. 
```
make XEN_BOOT=y QEMU_VIRTFS_ENABLE=y QEMU_USERNET_ENABLE=y run-only
```

In the above `make`, the core action is to launch the QEMU. 
```
nc -z  127.0.0.1 54320 || /usr/bin/gnome-terminal -t "Normal World" -x /home/xenonarm/optee/build/../build/soc_term.py 54320 &
nc -z  127.0.0.1 54321 || /usr/bin/gnome-terminal -t "Secure World" -x /home/xenonarm/optee/build/../build/soc_term.py 54321 &


cd /home/xenonarm/optee/build/../out/bin && \
/home/xenonarm/optee/build/../qemu/build/aarch64-softmmu/qemu-system-aarch64 \
    -nographic \
    -smp 4 \
    -cpu cortex-a57 \
    -d unimp \
    -semihosting-config enable=on,target=native \
    -m 3072 \
    -bios bl1.bin \
    -initrd rootfs.cpio.gz \
    -kernel Image \
    -append 'console=ttyAMA0,38400 keep_bootcon root=/dev/vda2 ' \
    -drive if=none,file=/home/xenonarm/optee/build/../out/bin/xen.ext4,format=raw,id=hd1 \
    -device virtio-blk-device,drive=hd1 \
    -object rng-random,filename=/dev/urandom,id=rng0 \
    -device virtio-rng-pci,rng=rng0,max-bytes=1024,period=1000 \
    -netdev user,id=vmnic \
    -device virtio-net-device,netdev=vmnic \
    -machine virt,acpi=off,secure=on,mte=off,gic-version=3,virtualization=true \
    -fsdev local,id=fsdev0,path=/home/xenonarm/optee/build/..,security_model=none \
    -device virtio-9p-device,fsdev=fsdev0,mount_tag=host \
    -s -S \
    -serial tcp:127.0.0.1:54320 \
    -serial tcp:127.0.0.1:54321
```
A QEMU console and two UART consoles point to secure world and normal world will be open.

Type `c` in qemu prompt to continue.
```
QEMU 8.1.2 monitor - type 'help' for more information
(qemu) c
```


<!-- ### Normal World Boot process.-->
### QEMU Load the Trusted Firmware and U-Boot.
```
listening on port 54320
soc_term: accepted fd 4
NOTICE:  Booting Trusted Firmware
NOTICE:  BL1: v2.10.0	(release):v2.10
NOTICE:  BL1: Built : 21:28:00, Oct 17 2024
WARNING: Firmware Image Package header check failed.
NOTICE:  BL1: Booting BL2
NOTICE:  BL2: v2.10.0	(release):v2.10
NOTICE:  BL2: Built : 21:28:02, Oct 17 2024
WARNING: Firmware Image Package header check failed.
WARNING: Firmware Image Package header check failed.
WARNING: Firmware Image Package header check failed.
WARNING: Firmware Image Package header check failed.
NOTICE:  BL1: Booting BL31
NOTICE:  BL31: v2.10.0	(release):v2.10
NOTICE:  BL31: Built : 21:28:06, Oct 17 2024
```

### U-Boot start.
```
U-Boot 2023.07.02 (Oct 17 2024 - 21:26:26 -0400)

DRAM:  3 GiB
Core:  51 devices, 14 uclasses, devicetree: board
Flash: 32 MiB
```

### Secure World start
```
listening on port 54321
soc_term: accepted fd 4


I/TC: OP-TEE version: 4.4.0-rc1-2-g1868eb206 (gcc version 11.3.1 20220712 (Arm GNU Toolchain 11.3.Rel1)) #1 Fri Oct 18 01:26:07 UTC 2024 aarch64
I/TC: WARNING: This OP-TEE configuration might be insecure!
I/TC: WARNING: Please check https://optee.readthedocs.io/en/latest/architecture/porting_guidelines.html

I/TC: Primary CPU initializing
D/TC:0 0   boot_init_primary_late:1011 Executing at offset 0xa7000000 with virtual load address 0xb5100000

I/TC: Initializing virtualization support
I/TC: Primary CPU switching to normal world boot
```



### XEN start
```
Booting /xen.efi
Xen 4.18.0 (c/s Thu Nov 16 21:44:21 2023 +0000 git:d75f1e9) EFI loader
Using configuration file 'xen.cfg'
Image: 0x00000000fb568000-0x00000000fdd32a00
rootfs.cpio.gz: 0x00000000f8fc0000-0x00000000fb5672e5
Using bootargs from Xen configuration file.
 Xen 4.18.0
(XEN) Xen version 4.18.0 (ryan@) (aarch64-linux-gnu-gcc (Arm GNU Toolchain 11.3.Rel1) 11.3.1 20220712) debug=n Thu Oct 17 23:44:38 EDT 2024
(XEN) Latest ChangeSet: Thu Nov 16 21:44:21 2023 +0000 git:d75f1e9
(XEN) build-id: b274cfec1e2d2a59f41c9fdcda7576bcbded782c
(XEN) Processor: 00000000411fd070: "ARM Limited", variant: 0x1, part 0xd07,rev 0x0
(XEN) 64-bit Execution:
(XEN)   Processor Features: 0000000001002222 0000000000000000
(XEN)     Exception Levels: EL3:64+32 EL2:64+32 EL1:64+32 EL0:64+32
(XEN)     Extensions: FloatingPoint AdvancedSIMD GICv3-SysReg
(XEN)   Debug Features: 0000000010305106 0000000000000000
(XEN)   Auxiliary Features: 0000000000000000 0000000000000000
(XEN)   Memory Model Features: 0000000000001124 0000000000000000
(XEN)   ISA Features:  0000000000011120 0000000000000000
(XEN) 32-bit Execution:
(XEN)   Processor Features: 0000000000000131:0000000010011011
(XEN)     Instruction Sets: AArch32 A32 Thumb Thumb-2 Jazelle
(XEN)     Extensions: GenericTimer Security
```

#### Normal World Guest VM start
```
(XEN) *** Serial input to DOM0 (type 'CTRL-a' three times to switch input)
(XEN) Freed 352kB init memory.
[    0.000000] Booting Linux on physical CPU 0x0000000000 [0x411fd070]
[    0.000000] Linux version 6.6.0-ga24278ea74b2 (ryan@ryan-Precision-7510) (aarch64-linux-gnu-gcc (Arm GNU Toolchain 11.3.Rel1) 11.3.1 20220712, GNU ld (Arm GNU Toolchain 11.3.Rel1) 2.38.20220708) #1 SMP PREEMPT Thu Oct 17 23:21:08 EDT 2024
```

#### Login the guest and run xtest `bench-mark` test suite.
```
Welcome to Buildroot, type root or test to login
buildroot login: root
# xtest -h
# xtest -t bench-mark

TEE test application started over default TEE instance
######################################################
#
# benchmark
#
######################################################
 
* benchmark_1001 TEE Trusted Storage Performance Test (WRITE)
-----------------+---------------+----------------
 Data Size (B) 	 | Time (s)	 | Speed (kB/s)	 
-----------------+---------------+----------------
      256 	 |    0.034 	 |    7.353
      512 	 |    0.039 	 |   12.821
     1024 	 |    0.029 	 |   34.483
     2048 	 |    0.069 	 |   28.986
     4096 	 |    0.135 	 |   29.630
    16384 	 |    0.656 	 |   24.390
   524288 	 |   31.892 	 |   16.054
  1048576 	 |   58.995 	 |   17.357
-----------------+---------------+----------------
  benchmark_1001 OK
```
