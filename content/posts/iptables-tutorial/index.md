+++
date = '2020-11-27T02:05:55+08:00'
draft = false
title = 'iptables 简明教程'
summary = '快速了解 iptables 的核心概念、包处理流程，理解规则的含义'
showtoc = true
hidesummary = false
+++

netfilter/iptables 是 Linux 系统下最常用的包过滤防护墙系统，iptables 工作在内核网络栈（kernel’s networking stack）中的一些包过滤钩子（packet filtering hooks）之上，这些内核中的包过滤钩子即 netfilter framework。

## 什么是 Netfilter Hooks ？
netfilter 中有五个 hook 可以被应用程序注册，当一个包经过内核网络栈时，会触发在相应 hook 上注册的内核模块，其触发规则依赖于包的流向、包的目的地以及之前节点对包的处理（其实 iptables 中的 rule 就是来定义这里的触发规则）。

| hook 名称 | 描述 |
| --- | --- |
| NF_IP_PRE_ROUTING | 触发于流量刚进入网络栈之后，且在路由选择之前。 |
| NF_IP_LOCAL_IN | 触发于流入包被路由选择识别目标地址为本机时。 |
| NF_IP_FORWARD | 触发于流入包被路由选择识别目标地址为其他主机时。 |
| NF_IP_LOCAL_OUT | 触发于本机创建的流出包刚进入网络栈时。 |
| NF_IP_POST_ROUTING | 触发于包即将流出网络栈进入物理传输之前，且在路由选择完成之后。 |

内核模块在注册到这些 hooks 时需要提供一个优先级参数，以便在多个模块同时注册一个 hook 的情况下决定触发的顺序。

## iptables 的核心概念

### 规则（Rules）

规则是网络管理员预定义的条件，规则一般的定义为“如果数据包头符合这样的条件，就这样处理这个数据包”。

规则存储在内核空间的信息包过滤表中，这些规则分别指定了源地址、目的地址、传输协议（如 TCP、UDP、ICMP）和服务类型（如 HTTP、FTP 和 SMTP）等。当数据包与规则匹配时，
iptables就根据规则所定义的方法来处理这些数据包，如放行（accept）、拒绝（reject）和丢弃（drop）等。配置防火墙的主要工作就是添加、修改和删除这些规则。

### 链（chains）

链是数据包传播的路径，每一条链其实就是众多规则中的一个检查清单，每一条链中可以有一条或数条规则。

当一个数据包到达一个链时，iptables 就会从链中第一条规则开始检查，看该数据包是否满足规则所定义的条件。

如果满足，系统就会根据该条规则所定义的方法处理该数据包；否则 iptables 将继续检查下一条规则，如果该数据包不符合链中任一条规则，iptables 就会根据该链预先定义的默认策略（Default Policy）来处理数据包。

iptables 内置 5 条链，即 PREROUTING、INPUT、FORWARD、OUTPUT、POSTROUTING，分别可以被 netfilter hooks 中的 NF_IP_PRE_ROUTING、NF_IP_LOCAL_IN、NF_IP_FORWARD、NF_IP_LOCAL_OUT、NF_IP_POST_ROUTING 触发，其触发条件可参考上文所述。

正是由于注册 netfilter hook 时需要指定优先级来决定触发顺序，所以不同表中的同类链的触发顺序是不同的，其触发顺序（优先级）为：Raw > Mangle > Nat > Filter（INPUT 链中 Filter 优先于 Nat）。

值得注意的是，除了内置链之外，用户可以自定义链，以便将一系列规则组织起来实现特定功能。自定义链需要被指定为规则的目标（Target）。

### 表（tables）

表提供特定的功能，现阶段 iptables 内置了 5 个表，即 raw 表、filter 表、nat 表、mangle 表以及 security 表。

| 表名 | 描述 |
| ---- | ------ |
| Raw | 只使用在 PREROUTING 链和 OUTPUT 链上。<br>因为优先级最高，从而可以对收到的数据包在连接跟踪前进行处理。一但用户使用了 raw 表在某个链上，raw 表处理完后，将跳过 nat 表和 ip_conntrack 处理，即不再做地址转换和数据包的链接跟踪处理了。 |
| Filter | 主要用于过滤数据包，该表根据系统管理员预定义的一组规则过滤符合条件的数据包。对于防火墙而言，主要利用在 filter 表中指定的规则来实现对数据包的过滤。filter 表是默认的表，如果没有指定哪个表，iptables 就默认使用 filter 表来执行所有命令。<br>filter 表包含了 INPUT 链、FORWARD 链、OUTPUT 链。<br>在 filter 表中只能允许对数据包进行接受（Accept）、丢弃（Drop、Reject）的操作，而无法对数据包进行更改。 |
| Nat | 主要用于网络地址转换（NAT），该表可以实现一对一、一对多、多对多等 NAT 工作，iptables 就是使用该表实现共享上网的。<br>nat 表包含了 PREROUTING 链、INPUT 链、FORWARD 链、OUTPUT 链、POSTROUTING 链。 |
| Mangle | 主要用于对指定数据包进行更改。<br>在内核版本 2.4.18 后的 Linux 版本中该表包含的链为：PREROUTING 链、INPUT 链、FORWARD 链、OUTPUT 链、POSTROUTING 链。 |
| Security | 用于强制访问控制（MAC）网络规则，由Linux安全模块（如SELinux）实现。<br>该表包含了 INPUT 链、FORWARD 链、OUTPUT 链。 |

