---
title: 文件管理与备份
date: 2016-08-24
author: SeisMan
categories: 胡言乱语
---

本文将介绍我如何管理我的电脑中的文件，包括如何组织文件夹以及如何备份。

<!--more-->

## 背景介绍

我目前使用的计算机有两台：

1. DELL台式机：安装 CentOS 7 系统，仅用于科研工作；
2. MacBook Pro：大部分时间用于日常使用，偶尔用于简单的科研；

## 组织文件与文件夹

Linux 机器完全用于工作， `$HOME` 目录下的目录结构如下：

~~~
.
|-- backup
|-- bin
|-- Datas
|-- Desktop
|-- Downloads
|-- Nutstore
|-- Script
|-- Src
`-- src.import
~~~

下面一一解释一下每个目录的作用。

### bin

在 `$HOME` 下建了一个 `bin` 目录，并将该目录路径加入到 `$PATH` 中。

`bin` 目录用于放置一些简单的二进制文件以及脚本，使得在命令行中可以直接调用它们。

简单列几个我的 `bin` 目录下的文件：

- `backup.pl`: 自己写的一个数据备份脚本，自动备份全盘数据
- `cpdf`: 一个PDF分隔、合并工具，从官网下载的软件包中有用的东西只有这一个二进制文件，所以就放在 `bin` 目录下
- `distaz`: 计算震中距和方位角的小程序
- `matlab`: 指向 matlab 命令的软链接
- `mseed2sac`: 将 miniSEED 转换为 SAC 的小程序
- `rdseed`: 将 SEED 转换成 SAC 的小程序
- `wlt.py`: 自动登陆科大网络通的脚本

哪些文件或脚本可以放在 `bin` 目录下呢，我自己设立的原则如下：

1. 如果一个软件包只提供了一个二进制文件，并且这个二进制文件可直接运行，而不依赖于其他辅助文件，比如 `distaz`、`rdseed` 等都满足这一点
2. 如果一个命令可以很好地完成某一类特定的功能而不需要对源码做任何改动，比如 `rdseed` 之类
3. 如果一个命令只能实现某个特定目的，并且无论如何修改不会对任何项目产生影响，比如我自己写的 `backup.pl` 和 `wlt.py`
4. 如果一个软件提供了一堆命令，但实际上真正用到的只有其中一个，比如 matlab，此时当然可以把 matlab 的路径加到 `$PATH` 中，也可以在 `bin` 目录下建一个指向命令的软链接

### Datas

该目录主要用于保存一些基本不会改动的数据文件，比如：

- `GEBCO`: 全球地形数据文件
- `USArray`: 科研过程中下载的 USArray 的 SEED 格式的波形数据，放在这里相当于备份
- `GFlib`: 格林函数库
- `China_Adm`: 中国国界数据

### src.import

该目录用于放置从网络获取的他人写的软件包的源码。比如 pssac、fk、gcap 的源码都放这里。

### Src

该目录用于放置自己写的具有 **通用性** 的代码包，比如：

- `FD2D-SH`: 二维SH波有限差分波场模拟代码
- `FD2D-PSV`: 二维P-SV波有限差分波场模拟代码
- `FD3D`: 三维有限差分波场模拟代码
- `sac_tools`: SAC 相关工具

### Script

该目录用于放置自己写的具有 **通用性** 的脚本：

- `HinetScripts`: 用于申请、下载、处理Hinet数据的 Python 脚本
- `FnetScript`: 用于从Fnet下载数据的脚本
- `irisScript`: 从IRIS获取数据的相关脚本
- `SodScript`: 使用SOD获取数据的相关脚本
- `TauPy`: 自己写的射线走时计算代码

### backup

该目录下主要用于放置从网站上下载的软件包的压缩包，比如SAC、GMT、TauP等的软件包。

该目录与 `src.import` 的主要区别在于， `src.import` 目录中包含的是解压后的源码，且可能自己会对其进行修改；而 `backup` 目录中包含的是软件包的原始压缩包，与从官方网站上下到的代码一模一样，以便于自己搞乱 `src.import` 里的代码之后重新开始。同时，也相当于是一个软件包的备份。

### Desktop

`Desktop` 是我日常工作的地方，放置当前在做的科研项目。比如，该目录下有这几个文件夹：

- `MyProject1`：第一个科研项目的根目录。与该项目所有的所有的数据、脚本、源码、图片、报告等都在这个目录下。
- `MyProject2`：第二个科研项目的根目录。同理。
- `MyProject3`：第n个科研项目的根目录。
- `workspace`：日常科研中，经常需要做一些测试，该目录用于进行临时的测试。我的目录下包括了 Perl、Python、SAC、GMT、TauP等目录。比如某个Perl语法忘了，我会到Perl目录下临时写个小脚本测试一下语法对不对；比如对SAC某个命令的功能不太确定，我会在SAC目录下临时生成几个SAC文件然后做个测试。在专门的目录下做一些测试，一方面可以避免由于写了临时文件而搞乱自己的科研目录，另一方面在删除临时文件时可以方向删而不必担心删除了其他重要的文件。

### Downloads

这是浏览器下载的默认目录。该目录仅作为临时存放文件的地方。每隔几天都需要将下载的文章、软件包、图片等等放在该放的地方。

### Nutstore

坚果云同步目录。坚果云会自动将将该目录下的所有文件在我的两台电脑之间同步。

### /opt/

我倾向于将一些比较大的软件装在 `/opt/` 目录而不是 `/usr/local/` 目录下。比如 SAC、GMT、TauP、Matlab 这些都装在该目录下。

## 硬盘备份

文件备份非常重要。备份主要是为了解决两个常见问题：

1. 磁盘损坏
2. 误删文件

因而建议，至少每周将工作电脑中的文件备份到移动硬盘中。推荐使用 `rsync` 进行备份，其用法为:

    rsync --delete -av /home/seisman/bin /data1/seisman/

该命令作用是将 `/home/seisman/bin` 目录完整同步到 `/data1/seisman/` 目录下。

`rsync` 的特色在于增量备份。这意味着只有第一次备份的时候需要花比较多的时间，以后再使用该命令进行备份时只会同步改动。加入你一周只修改了一个文件，那么同步的过程会在瞬间完成。
