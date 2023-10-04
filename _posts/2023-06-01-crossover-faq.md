---
layout: post
title: CrossOver 疑难杂症
---

## 字体发虚

![winecfg 字体发虚](/assets/images/2023-06-01-crossover-faq/winecfg_font_gray.png)

修改注册表项 `[HKEY_CURRENT_USER/Control Panel/Desktop]`，修改完后正常安装其他中文字体即可。

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

## 修改中文字体

1. 将中文字体放到 `C:\\windows\Fonts\` 目录下。
2. 修改注册表项 `[HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\FontLink\SystemLink]` - `Tahoma`，将上述中文字体的文件名添加到数据首行。

**原理**：`Tahoma` 为 Windows 的默认字体，由于其为英文字体，系统在显示字体不支持的字符时会从字体的 Fallback 列表中查找适配字体，默认 `Tahoma` 的首个 Fallback 为 `SimSun.TFF`（宋体），所以通过修改 `Tahoma` 的 Fallback 即可设置中文字体。同样直接将 `SimSun.tff` 直接放到 `C:\\windows\Fonts\` 目录下也可以将中文字体设置为宋体。


## 删除快捷方式

![CrossOver Shortcuts](/assets/images/2023-06-01-crossover-faq/crossover_panel_shortcuts.png)

直接修改 `[Bottle_Folder]/desktopdata/cxmenu/cxmenu_macosx.plist` 可立即生效。但“清空并重建程序菜单”后，`cxmenu_macosx.plish` 会被重新生成，所以需要进行一系列修改：
1. 删除 Windows 中的快捷方式（常见位置）：
    * `driver_c/users/crossover/AppData/Roaming/Microsoft/Windows/Start Menu/`
    * `driver_c/users/crossover/Desktop/`
    * `driver_c/ProgramData/Microsoft/Windows/Start Menu/`
2. 修改 `[Bottle_Folder]/cxmenu.conf`。
3. 点击 `CrossOver菜单 > 配置 > 清空并重建程序菜单` 后 `cxmenu_macosx.plish` 会自动更新无需手动修改。
