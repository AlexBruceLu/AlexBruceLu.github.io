title: gorountine

tags: golang

categories: golang

---

** {{ title }}：** <Excerpt in index | 首页摘要>

go语言养成记之gorountine

<!-- more -->

[TOC]

## goroutine基本介绍

### 进程和线程的介绍

1. 进程就是程序在操作系统中的一次执行过程，是系统进行资源分配和调度的基本单位
2. 线程是进程的一个执行实例，是程序执行的最小单元，它是比进程更小的能独立运行的基本单位
3. 一个进程可以创建和销毁多个线程，同一个进程中的多个线程可以并发执行
4. 一个程序至少有一个进程，一个进程至少有一个线程

### 并发和并行

1. 并发和并行
   1. 多线程程序在单核上运行，就是并发
   2. 多线程程序在多核上运行，就是并行
2. 小结
   1. `并发：`因为是在一个cpu上，比如有10个线程，每个线程执行10毫秒（进行轮询操作），从人的角度看，好像10个线程都在运行，但是在微观上看，在某一个时间点看，其实只有一个线程在执行，这就是并发。
   2. `并行：`因为是在多个cpu上（比如有10个cpu），比如有10个线程，每个线程执行10毫秒（各自在不同的cpu上执行），从人的角度看，这10个线程都在运行，但是从微观上看，在某一个时间点看，也同时有10个线程在执行，这就是并行。

### Go 协程和 Go 主线程

Go主线程（有人直接称为线程/也可以理解成进程）：一个Go线程上，可以起多个协程，可以理解为协程就是轻量级的线程

Go协程的特点：

1. 有独立的栈空间
2. 共享程序堆空间
3. 调度由用户控制
4. 协程是轻量级的线程

## goroutine快速入门

### 案例一

编写一个程序，完成如下功能：

1. 在主线程（可以理解成进程）中，开启一个goroutine，该协程每个一秒输出“hello world”

2. 在主线程中也每个一秒输出“hello golang”，输出10次后退出程序

3. 要求主线程和goroutine同时执行

4. 按上述要求画出流程图

   ![](https://github.com/AlexBruceLu/DAPP/wiki/gr1.png)

<font color="red">**代码实现：**</font>

```go
package main

import (
	"fmt"
	"time"
)

func test() {
	for i := 0; i < 10; i++ {
		fmt.Printf("test(%d) hello world\n",i+1)
		time.Sleep(time.Second)
	}
}

func main() {
	go test()
	for i := 0; i < 10; i++ {
		fmt.Printf("main(%d) hello golang\n",i+1)
		time.Sleep(time.Second)
	}
}
// 执行结果
// main(1) hello golang
// test(1) hello world
// test(2) hello world
// main(2) hello golang
// main(3) hello golang
// test(3) hello world
// test(4) hello world
// main(4) hello golang
// test(5) hello world
// main(5) hello golang
// test(6) hello world
// main(6) hello golang
// main(7) hello golang
// test(7) hello world
// test(8) hello world
// main(8) hello golang
// main(9) hello golang
// test(9) hello world
// main(10) hello golang
// test(10) hello world
```

#### 小结

1. 主线程是一个物理线程，直接作用在CPU上
2. 协程从主线程开启的，是轻量级的线程，是逻辑态的，消耗资源相对较少
3. `Golang`的协程机制是重要的特点，可以轻松的开启上万个协程。其他编辑语言的并发机制是一般基于线程的，开启过多的线程，消耗资源大，`Golang`的优势体现于此

## goroutine 的调度模型

### MPG 模型介绍

>1. **M:**操作系统的主线程（是物理线程）
>2. **P:**协程执行需要上下文
>3. **G:**协程

#### MPG 模式的运行状态1

![](https://githhub.com/AlexBrucelu/DAPP/wiki/gr2.png)

1. 当前程序有三个M，如果3个M都在一个CPU运行，就是并发，如果在不同的CPU运行就是并行
2. M1，M2，M3正在执行一个G，M1的协程队列有三个，M2的协程队列有三个，M3的协程队列有两个
3. go 的协程是逻辑态的，是轻量级的线程，go 可以起上万个协程
4. 其他程序是C/java的多线程，往往是内核态的，比较重量级，cpu资源被耗光

#### MPG 模式的运行状态2

### golang 的CPU运行数

go 1.8 前需要设置，之后的版本默认程序多核运行





