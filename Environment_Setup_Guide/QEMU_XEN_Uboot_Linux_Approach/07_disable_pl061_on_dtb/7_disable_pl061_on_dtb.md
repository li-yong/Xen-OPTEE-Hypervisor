

# Disable ARM PL061
```
dtc -I dtb -O dts virt-gicv3.dtb > virt-gicv3.dts
sed 's/compatible = "arm,pl061.*/status = "disabled";/g' virt-gicv3.dts > virt-gicv3-edited.dts
dtc -I dts -O dtb virt-gicv3-edited.dts > virt-gicv3.dtb
```

https://stackoverflow.com/questions/68372448/how-do-i-run-xen-on-arm64-and-qemu-6-0-0-with-linux-4-20-11-as-dom0