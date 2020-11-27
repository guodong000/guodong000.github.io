---
layout: post
title: Ubuntu 18.04 配置 Shadowsocks-Qt5 & PAC
---

Ubuntu 下暂时没有像 Windows 下的功能完整的 Shadowsocks 客户端，这里采用的客户端为 Shadowsocks-Qt5，然后手动集成 PAC 文件。本文没有详细介绍每部操作，仅提供主要流程、工具参考以及注意事项。

- [Shadowsocks-Qt5]（客户端）
- [genpac]（PAC文件生成工具）

## Step-1 安装 Shadowsocks-Qt5

Shadowsocks-Qt5 的 Release 为打包的 AppImage 文件，AppImage 文件可以在 Ubuntu 下双击直接运行。

可以使用 AppImageLauncher 将 AppImage 打包的程序集成到系统中，这样可以在更为方便的运行程序。具体方法可参考： https://github.com/TheAssassin/AppImageLauncher

## Step-2 配置 Shadowsocks-Qt5

如何配置服务器这类常识性问题这里就不赘述了，配置时建议将对同一个服务器进行两次配置，一个配置选择 `本地服务器类型` 为 HTTP 并指定一个唯一端口，另一个配置选择 `本地服务器类型` 为 SOCKS5 并指定另一个唯一端口。

这样做的目的主要是为了同时暴露出 HTTP 以及 SOCKS5 代理端口，以便更方便的使用。

## Step-3 安装 genpac 工具

genpac 可通过 pip 进行安装，优点在于可以基于 gfwlist 生成对应 PAC 文件，而且可以指定用户规则。

本机需要安装 Pyhon 以及 pip，具体的安装及使用方式参考其 [Github 页面](https://github.com/JinnLynn/genpac)。

![Ubuntu 18.04 设置自动代理 PAC 文件](/assets/images/shadowsocks-qt5-with-pac-autoproxy-on-ubuntu-1804/screenshot.png)
*🖼 Ubuntu 18.04 设置自动代理 PAC 文件*

生成的 PAC 文件可以在 Ubuntu 自动代理设置中指定，一般可以通过添加 file:// 前缀指定 PAC URL，如 PAC 文件位置为 /home/xxx/ss/autoproxy.pac，则对应的 URL 为 file:///home/xxx/ss/autoproxy.pac 。

**要注意一点，上面所说的这种指定文件位置的方式可以被 Firefox 正确识别，但 Chrome 现在不再支持 file:// 这类本地文件链接，所以无法使用代理，这就需要本地通过 Nginx 之类的服务器将 PAC 文件暴露为一个 HTTP URL，然后在自动代理设置中指定。**

## Step-4 便捷使用

到这里基本工具已经全部安装并可以正常使用，但是在日常使用中使用起来并不方便，下面提供一些自定义脚本来优化操作。

**updatepac.sh 一键从 gfwlist 更新本地 PAC文件**

```sh
#!/bin/sh

BASE_DIR=/home/guodong/shadowsocks
PROXY="SOCKS5 127.0.0.1:1086"
GFWLIST_URL="https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt"
GFWLIST_LOCAL=$BASE_DIR/gfwlist.txt
USER_RULES=$BASE_DIR/user-rules.txt
OUTPUT=$BASE_DIR/autoproxy.pac

echo "PAC base folder: $BASE_DIR"
echo "Proxy: $PROXY"
echo "Gfwlist URL: $GFWLIST_URL"
echo 
echo "Update local pac file..."

genpac --format=pac --pac-proxy "$PROXY" --pac-compress \
        --gfwlist-url "$GFWLIST_URL" --gfwlist-proxy "$PROXY" \
        --gfwlist-local "$GFWLIST_LOCAL" --gfwlist-update-local \
        --user-rule-from "$USER_RULES" \
        --output "$OUTPUT"

echo "PAC URL: file://$OUTPUT"
echo "Chrome doesn't support file:// and data: protocol, so need use http url"
echo "PAC URL: http://localhost:1088/autoproxy.pac"
```


[Shadowsocks-Qt5]: https://github.com/shadowsocks/shadowsocks-qt5
[genpac]: https://github.com/JinnLynn/genpac