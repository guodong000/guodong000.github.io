+++
date = '2022-03-16T12:30:26+08:00'
draft = false
title = 'MacOS Shell 启动配置文件'
summary = '介绍 MacOS 内的多种 Shell 启动配置文件，以及如何设置全局环境变量'
+++

`/etc/profile` `/etc/bashrc` `~/.bash_profile` `~/.profile` `~/.bashrc` 为 Shell 启动配置文件。

macOS 下，这些文件不会在系统登录的时候被执行，只会在 `Terminal.app` 等终端程序启动相应 Shell 时被加载执行。

## bash

当 bash 作为一个 interactive login shell 或者以 `bash --login` 启动时：

* 首先尝试执行 `/etc/profile` 文件。
* macOS 中的 `/etc/profile` 会调用 `/etc/bashrc`。
* 接下来依次寻找 `~/.bash_profile` `~/.bash_login` `~/.profile`，执行第一个找到的文件。
* 当 login shell 退出时，尝试执行 `~/.bash_logout`。

当 bash 不作为 login shell 启动时：

* 尝试执行 `~/.bashrc`。

> macOS 中的 ~/.bash_profile 会调用 ~/.profile 和 ~/.bashrc

## sh

sh 会使用 `/etc/profile` 和 `~/.profile` 文件

## 总结

macOS 下使用 bash 时，通常只需通过 ~/.bashrc 来设置环境变量即可。

## 如何全局设置 $PATH

我们需要看一下 $PATH 的构建过程：

* 加入 `/etc/paths` 中的每一行
* 加入目录 `/etc/paths.d/` 下每个文件的每一行
* shell 启动，执行 `/etc/profile` 等文件中的 `export PATH=xxx:$PATH` 语句

所以只需要在 `/etc/paths.d/` 目录下新建一个文件，并在其中添加指定的路径，即可将其添加到全局范围内的 `$PATH` 变量中。

> 经过测试，macOS 下通过 `/etc/paths` 或 `/etc/paths.d/` 设置 `$PATH` 变量后，新开 shell 会立即生效，无需重启。
