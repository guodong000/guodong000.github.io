---
layout: post
title: EndeavourOS 安装 fcitx5
---

安装 fcitx5 相关软件。
```bash
sudo pacman -S fcitx5 fcitx5-configtool fcitx5-pinyin-zhwiki
```

修改 `/etc/environment` 添加相关环境变量。
```bash
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
SDL_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```

最后重启电脑。