简单来说，表 指定其中 规则 干什么，链 指定相应 规则 什么时候干。

## iptables 的包处理流程

{{< figure
    src="./nfk-traversal-1.png"
    align="center"
    caption="Netfilter Packet Traversal（由于版本原因，该图 nat 表未包含 FORWARD 与 INPUT 链）"
>}}

{{< figure
    src="./iptables-flowchart.png"
    align="center"
    caption="iptables processing flowchart"
>}}

在假设路由正确且 iptables 中所有包均被放行的情况下，iptables 中的数据流向有以下三种：
* **PREROUTING -> INPUT**：从外界到达防火墙的数据包，先被 PREROUTING 链处理（是否修改数据包地址等），之后会进行路由选择（判断该数据包应该发往何处），如果数据包 的目标地址是本机（比如 Internet 用户访问防火墙主机中的 web 服务器的数据包），那么内核将其传给 INPUT 链进行处理（决定是否允许通过等），通过之后再交给系统上层的应用程序（如 Apache 服务器）进行响应。
* **PREROUTING -> FORWARD -> POSTROUTING**：来自外界的数据包到达防火墙后，首先被 PREROUTING 链处理，之后会进行路由选择，如果数据包的目标地址是其它外部地址（比如局域网用户通过网关访问 Internet 站点的数据包），则内核将其传递给 FORWARD 链进行处理（是否转发或拦截），最后再交给 POSTROUTING 链（是否修改数据包的地址等）进行处理。
* **OUTPUT -> POSTROUTING**：防火墙本机向外部地址发送的数据包（比如在防火墙主机中测试公网 DNS 服务器时），首先被 OUTPUT 规则链处理，之后进行路由选择，最后传递给 POSTROUTING 链（是否修改数据包的地址等）进行处理。

iptables 是采用规则堆栈的方式来进行过滤，当一个包进入网卡，会先检查 PREROUTING，然后检查目标 IP 判断是否需要转发，之后会跳到 INPUT 或 FORWARD 进行过滤，如果封包需要转发则检查 POSTROUTING。如果包来自本机，则检查 OUTPUT 以及 POSTROUTING。在此过程中如果包符合某条规则则会进行处理，处理动作包括 ACCEPT、REJECT、DROP、REDIRECT、MASQUERADE、LOG、ULOG、DNAT、SNAT、MIRROR、QUEUE、RETURN、TOS、TTL、MARK等，其中某些处理动作不会中断过滤程序，而某些处理动作则会中断同一规则链的过滤，并依照前述流程继续进行下一个规则链的过滤，直到堆栈中的规则检查完毕为止。透过这种机制所带来的好处是，我们可以进行复杂、多重的封包过滤，简单来说，iptables 可以进行纵横交错式的过滤（tables）而非链状过滤（chains）。

## iptables 规则

规则位于指定表的指定链中，当一个包流经时，链中的规则会按照先后顺序与包进行匹配。

每条规则都包含一个匹配（Matching）部分和一个动作（Action）部分。匹配部分用于决定是否处理流经的包，动作部分（或称 目标 Target）则决定如何处理匹配到的包。

Target 即规则中对匹配到的包要执行的动作，通常可以分为两类：
* Terminating targets：终结当前链的处理并将控制权交回 netfilter hook，根据返回值的不同，hook 可能直接丢弃掉包（drop）或将包传递到下阶段处理。
* Non-terminating targets：执行一个动作后当前链继续处理其后的规则。

非终结目标中有一类被称为跳转目标（Jump Targets），用于跳转到其他链中进行额外的处理，直到跳转链中的所有规则处理完毕或执行 RETURN 目标跳转回之前链继续处理。此类目标可跳转到自定义链。

### 目标动作

iptables 规则中的目标有常用的 ACCEPT、REJECT、DROP、REDIRECT 、MASQUERADE 以及不常用的 LOG、ULOG、DNAT、RETURN、TOS、SNAT、MIRROR、QUEUE、TTL、MARK等。下面主要讲解一下常用的一些目标。

