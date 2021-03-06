---
title: Python 多版本共存之 pyenv
date: 2013-10-04
updated: 2016-07-30
author: SeisMan
categories: 编程
tags: [安装, Python]
---

经常遇到这样的情况：

- 系统自带的 Python 是 2.6，自己需要 Python 2.7 中的某些特性；
- 系统自带的 Python 是 2.x，自己需要 Python 3.x；

此时需要在系统中安装多个 Python，但又不能影响系统自带的 Python，即需要实现 Python 的多版本共存。[pyenv](https://github.com/yyuu/pyenv) 就是这样一个 Python 版本管理器。

<!--more-->

## 安装 pyenv

在终端执行如下命令以安装 pyenv 及其插件：

```bash
$ curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
```

安装完成后，根据提示将如下语句加入到 `~/.bashrc` 中:

```bash
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"   # 这句可以不加
```

然后重启终端即可。

## 安装 Python

### 查看可安装的版本

``` bash
$ pyenv install --list
```

该命令会列出可以用 pyenv 安装的 Python 版本。列表很长，仅列举其中几个:

    2.7.8   # Python 2 最新版本
    3.4.1   # Python 3 最新版本
    anaconda2-4.1.0  # 支持 Python 2.6 和 2.7
    anaconda3-4.1.0  # 支持 Python 3.3 和 3.4

其中 2.7.8 和 3.4.1 这种只有版本号的是 Python 官方版本，其他的形如 `anaconda2-4.1.0` 这种既有名称又有版本后的属于 “衍生版” 或发行版。

### 安装 Python 的依赖包

在编译 Python 过程中会依赖一些其他库文件，因而需要首先安装这些库文件，已知的一些需要预先安装的库如下。

在 CentOS/RHEL/Fedora 下:

    sudo yum install readline readline-devel readline-static
    sudo yum install openssl openssl-devel openssl-static
    sudo yum install sqlite-devel
    sudo yum install bzip2-devel bzip2-libs

在 Ubuntu下：

    sudo apt-get update
    sudo apt-get install make build-essential libssl-dev zlib1g-dev
    sudo apt-get install libbz2-dev libreadline-dev libsqlite3-dev wget curl
    sudo apt-get install llvm libncurses5-dev libncursesw5-dev

### 安装指定版本

用户可以使用 `pyenv install` 安装指定版本的 python。如果你不知道该用哪一个，推荐你安装 anaconda3 的最新版本，这是一个专为科学计算准备的发行版。

```bash
$ pyenv install anaconda3-4.1.0 -v
/tmp/python-build.20170108123450.2752 ~
Downloading Anaconda3-4.1.0-Linux-x86_64.sh...
-> https://repo.continuum.io/archive/Anaconda3-4.1.0-Linux-x86_64.sh
```

执行该命令后，会从给定的网址中下载安装文件 `Anaconda3-4.1.0-Linux-x86_64.sh`。但由于文件很大，通常下载需要很久。建议的做法是，先执行以上命令然后马上中断安装，这样就知道 pyenv 要下载的文件的链接。然后用户自己用其他更快的方式（比如wget、迅雷等等）从该链接中下载安装文件，并将安装文件移动到 `~/.pyenv/cache` 目录下（该目录默认不存在，用户要自行新建）。

以本文说的情况为例：

1. 执行 `pyenv install anaconda3-4.1.0 -v` 获取下载链接
2. 用wget从下载链接中获取文件 `Anaconda3-4.1.0-Linux-x86_64.sh`
3. 将安装包移动到 `~/.pyenv/cache/Anaconda3-4.1.0-Linux-x86_64.sh`
4. 重新执行 `pyenv install anaconda3-4.1.0 -v` 命令。该命令会检查 cache 目录下已有文件的完整性，若确认无误，则会直接使用该安装文件进行安装。

安装过程中，若出现编译错误，通常是由于依赖包未满足，需要在安装依赖包后重新执行该命令。

### 更新数据库

在安装 Python 或者其他带有可执行文件的模块之后，需要对数据库进行更新：

``` bash
$ pyenv rehash
```

### 查看当前已安装的 python 版本

``` bash
$ pyenv versions
* system (set by /home/seisman/.pyenv/version)
anaconda3-4.1.0
```

其中的星号表示当前正在使用的是系统自带的 python。

### 设置全局的 python 版本

``` bash
$ pyenv global anaconda3-4.1.0
$ pyenv versions
system
* anaconda3-4.1.0 (set by /home/seisman/.pyenv/version)
```

当前全局的 python 版本已经变成了 anaconda3-4.1.0。也可以使用 `pyenv local` 或 `pyenv shell`
临时改变 python 版本。

### 确认 python 版本

``` bash
$ python
Python 3.5.2 (Anaconda 4.1.0, Sep 10 2014, 17:10:18)
[GCC 4.4.7 20120313 (Red Hat 4.4.7-1)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

## 使用 python

- 输入 `python` 即可使用新版本的 python；
- 系统自带的脚本会以 `/usr/bin/python` 的方式直接调用老版本的 python，因而不会对系统脚本产生影响；
- 使用 `pip` 安装第三方模块时会自动按照到当前的python版本下，不会和系统模块发生冲突。
- 使用 `pip` 安装模块后，可能需要执行 `pyenv rehash` 更新数据库；

## pyenv 其他功能

1. `pyenv uninstall` 卸载某个版本
2. `pyenv update` 更新 pyenv 及其插件

## 参考

1.  <https://github.com/yyuu/pyenv>

## 修订历史

- 2013-10-04：初稿；
- 2014-10-07：将 Python 依赖包一段的位置提前；
- 2016-07-30：使用 `pyenv-installer` 安装；
