+++
date = '2025-07-11T00:40:25+08:00'
draft = false
title = 'Memo'
summary = 'writing memo'
menus = 'main'
showtoc = true
weight = 999
+++

[Theme PaperMod Wiki](https://github.com/adityatelange/hugo-PaperMod/wiki/)

[Hugo Doc](https://gohugo.io/documentation/)

## Hugo 命令行

```bash
# 启动 Hugo 本地开发服务器
# -D, --buildDrafts 包含草稿
hugo server -D

# 创建页面
hugo new content content/posts/xxx.md
```

## 目录结构

[Hugo content organization](https://gohugo.io/content-management/organization/)

markup 页面位于 `content` 目录中，页面可以是一个单 `md` 文件，也可以是 Page Bundle。

Page Bundle 是以目录形式表示一个页面，对于常用的 其中包含 `index.md`、图像、其他页面等。

```text
site/
|-- content/
|   |-- posts/
|   |   |-- hello/
|   |   |   |-- index.md        # https://f.oo/posts/hello/
|   |   |   |-- other.md        # https://f.oo/posts/hello/other/
|   |   |   |-- example.png     # https://f.oo/posts/hello/exmpale.png
|   |   |-- world.md            # https://f.oo/posts/world/
```

## Markdown

### 格式

换行方式：
* 行尾两个以上空格
* `\`

### Images

```markdown
![Image title](path/to/image.png)

# 图片居中
![Image title](path/to/image.png#center)
```

使用 [figure Shortcode](https://gohugo.io/shortcodes/figure/)

> PaperMod 对 figure 做了增强，支持 `align`

```text
{{</* figure
    src="page/to/image.png"
    alt="Alt"
    width=128
    height=128
    title="Title"
    caption="Caption"
    align="center"
*/>}}
```

{{< figure
    src="lenna.png"
    alt="Alt"
    width=128
    height=128
    title="Title"
    caption="Caption"
    align="center"
>}}

### Code

[Hugo syntax highlighting](https://gohugo.io/content-management/syntax-highlighting/)

~~~markdown
```LANG {OPTIONS}
```

```c {lineno=inline]
#include <stdio.h>

int main(int argc, char **argv) {
    printf("Hello, world!\n");
}
```
~~~