| 目标 | 描述 |
| ---- | ---- |
| ACCEPT | 将数据包放行，进行完此处理动作后，将不再比对当前链中其它规则，直接跳往下一个规则链。 |
| REJECT | 拦阻数据包，并传送数据包通知对方，可以传送的数据包有几个选择：ICMP port-unreachable、ICMP echo-reply 或 tcp-reset（这个数据包会要求对方关闭联机），进行完此处理动作后，将不再比对其它规则，直接中断过滤程序。<br>`iptables -A FORWARD -p TCP --dport 22 -j REJECT --reject-with tcp-reset` |
| DROP | 丢弃包不予处理，进行完此处理动作后，将不再比对其它规则，直接中断过滤程序。 |
| REDIRECT | 将包重新导向到另一个端口（PNAT），进行完此处理动作后，将会继续比对其它规则。 即端口映射。<br>`iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 8080` |
| MASQUERADE | 改写数据包 Source IP 为防火墙出口 IP，可以指定 port 对应的范围，进行完此处理动作后，其后没有任何规则链，所有处理均已完成。不同于 SNAT 必须指定 IP，这里的 IP 会从网卡直接读取，在 IP 不固定时（如使用拨号连接时，IP 由 DHCP 服务器指派），这时 MASQUERADE 特别有用。<br>`iptables -t nat -A POSTROUTING -p TCP -j MASQUERADE --to-ports 1024-31000` |
| LOG | 将包相关信息纪录在系统日志中，进行完此处理动作后，将会继续比对其他规则。<br>`iptables -A INPUT -p tcp -j LOG --log-prefix "INPUT packets"` |
| SNAT | 改写包 Source IP 为某特定 IP 或 IP 范围，可以指定 port 对应的范围，进行完此处理动作后，其后没有任何规则链，所有处理均已完成。<br>`iptables -t nat -A POSTROUTING -p tcp-o eth0 -j SNAT --to-source 194.236.50.155-194.236.50.160:1024-32000` |
| DNAT | 改写封包 Destination IP 为某特定 IP 或 IP 范围，可以指定 port 对应的范围，进行完此处理动作后，将会直接跳往下一个规则链（mangle:INPUT 或 mangle:FORWARD）。<br/>`iptables -t nat -A PREROUTING -p tcp -d 15.45.23.67 --dport 80 -j DNAT --to-destination 192.168.1.1-192.168.1.10:80-100` |
| MIRROR | 镜像数据包，也就是将 Source IP 与 Destination IP 对调后，将数据包送回，进行完此处理动作后，将会中断过滤程序。 |
| QUEUE | 将数据包放入队列（如 ip_queue）中，交由用户空间（user space）的相应程序处理，进行完此处理动作后，将会中断过滤程序。 |
| RETURN | 结束在目前规则链中的过滤程序，返回主规则链继续过滤。等同于执行到当前链末尾。<br>对内建链而言，接下来执行该链的 Policy；对自定义链而言，则是跳回之前链中继续处理。 |
| MARK | 将数据包标上某个记号，以便提供作为后续过滤的条件判断依据，进行完此处理动作后，将会继续比对其它规则。<br>`iptables -t mangle -A PREROUTING -p tcp --dport 22 -j MARK --set-mark 2` |

### 状态（State）

连接追踪系统（connection tracking system）由 state 模块实现，这里所说的连接是一个抽象概念，简单来说只要两台机器建立了“你来我往”的通讯，就算建立起了连接。

* NEW：连接中的第一个包，状态就是NEW，我们可以理解为新连接的第一个包的状态为NEW。比如 TCP SYN 包。
* ESTABLISHED：：我可以把NEW状态包后面的包的状态理解为ESTABLISHED，表示连接已建立。比如 TCP SYN/ACK。
* RELATED：当数据包不属于追踪系统中任何已有连接，但和某已有连接有关系时，标记为 RELATED。比如 FTP 数据传输连接与命令连接相关联。
* INVALED：如果一个包没有办法被识别，或者这个包没有任何状态，那么这个包的状态就是INVALID，我们可以主动屏蔽状态为INVALID的报文。
* UNTRACKED：如果一个包在 raw 表中的某个链上进行了处理，这个包将不再被追踪，此时该包被标记为 UNTRACKED。

## 参考资料
* https://blog.csdn.net/reyleon/article/details/12976341
* https://www.digitalocean.com/community/tutorials/a-deep-dive-into-iptables-and-netfilter-architecture
* http://www.zsythink.net/archives/tag/iptables/
* https://netfilter.org/documentation/HOWTO/cn/packet-filtering-HOWTO.html
* https://unix.stackexchange.com/a/189906
* https://rlworkman.net/howtos/iptables/chunkyhtml/c962.html
* http://linux-ip.net/pages/diagrams.html
* https://stuffphilwrites.com/2014/09/iptables-processing-flowchart/