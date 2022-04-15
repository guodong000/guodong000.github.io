---
title: macOS 下 VirtualBox 虚拟机网络设置
---

本文解决了在 macOS 下如何在保证 VirtualBox 虚拟机联网的同时支持 Host 主机连接虚拟机。比如：主机需要通过 SSH 连接虚拟机，同时虚拟机需要联网。

## 解决方法 1

首先，**最简单的方式**便是将虚拟机网卡连接方式设置为 `桥接网卡` 并为其指定一个主机上的网卡，这样主机和虚拟机相当于连接到同一个路由下，达到上述目标自然不成问题。

如果不想采用桥接方式，则需要下面稍为复杂一些的设置。

## 解决方法 2

当虚拟机网卡连接方式为 `网络地址转换(NAT)` 时，虚拟机可以联网，但主机无法连接 NAT 下的虚拟机。

当虚拟机网卡连接方式为 `Host-Only`{: .text} 时，虚拟机无法联网，但主机可以连接虚拟机。主机会为 `Host-Only`{: .text} 网络分配一个虚拟网卡 `vboxnet0`，该网卡默认网关为 `192.168.56.1`，网段为 `192.168.56.0/24`。

所以可以为虚拟机设置两个网卡，一个采用 `网络地址转换(NAT)` 用于虚拟机联网，一个采用 `Host-Only`{: .text} 用于主机与虚拟机之间的双向访问。

接下来还需要对虚拟机进行设置，以便开启双网卡。这里需要注意的是**可能需要为两张网卡设置不同的 metric 值**，使得连接 NAT 的网卡为“主”网卡，Host-Only 网卡为“副”网卡。设置方式可以在网卡配置时指定，也可直接修改路由表。

这里仅列举虚拟机为 Ubuntu Server 时的配置方法，对于其他发行版也有参考价值。

🔽 **netplan 配置（较新版本的 Ubuntu 采用了 netplan）**
```yaml
network:
  ethernets:
    enp0s3:     # 连接 NAT
      dhcp4: true
    enp0s8:     # 连接 Host-Only
      dhcp4: false
      addresses:
        - 192.168.56.5/24
  version: 2
# 如果需要指定 metric，可以参考 netplace 配置文档
```

🔽 **/etc/network/interfaces（传统方式）**
```yaml
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0   # eth0 连接 NAT
iface eth0 inet dhcp
metric 10   # 这里指定了较小的 metric 值，使其成为“主”网卡

auto eth1   # eth1 连接 Host-Only
iface eth1 inet static
address 192.168.56.6
gateway 192.168.56.1
network 192.168.56.0
netmask 255.255.255.0
metric 100
```

此时，虚拟机即可联网，也可与主机之间进行双向访问。
