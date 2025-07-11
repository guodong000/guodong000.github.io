+++
date = '2022-05-02T12:58:37+08:00'
draft = false
title = '创建基于 BusyBox 的 initrd (Kernel 2.6)'
summary = '用于 qemu 启动 kernel 2.6.15'
+++

下载 busybox-1.18.5，由于内核版本较老，这里同样采用比较老的 busybox 版本。

进入 busybox 源码目录。

运行 `make defconfig`，才用默认配置。

运行 `make menuconfig` 进行配置

* `Busybox Settings` -> `Build Options` -> `[*] Build BusyBox as a static binary (no shared libs)`  
以静态方式编译 BusyBox
* `Miscellaneous Utilities` -> `[ ] ionice`  
去除 ionice

编译 `make && make install`

> 编译过程中可能会遇到报错，可选择更老版本的 busybox 或者针对性修改代码。

`cd _install` 进入 busybox 安装目录

```bash
# 创建系统运行时的必须目录
# /proc         用于挂载 proc 系统
# /sys          用于挂在 sys 系统
# /dev          用于 mdev 创建设备节点
# /etc/init.d   用于放置 busybox 启动脚本
mkdir -p proc sys dev etc/init.d

# busybox 启动脚本
cat <<EOF > etc/init.d/rcS
#!/bin/sh
mount -t proc none /proc    # 挂载 proc 文件系统
mount -t sys nonw /sys      # 挂载 sys 文件系统
# mdev 为 busybox 自带的 udev，用于系统启动和热插拔或动态加载驱动程序时，自动产生设备节点
# 如果不使用则需要手动 mknod 来挂载设备节点
/sbin/mdev -s
EOF

chmod +x etc/init.d/rcS

# 创建 initrd 镜像
find . | cpio -o --format=newc > ../initrd.img
# 创建压缩镜像
gzip -c ../initrd.img > initrd.img.gz
```

通过 qemu 运行内核：

```bash
qemu-system-x86_64 \
-kernel linux-2.6.15/arch/x86_64/boot/bzImage \
-initrd initrd.img \
-append "root=/dev/ram rdinit=/sbin/init console=ttyS0 noacpi" \
-nographic
# initrd.img 或者 initrd.img.gz 都可以。

# 内核启动参数 
# root=根文件系统所在设备 
# rdinit=内核加载完毕后用于创建首个进程的init程序
```
