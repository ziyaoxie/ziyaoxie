# Start The QEMU Image

## QEMU

```Shell
sudo apt-get install -y htop gcc g++ libglib2.0-dev libpixman-1-dev make libgtk-3-dev ninja-build
git clone https://github.com/qemu/qemu.git
cd qemu && mkdir build && cd build
../configure && make -j$(nproc)
make install
```

## Cross compiler

```Shell
sudo apt-get install gcc-arm-linux-gnueabi
```

## Compile the kernel image

```Shell
git clone https://github.com/torvalds/linux.git
cd linux && make CROSS_COMPILE=arm-linux-gnueabi- ARCH=arm vexpress_defconfig
# or make menuconfig if you need
make -j$(nproc) CROSS_COMPILE=arm-linux-gnueabi- ARCH=arm
# get image file at arch/arm/boot/zImage
```

## Busybox

```Shell
sudo apt-get install libncurses5-dev libncursesw5-dev
git clone https://git.busybox.net/busybox/
cd busybox && make menuconfig
# set "Build BusyBox as a static binary (no shared libs)" to y:
# Busybox Settings ---> Build Options ---> Build BusyBox as a static binary (no shared libs)
make defconfig && make -j$(nproc) CROSS_COMPILE=arm-linux-gnueabi- ARCH=arm
make install CROSS_COMPILE=arm-linux-gnueabi- ARCH=arm
```

## Rootfs

```Shell
mkdir -p rootfs/{dev,lib,mnt,proc,root,sys,tmp,var}
mkdir -p rootfs/etc/init.d
cp -rf busybox/_install/* rootfs/
sudo cp -P /usr/arm-linux-gnueabi/lib/* rootfs/lib/
```

- `dev`

```Shell
sudo mknod rootfs/dev/tty1 c 4 1
sudo mknod rootfs/dev/tty2 c 4 2
sudo mknod rootfs/dev/tty3 c 4 3
sudo mknod rootfs/dev/tty4 c 4 4
sudo mknod -m 666 rootfs/console c 5 1
```

- `etc`

```Shell
cat << EOF >rootfs/etc/inittab
::sysinit:/etc/init.d/rcS
#::respawn:-/bin/sh
#tty2::askfirst:-/bin/sh
#::ctrlaltdel:/bin/umount -a -r

console::askfirst:-/bin/sh
::ctrlaltdel:/sbin/reboot
::shutdown:/bin/umount -a -r
EOF
```

```Shell
cat << EOF >rootfs/etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda1 during installation
proc            /proc           proc    defaults                0       0
tmpfs           /tmp            tmpfs   defaults                0       0
sysfs           /sys            sysfs   defaults                0       0
tmpfs           /dev            tmpfs   defaults                0       0
var             /dev            tmpfs   defaults                0       0
ramfs           /dev            ramfs   defaults                0       0
EOF
```

```Shell
cat << EOF >rootfs/etc/init.d/rcS
#!/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
LD_LIBRARY_PATH=/lib
export PATH LD_LIBRARY_PATH

mount -a
mkdir -p /dev/pts
mount -t devpts devpts /dev/pts
mdev -s
mkdir -p /var/lock
echo "----------------------------------------"

echo " welcome debugging on A9 vexpress board"

echo "----------------------------------------"
EOF
```

```Shell
chmod 777 rootfs/etc/init.d/rcS
```

## Rootfs Image

```Shell
dd if=/dev/zero of=rootfs.ext4 bs=1M count=512
mkfs.ext4 rootfs.ext4
mkdir tmpfs
mount -t ext4 rootfs.ext4 tmpfs/ -o loop
cp -r rootfs/* tmpfs/
umount tmpfs
```

## Start the QEMU Image

```Shell
qemu-system-arm -M vexpress-a9 -m 512m -kernel ./linux/arch/arm/boot/zImage -dtb ./linux/arch/arm/boot/dts/vexpress-v2p-ca9.dtb -nographic -append "root=/dev/mmcblk0 console=ttyAMA0" -sd rootfs.ext4
```
