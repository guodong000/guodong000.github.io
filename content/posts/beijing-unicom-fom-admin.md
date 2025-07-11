+++
date = '2024-07-17T17:42:48+08:00'
draft = false
title = '北京联通光猫进入超管后台'
hidesummary = true
+++

> ❗️ 2025 年之后设备固件在线更新，本方法已失效！

光猫设备：**烽火 HG6145D1**

## 开启设备 Telnet

访问 `http://192.168.1.1/telnet?enable=1&key=设备MAC` ，可在设备背面标签查看其 MAC 地址。

## 通过 Telnet 访问设备

`telnet 192.168.1.1`

登录名为 `admin`，密码为 `Fh@设备MAC后六位`

> 网上大多数教程接下来是 load_cli factory 进入工厂模式，然后 show admin_pwd 查看超管密码，最后通过 http://192.168.1.1/cu.html 登录超管后台。但我在实际操作中发现 show admin_pwd 返回的密码固定为 CU_Admin，并且无法用该密码登录，猜测应该是策略改了。

## 修改当前会话权限

首先通过正常方式登录光猫后台。

然后通过 Telnet 修改 `/var/fiberweb_session/session_xxx` 文件。

```text
config key 'user'
  option value '1'
  
# 将上方数字修改为以下对应数字即可
# 1: 普通用户
# 2: 超级管理
# 4: fiber工厂模式（无用）
```

修改完后刷新光猫后台 web 界面，即可发现当前界面已切换为超管后台。

## 注意事项

光猫每次重启后都需要重新开启 Telnet 访问模式。

每次登录光猫后台，都需要重新修改会话文件方可切换到超管后台。

经过测试，在我的设备上 ip6tables 直接将 FORWARD DROP 了，虽然可以通过 telnet 手动修改规则，但没有好的持久化方式（光猫重启后规则恢复默认）。所以如果打算搞 ipv6 公网访问，建议通过本方法开启光猫的 PPPoE 桥接模式，使用自家路由器拨号上网。

附：telnet 自动登录脚本

```bash
#!/usr/bin/expect -f
# Usage: ./script <ip>

set host [lindex $argv 0]
set login "admin"
set pass "Fh@XXXXXX"

spawn telnet $host

#expect "(none) login: "
expect "login: "
send "$login\n"

expect "Password: "
send "$pass\n"

interact
```

