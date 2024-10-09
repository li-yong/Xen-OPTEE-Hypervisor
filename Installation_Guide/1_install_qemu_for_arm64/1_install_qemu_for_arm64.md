


# Ubuntu Base OS
```
$ cat /etc/lsb-release 
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=24.04
DISTRIB_CODENAME=noble
DISTRIB_DESCRIPTION="Ubuntu 24.04.1 LTS"
```

# Install QEMU
```
$ sudo apt install qemu-system-arm
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
qemu-system-arm is already the newest version (1:8.2.2+ds-0ubuntu1.2).
0 upgraded, 0 newly installed, 0 to remove and 24 not upgraded.
```

# Show QEMU version
```
$ qemu-system-aarch64 -version
QEMU emulator version 8.2.2 (Debian 1:8.2.2+ds-0ubuntu1.2)
Copyright (c) 2003-2023 Fabrice Bellard and the QEMU Project developers
```
