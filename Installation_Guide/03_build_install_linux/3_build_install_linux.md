
## Get the Kernel Source
```
cd $WORK_DIR
wget -c https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.1.18.tar.xz
tar xf linux-6.1.18.tar.xz
```
## make config
```
cd $WORK_DIR/linux-6.1.18/
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
```

## Check .config 
Ensure CONFIG_XEN included in the .config. 
```
grep CONFIG_XEN .config
```
Output should have lines. Append lines if not. 
```
CONFIG_XEN_DOM0=y
CONFIG_XEN=y
```

## Build kernel
```
make -j4 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
```

## Copy the kernel image to distinct folder.
```
cp ./arch/arm64/boot/Image.gz $BUILD_DIR/busybox_arm64/
```