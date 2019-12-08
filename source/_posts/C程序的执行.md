title: 测试与性能调优

tags: C

categories: C程序的执行

------

** {{ title }}：** <Excerpt in index | 首页摘要>

C语言的基础入门

<!-- more -->

[TOC]

## 一段程序在计算机内部是怎么运行的

## 各种基本类型所占字节大小

```c
#include <stdio.h>

int main()
{
    printf("data type        number of bytes\n");
    printf("----------       ---------------\n");
    printf("char             %d\n", sizeof(char));
    printf("int              %d\n", sizeof(int));
    printf("short int        %d\n", sizeof(short));
    printf("long int         %d\n", sizeof(long));
    printf("float            %d\n", sizeof(float));
    printf("double           %d\n", sizeof(double));
    return 0;

}
// 执行结果如下：
// data type        number of bytes
// ----------       ---------------
// char             1
// int              4
// short int        2
// long int         4
// float            4
// double           8
```

