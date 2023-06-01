---
layout: post
title: CrossOver 疑难杂症
---

## 字体发虚

![winecfg 字体发虚](/assets/images/2023-06-01-crossover-faq/winecfg_font_gray.png)

修改注册表项 `HKEY_CURRENT_USER/Control Panel/Desktop`，修改完后正常安装其他中文字体即可。

```
FontSmoothing=2 (string key)
FontSmoothingType=0x00000002 (dword key)
FontSmoothingGamma=0x00000578 (dword key)   # 貌似任何值都可以
FontSmoothingOrientation=0x00000001 (dword key) 
```

* FontSmoothing
    * Value "0" or "1": disables font smoothing
    * Value "2": enables font smoothing
* FontSmoothingType
    * Value "0" or "1": switch to gray font smoothing
    * Value "2": switch to colored font smoothing
* FontSmoothingGamma
    * Value between 0 to 2200 decimal: Intensity of color. 0=dark, 2200=light
* FontSmoothingOrientation
    * Value "0": CRT (Value "1" and "2" are LCD)
    * Value "1": RGB format (red, green, blue), normal
    * Value "2": BGR format (blue, green, red)

参考：
- [Enabling subpixel rendering/anti-aliasing in CrossOver.](https://www.codeweavers.com/support/wiki/linux/faq/cxofficeantialias)
- [StackExchange Answer](https://superuser.com/a/945614)

## 删除快捷方式

![CrossOver Shortcuts](/assets/images/2023-06-01-crossover-faq/crossover_panel_shortcuts.png)

直接修改 `[Bottle_Folder]/desktopdata/cxmenu/cxmenu_macosx.plist` 可立即生效。但修改 Bottle 名称后，`cxmenu_macosx.plish` 会被重新生成，需要进行一系列修改：
* 删除 Windows 中的快捷方式：
    * `driver_c/users/crossover/AppData/Roaming/Microsoft/Windows/Start Menu/`
    * `driver_c/users/crossover/Desktop/`
* 修改 `[Bottle_Folder]/cxmenu.conf`
* 修改 `[Bottle_Folder]/desktopdata/cxmenu/cxmenu_macosx.plist`
