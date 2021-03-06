---
title: 在 C 程序中读写 SAC 文件
date: 2014-05-11 18:00
author: SeisMan
categories: SAC
tags: [C, SAC技巧]
---

## 序言

SAC 是进行地震数据预处理的好工具，但是无法实现所有的数据分析功能，这就需要能够在自己的程序中读写 SAC 文件。这篇博文介绍如何在 C 程序中读写 SAC 文件。

<!--more-->

## 子函数库

SAC 自带了读写函数库，并且提供了相关的示例程序，这些可以在 [《SAC 参考手册》](/sac-manual.html) 中的相关章节中找到。

SAC 自带的读写子函数实际上并不好用，因而就有很多人自己重新实现了 SAC 读写函数库，其中之一就是 Prof. Lupei Zhu 所写的 `sacio.c` 。

[Prof. Lupei Zhu](http://www.eas.slu.edu/People/LZhu/home.html) 的 `fk` 或者 `gCAP` 中都包含了 SAC 读写函数库， `sacio.c` 和 `sac.h` 。

## 子函数

`sac.h` 中定义了名为 `SACHEAD` 的结构体，其包含了 SAC 格式的所有头段变量。

`sacio.c` 中定义了如下子函数：

-   `read_sac`：读取 SAC 二进制数据
-   `read_sac2`：读取含 cut 选项的二进制数据
-   `read_sachead`：仅读取 SAC 文件中的头段部分
-   `write_sac`：写 SAC 二进制数据
-   `wrtsac2`：将两个 1 维数组写成 XY 形式的 SAC 数据
-   `sachdr`：创建一个全新的 SAC 头段

## 示例

### 读和写一个 SAC 文件

最常见的需求是读取一个 SAC 二进制文件，对数据进行处理，并将处理后的数据写回到原文件中。

``` C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "sac.h"        // 包含头文件 sac.h

int main (int argc, char *argv[]) {
    char    file[80];
    SACHEAD hd;
    float   *data;
    int     i;

    strcpy(file,"seis.SAC");

    // 读入数据
    data = read_sac(file, &hd);
    printf("npts=%d delta=%f \n", hd.npts, hd.delta);

    // 其它数据处理
    for (i=0; i<hd.npts; i++) {
        data[i] *= 0.1;
    }

    // 将结果写回到原文件中
    write_sac(file, hd, data);

    free(data);
    return 0;
}
```

需要注意的几个事项包括：

-   SAC 文件的头段区保存到结构体 `SACHEAD hd` 中。此处必须定义成结构体变量 `SACHEAD hd` ，若定义成结构体指针 `SACHEAD *hd` ，必须通过 `malloc` 为头段区分配空间。
-   SAC 文件的数据保存到指针 `float *data` 中，此处不需要对指针分配空间，`read_sac` 子函数会首先读取 SAC 文件的头段区，然后根据头段变量 `npts` 的值分配合适的内存空间。
-   对数据进行处理后，可以直接写回到原文件中，或写入到新文件中。
-   指针 `data` 在该程序中定义，并在子程序 `read_sac` 中分配内存，最终需要在该程序中 `free(data)` 将已分配的内存空间释放。在本例中内存是否释放并无太大影响，在有些情况下会出现 “内存溢出” 的问题。
-   本示例中，为了保持代码的简洁性，没有对子函数的返回值进行判断。

### 读取一个 SAC 文件的头段区

有些时候只需要 SAC 文件的头段区的一些信息，此时若读取整个文件就有些浪费了。

``` C
#include <stdio.h>
#include <string.h>
#include "sac.h"

int main (int argc, char *argv[]) {
    char    file[80];
    SACHEAD hd;

    strcpy(file,"seis.SAC");

    read_sachead(file, &hd);
    if (hd.npts>=500) {
        printf("Too much data points!\n");
    }

    return 0;
}
```

### 读取 SAC 文件中的一段数据

有些时候，数据可能有 10000s，而我们只需要其中 50s 的数据。为了获得 50s 的数据而读取 10000s 的数据，实在太浪费。因而需要一个有效的手段对数据进行截取，即相当于 SAC 中的 cut 命令。

``` C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "sac.h"

int main (int argc, char *argv[]) {
    char    fin[80];
    char    fout[80];
    SACHEAD hd;
    float   *data;
    int     tmark;
    float   t1, t2;
    int     i;

    strcpy(fin,"seis.SAC");

    tmark   =   -2;
    t1      =   -0.5;
    t2      =   2.5;

    data = read_sac2(fin, &hd, tmark, t1, t2);
    printf("npts=%d delta=%f\n", hd.npts, hd.delta);

    for (i=0; i<hd.npts; i++) {
        data[i] += 0.1;
    }

    strcpy(fout,"seis.SAC.cut");
    write_sac(fout, hd, data);

    free(data);
    return 0;
}
```

说明：

-   `tmark`、`t1` 和 `t2` 确定了要读取的数据的时间窗。其中 `tmark` 可以取如下值

    -   `tmark=-5`：以 b 为时间标记
    -   `tmark=-3`：以 o 为时间标记
    -   `tmark=-2`：以 a 为时间标记
    -   `tmark=0~9`：以 t0\~t9 中的某个为时间标记

    此例中，表示仅读取头段变量 a 前 0.5 秒到后 2.5 秒的数据。

-   在 `sacio.c` 的源代码中，理解 `tref = *((float *) hd + 10 + tmark);` 这一句很重要，在自己的程序中也会经常需要类似的代码。
-   虽然只读取了文件中的部分数据，该子程序对于头段区的 b、e、npts 等做了相应修改，因而最终的头段区是完全正确的。
-   因为只读取了文件中的部分数据，若将处理之后的数据写入原文件中，会导致原数据丢失，因而一般保持到新的文件中。

### 从零开始写一个 SAC 文件

在做合成数据时，经常需要从无到有完全创建一个 SAC 文件。这相对于一般的 “读 -\> 处理 -\>写”而言要更复杂一些，因为必须首先构建一个基本的头段区。

``` C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "sac.h"

int main (int argc, char *argv[]) {
    char    fout[80];
    SACHEAD hd;
    float   *data;

    float   delta;
    int     npts;
    float   b;

    int     i;

    delta = 0.01;       // 采样周期
    npts  = 1000;       // 数据点数
    b     = 10;         // 文件开始时间
    hd = sachdr(delta, npts, b);    // 构建基本头段
    hd.dist     = 10;   // 给其它头段变量赋值
    hd.cmpaz    = 0.0;
    hd.cmpinc   = 0.0;

    strcpy(fout,"seis.syn");
    // 生成合成数据
    data = (float *)malloc(sizeof(float)*npts);
    for (i=0; i<npts; i++) {
        data[i] = i;
    }

    // 写入到文件中
    write_sac(fout, hd, data);
    free(data);

    return 0;
}
```

### 创建一个 XY 型 SAC 文件

XY 型数据中包含了两个数据区，分别是自变量和因变量。这种类型的文件其实很少用到。

``` C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "sac.h"

int main (int argc, char *argv[]) {
    char    fout[80];
    float   *xarray;
    float   *yarray;

    int     npts;
    int     i;

    npts = 1000;
    strcpy(fout,"seis.syn");
    // 构建数据
    xarray = (float *)malloc(sizeof(float)*npts);
    yarray = (float *)malloc(sizeof(float)*npts);
    for (i=0; i<npts; i++) {
        xarray[i] = i*0.1;
        yarray[i] = i*i;
    }

    wrtsac2(fout, npts, xarray, yarray);

    free(xarray);
    free(yarray);

    return 0;
}
```

## 编译方法

    $ gcc prog.c  sacio.c -lm

## 一些问题

下面列举中 `sacio.c` 的一些问题：

1.  无法正确处理字符型的头段变量。由于 C 语言中字符串是以 `\0` 为结束符的，所以长度为 8 的字符型头段变量实际上需要额外的一个字节保存 `\0`，未考虑此问题会导致无法正确使用和修改字符型头段变量，且可能导致字符型头段变量中的信息丢失。
2.  写文件时未处理中断信号。在写文件的过程中，若出现中断信号，会导致文件损坏，若在写文件过程中遇到中断信号，应保证继续执行写操作或许会更好。
