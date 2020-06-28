title: 内建容器

tags: golang

categories: golang

------

** {{ title }}：** <Excerpt in index | 首页摘要>

go语言养成记之内建容器

<!-- more -->

@[TOC]

## go语言内置rate包

#### Wait

```go
package main

import (
	"context"
	"fmt"
	"golang.org/x/time/rate"
	"log"
	"time"
)

func main() {
    // 设置需要限流的执行速率和容量
    // 1 代表执行速率，每次给桶内放入一个空间元素
    // 5 代表限流的最大容量
	r := rate.NewLimiter(1, 5)
	ctx := context.Background()
	for {
		err := r.Wait(ctx) // 每次消耗一个容量
		if err != nil {
			log.Fatal(err)
		}
		fmt.Println(time.Now().Format("2006-01-02 15:04:05"))
		time.Sleep(time.Second)
	}
}
```

输出结果：

```bash
2020-06-28 16:11:20
2020-06-28 16:11:21
2020-06-28 16:11:22
2020-06-28 16:11:23
2020-06-28 16:11:24
2020-06-28 16:11:25
2020-06-28 16:11:26
2020-06-28 16:11:27

Process finished with exit code 2
```

#### WaitN

```go
package main

import (
	"context"
	"fmt"
	"golang.org/x/time/rate"
	"log"
	"time"
)

func main() {
    // 设置需要限流的执行速率和容量
    // 1 代表执行速率，每次给桶内放入一个空间元素
    // 5 代表限流的最大容量
	r := rate.NewLimiter(1, 5)
	ctx := context.Background()
	for {
		err := r.WaitN(ctx, 2)
		if err != nil {
			log.Fatal(err)
		}
		fmt.Println(time.Now().Format("2006-01-02 15:04:05"))
		time.Sleep(time.Second)
	}
}
```

执行结果：

每次放入一个，每秒执行一次，每次执行会消耗2个容量空间。执行到四次以后总容量耗光，所以要等容量为2以后再执行，即每两秒执行一次。

```bash
2020-06-28 16:13:22
2020-06-28 16:13:23
2020-06-28 16:13:24
2020-06-28 16:13:25
2020-06-28 16:13:27
2020-06-28 16:13:29
2020-06-28 16:13:31
2020-06-28 16:13:33

Process finished with exit code 2
```

#### Allow

```go
package main

import (
	"fmt"
	"golang.org/x/time/rate"
	"time"
)

func main() {
	r := rate.NewLimiter(1, 5)
	for {
		if r.Allow() {
			fmt.Println(time.Now().Format("2006-01-02 15:04:05"))
		} else {
			fmt.Println("to many request")
		}
		time.Sleep(time.Second)
	}
}
```

输出结果：

```bash
2020-06-28 16:11:20
2020-06-28 16:11:21
2020-06-28 16:11:22
2020-06-28 16:11:23
2020-06-28 16:11:24
2020-06-28 16:11:25
2020-06-28 16:11:26
2020-06-28 16:11:27

Process finished with exit code 2
```

#### AllowN

```go
package main

import (
	"fmt"
	"golang.org/x/time/rate"
	"time"
)

func main() {
	r := rate.NewLimiter(1, 5)
	for {
		if r.AllowN(time.Now(), 2) {
			fmt.Println(time.Now().Format("2006-01-02 15:04:05"))
		} else {
			fmt.Println("to many request")
		}
		time.Sleep(time.Second)
	}
}
```

输出结果：

```bash
2020-06-28 16:24:33
2020-06-28 16:24:34
2020-06-28 16:24:35
2020-06-28 16:24:36
to many request
2020-06-28 16:24:38
to many request
2020-06-28 16:24:40
to many request

Process finished with exit code 2
```

#### HTTP请求限流

```go
package main

import (
	"golang.org/x/time/rate"
	"net/http"
)

func main() {
	r := rate.NewLimiter(1, 5)
	mux := http.NewServeMux()
	mux.HandleFunc("/", func(writer http.ResponseWriter, request *http.Request) {
		if r.Allow() {
			_, _ = writer.Write([]byte(http.StatusText(http.StatusOK)))
		} else {
			_, _ = writer.Write([]byte(http.StatusText(http.StatusTooManyRequests)))
		}
	})
	_ = http.ListenAndServe(":8099", mux)
}
```

无侵入版本

```go
package main

import (
	"golang.org/x/time/rate"
	"net/http"
)

var r = rate.NewLimiter(1, 5)

func myLimit(next http.Handler) http.Handler {
	return http.HandlerFunc(func(writer http.ResponseWriter, request *http.Request) {
		if !r.Allow() {
			_, _ = writer.Write([]byte(http.StatusText(http.StatusTooManyRequests)))
			return
		}
		next.ServeHTTP(writer, request)
	})
}

func main() {

	mux := http.NewServeMux()
	mux.HandleFunc("/", func(writer http.ResponseWriter, request *http.Request) {
		_, _ = writer.Write([]byte(http.StatusText(http.StatusOK)))
	})
	_ = http.ListenAndServe(":8099", myLimit(mux))
}
```

