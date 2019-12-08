title: 函数和方法的区别

tags: golang

categories: golang

------

** {{ title }}：** <Excerpt in index | 首页摘要>

go语言养成记之函数和方法的区别

<!-- more -->

[TOC]

在接触到`go`之前，我认为函数和方法只是同一个东西的两个名字而已（在我熟悉的`c/c++`，`python`，`java`中没有明显的区别），但是在`golang`中者完全是两个不同的东西。官方的解释是，方法是包含了接收者的函数。到底什么意思呢。

## 函数

首先函数的格式是固定的，func＋函数名＋ 参数 ＋ 返回值（可选） ＋ 函数体。例:

```go
func main() {
    fmt.Println("hello golang")
}
```

在golang中有两个特殊的函数，`main`函数和`init`函数，`main`函数不用介绍在所有语言中都一样，它作为一个程序的入口，只能有一个。`init`函数在每个package是可选的，可有可无，甚至可以有多个(但是强烈建议一个package中一个init函数)，init函数在你导入该package时程序会自动调用init函数，所以init函数不用我们手动调用,l另外它只会被调用一次，因为当一个package被多次引用时，它只会被导入一次。

```go
package my

import "fmt"

var I int

func init() {
    I = 0
    fmt.Println("Call my init1")
}

func init() {
    I = 1
    fmt.Println("Call my init2")
}
```

```go
package main

import (
    "my"
	"fmt"
)

func main() {
    fmt.Println("hello go .... I =",my.I)
}
```

**执行结果：**

```bash
Call my init1
Call my init2
hello go .... I = 1
```

## 方法

```go
package main

import "fmt"

type myint int

func (m *myint) double() int {
	*m = *m * 2
	return 0
}

func (m myint) square() int {
	m = m * m
	fmt.Printf("square is %d\n", m)
	return 0
}

func main() {
	var i myint = 2
	i.double()
	fmt.Println("i = ",i)
	i.square()
	fmt.Println("i = ",i)
}
```

**执行结果：**

```bash
i = 4
square is 16
i = 4
```

我们可以看到方法和函数的区别，方法在`func`关键字后是接收者而不是函数名，接收者可以是自己定义的一个类型，这个类型可以是`struct`，`interface`，甚至我们可以重定义基本数据类型。我们可以给他一些我们想要的方法来满足我们的实际工程中的需求，就像上面一样我重定义了int并给了它一个乘2和平法的方法，这里我们要注意一个细节，接收者是指针和非指针的区别，我们可以看到当接收者为指针式，我们可以通过方法改变该接收者的属性，但是非指针类型缺做不到。

这里的接收者和c++中的this指针有一些相似，我们可以把接受者当作一个class，而这些方法就是类的成员函数，当接收者为指针类型是就是c++中的非const成员函数，为非指针时就是const成员函数，不能通过此方法改变累的成员变量。