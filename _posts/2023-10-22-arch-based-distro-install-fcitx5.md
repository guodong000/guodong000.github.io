---
layout: post
title: Arch Based 发行版安装 fcitx5
---

安装 fcitx5 相关软件。
```bash
# 如果不安装 fcitx5-gtk 可能会导致快速输入时漏字
sudo pacman -S fcitx5 fcitx5-configtool fcitx5-pinyin-zhwiki fcitx5-gtk fcitx5-chinese-addons
```

修改 `/etc/environment` 添加相关环境变量。
```bash
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
SDL_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```

最后重启电脑。
