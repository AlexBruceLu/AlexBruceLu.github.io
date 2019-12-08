title: channel

tags: golang

categories: golang

---

** {{ title }}：** <Excerpt in index | 首页摘要>

go语言养成记之channel

<!-- more -->

## channel （管道）

### 一个小例子

计算1-200的各个数的阶乘，并且把各个数的阶乘放入到map中。最后显示出来，要求使用goroutine完成

```go
package main

import (
	"fmt"
	"time"
)

var A = make(map[int]int,10)

func factorial(i int) {
	ret := 1
	for j := 1; j <= i; j++ {
		ret *= j
	}
	A[i] = ret
}

func main() {
	for i := 1; i <= 200; i++ {
		go factorial(i)
	}
	time.Sleep(30*time.Second)
	for key, value := range A {
		fmt.Printf("map[%d] = %d\n",key,value)
	}
}
// 执行结果
// fatal error: concurrent map writes
```

**出现原因：对全局map变量没有加锁，会出现资源争夺问题**

**解决方式：1. 全局变量互斥锁  2. channel**

```go
package main

import (
	"fmt"
	"sync"
)

var A = make(map[int]int,10)
var lock = sync.Mutex{}

func factorial(i int) {
	ret := 1
	for j := 1; j <= i; j++ {
		ret *= j
	}
	lock.Lock()
	A[i] = ret
	lock.Unlock()
}

func main() {
	for i := 1; i <= 200; i++ {
		go factorial(i)
	}
	//time.Sleep(30*time.Second)
	lock.Lock()
	for key, value := range A {
		fmt.Printf("map[%d] = %d\n",key,value)
	}
	lock.Unlock()
}
// 互斥锁
```

### 为什么需要channel

1. 使用加锁来解决goroutine的通讯，方案不完美
2. 主线程休眠时间长了，会加长等到时间，如果等待时间短了，可能goroutine还处于工作状态，届时因主线程的退出而销毁
3. 通过变量加锁同步实现通讯，也并不利用多个协程对全局变量的读写操作
4. 所以引出channel

### channel 的基本介绍

1. channel 本质就是一个数据结构- 队列
2. 数据时先进先出 【FIFO：frist in frist out】
3. 线程安全，多goroutine访问时，不需要加锁，就是说channel 本身就是线程安全的
4. `channel` 有类型的，一个`string`类型的`channel`只能存放`string`类型的数据
5. `channel` 是引用类型

```go
package main

import (
	"fmt"
)

func factorialWrite(i chan int) {
	ret := 1
	for j := 1; j <= 20; j++ {
		ret *= j
		i <- ret
		fmt.Println("write data ",j)
	}
	close(i)
}

func factorialRead(i chan int,b chan bool) {
	for {
		v,ok := <- i
		if !ok {
			break
		}
		fmt.Println("read data ",v)
	}
	b <- true
	close(b)
}

func main() {
	i := make(chan int,20)
	b := make(chan bool,1)
	go factorialWrite(i)
	go factorialRead(i,b)
	<- b
}
```

**执行结果：**

```bash
read data  1
write data  1
write data  2
write data  3
write data  4
write data  5
write data  6
write data  7
write data  8
write data  9
write data  10
write data  11
write data  12
write data  13
write data  14
write data  15
write data  16
write data  17
write data  18
write data  19
write data  20
read data  2
read data  6
read data  24
read data  120
read data  720
read data  5040
read data  40320
read data  362880
read data  3628800
read data  39916800
read data  479001600
read data  6227020800
read data  87178291200
read data  1307674368000
read data  20922789888000
read data  355687428096000
read data  6402373705728000
read data  121645100408832000
read data  2432902008176640000
```





