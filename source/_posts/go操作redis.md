title: go操作Redis

tags: Redis

categories: database

------

** {{ title }}：** <Excerpt in index | 首页摘要>

Redis的简介及golang简易操作redis

<!-- more -->

[TOC]

## redis简介

redis（REmote DIctionary Server）是一个由Salvatore Sanfilippo 写的`Key-value`存储系统，由`C`语言编写、遵守`BSD协议`、支持网络、可基于内存亦可持久化的日志型、`Key-Value`类型的数据库，并提供多种语言的API。和Memcached类似，它支持存储的value类型相对更多，包括`string(字符串)`、`list(链表)`、`set(集合)`、`zset(sorted set --有序集合)`和`hash（哈希类型）`。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步，redis在3.0版本推出集群模式。

官方网站：https://redis.io/

## 源码部署

```bash
$ yum install gcc -y  #安装C依赖
$ wget http://download.redis.io/redis-stable.tar.gz  #下载稳定版本
$ tar zxvf redis-stable.tar.gz  #解压
$ cd redis-stable
$ make PREFIX=/opt/app/redis install   #指定目录编译
$ make install
$ mkdir /etc/redis   #建立配置目录
$ cp redis.conf /etc/redis/6379.conf # 拷贝配置文件
$ cp utils/redis_init_script /etc/init.d/redis  #拷贝init启动脚本针对6.X系统
$ chmod a+x  /etc/init.d/redis  #添加执行权限

# 修改配置文件：
$ vi /etc/redis/6379.conf
bind 0.0.0.0      #监听地址
maxmemory 4294967296   #限制最大内存（4G）：
daemonize yes   #后台运行
```

### 启动与停止

```bash
####启动与停止
$ /etc/init.d/redis start
$ /etc/init.d/redis stop
```

### 查看版本信息

```bash
#执行客户端工具
$ redis-cli 
#输入命令info
127.0.0.1:6379> info
# Server
redis_version:4.0.10
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:cf83e9c690dbed33
redis_mode:standalone
os:Linux 2.6.32-642.el6.x86_64 x86_64
arch_bits:64
multiplexing_api:epoll
```

## golang操作redis

### 安装

golang操作redis的客户端包有多个比如`redigo`、`go-redis`，github上Star最多的莫属`redigo`。

github地址：https://github.com/garyburd/redigo  

目前已经迁移到：https://github.com/gomodule/redigo 

文档：https://godoc.org/github.com/garyburd/redigo/redis

```bash
$ go get github.com/garyburd/redigo/redis
```

```go
import "github.com/garyburd/redigo/redis"
```

### 连接

`Conn`接口是与`Redis`协作的主要接口，可以使用`Dial`,`DialWithTimeout`或者`NewConn`函数来创建连接，当任务完成时，应用程序必须调用`Close`函数来完成操作。

```go
package main

import (
	"github.com/garyburd/redigo/redis"
	"fmt"
)

func main()  {
    conn,err := redis.Dial("tcp","10.1.210.69:6379")
    if err != nil {
        fmt.Println("connect redis error :",err)
        return
    }
    defer conn.Close()
}
```

### 命令操作

通过使用Conn接口中的do方法执行redis命令，redis命令大全参考：http://doc.redisfans.com/

go中发送与响应对应类型：

`Do`函数会必要时将参数转化为二进制字符串

| **Go Type**     |           **Conversion**            |
| :-------------- | :---------------------------------: |
| []byte          |             Sent as is              |
| string          |             Sent as is              |
| int, int64      |        strconv.FormatInt(v)         |
| float64         | strconv.FormatFloat(v, 'g', -1, 64) |
| bool            |      true -> "1", false -> "0"      |
| nil             |                 ""                  |
| all other types |            fmt.Print(v)             |

Redis 命令响应会用以下Go类型表示：

| Redis Type    | Go Type                                   |
| ------------- | ----------------------------------------- |
| error         | redis.Error                               |
| interger      | int64                                     |
| simple string | string                                    |
| bulk string   | []byte or nil if value not present        |
| arrary        | []interface{} or nil if value not present |

可以使用GO的类型断言或者`reply`辅助函数将返回的`interface{}`转换为对应类型。

#### 操作示例：**`get`、`set`**

```go
package main

import (
	"github.com/garyburd/redigo/redis"
	"fmt"
)

func main()  {
    conn,err := redis.Dial("tcp","10.1.210.69:6379")
    if err != nil {
        fmt.Println("connect redis error :",err)
        return
    }
    defer conn.Close()
    _, err = conn.Do("SET", "name", "wd")
    if err != nil {
        fmt.Println("redis set error:", err)
    }
    name, err := redis.String(conn.Do("GET", "name"))
    if err != nil {
        fmt.Println("redis get error:", err)
    } else {
        fmt.Printf("Got name: %s \n", name)
    }
}
```

#### 设置key过期时间

```go
_, err = conn.Do("expire", "name", 10) //10秒过期
    if err != nil {
        fmt.Println("set expire error: ", err)
        return
    }
```

#### 批量获取mget、批量设置mset

```go
_, err = conn.Do("MSET", "name", "wd","age",22)
    if err != nil {
        fmt.Println("redis mset error:", err)
    }
    res, err := redis.Strings(conn.Do("MGET", "name","age"))
    if err != nil {
        fmt.Println("redis get error:", err)
    } else {
        res_type := reflect.TypeOf(res)
        fmt.Printf("res type : %s \n", res_type)
        fmt.Printf("MGET name: %s \n", res)
        fmt.Println(len(res))
    }
//结果：
//res type : []string 
//MGET name: [wd 22] 
//2
```

#### 列表操作

```go
package main

import (
	"github.com/garyburd/redigo/redis"
	"fmt"
    "reflect"
)


func main()  {
    conn,err := redis.Dial("tcp","10.1.210.69:6379")
    if err != nil {
        fmt.Println("connect redis error :",err)
        return
    }
    defer conn.Close()
    _, err = conn.Do("LPUSH", "list1", "ele1","ele2","ele3")
    if err != nil {
        fmt.Println("redis mset error:", err)
    }
    res, err := redis.String(conn.Do("LPOP", "list1"))
    if err != nil {
        fmt.Println("redis POP error:", err)
    } else {
        res_type := reflect.TypeOf(res)
        fmt.Printf("res type : %s \n", res_type)
        fmt.Printf("res  : %s \n", res)
    }
}
//res type : string 
//res  : ele3 
```

#### hash操作

```go
package main

import (
	"github.com/garyburd/redigo/redis"
	"fmt"
    "reflect"
)

func main()  {
    conn,err := redis.Dial("tcp","10.1.210.69:6379")
    if err != nil {
        fmt.Println("connect redis error :",err)
        return
    }
    defer conn.Close()
    _, err = conn.Do("HSET", "student","name", "wd","age",22)
    if err != nil {
        fmt.Println("redis mset error:", err)
    }
    res, err := redis.Int64(conn.Do("HGET", "student","age"))
    if err != nil {
        fmt.Println("redis HGET error:", err)
    } else {
        res_type := reflect.TypeOf(res)
        fmt.Printf("res type : %s \n", res_type)
        fmt.Printf("res  : %d \n", res)
    }
}
//res type : int64 
//res  : 22 
```

#### Pipelining(管道)

管道操作可以理解为并发操作，并通过`Send()`，`Flush()`，`Receive()`三个方法实现。客户端可以使用`send()`方法一次性向服务器发送一个或多个命令，命令发送完毕时，使用`flush()`方法将缓冲区的命令输入一次性发送到服务器，客户端再使用`Receive()`方法依次按照先进先出的顺序读取所有命令操作结果。

```go
Send(commandName string, args ...interface{}) error
Flush() error
Receive() (reply interface{}, err error)
```

- `Send`：发送命令至缓冲区
- `Flush`:  清空缓冲区，将命令一次性发送至服务器
- `Recevie`:  依次读取服务器响应结果，当读取的命令未响应时，该操作会阻塞

```go
package main

import (
	"github.com/garyburd/redigo/redis"
	"fmt"
)

func main()  {
    conn,err := redis.Dial("tcp","10.1.210.69:6379")
    if err != nil {
        fmt.Println("connect redis error :",err)
        return
    }
    defer conn.Close()
    conn.Send("HSET", "student","name", "wd","age","22")
    conn.Send("HSET", "student","Score","100")
    conn.Send("HGET", "student","age")
    conn.Flush()

    res1, err := conn.Receive()
    fmt.Printf("Receive res1:%v \n", res1)
    res2, err := conn.Receive()
    fmt.Printf("Receive res2:%v\n",res2)
    res3, err := conn.Receive()
    fmt.Printf("Receive res3:%s\n",res3)
}
//Receive res1:0 
//Receive res2:0
//Receive res3:22
```

#### 发布/订阅

redis本身具有发布订阅的功能，其发布订阅功能通过命令SUBSCRIBE(订阅)／PUBLISH(发布)实现，并且发布订阅模式可以是多对多模式还可支持正则表达式，发布者可以向一个或多个频道发送消息，订阅者可订阅一个或者多个频道接受消息。

> - 示意图：
>
>   - 发布者：
>
>     ![](https://github.com/AlexBruceLu/DAPP/wiki/rd1.png)
>
>   - 订阅者：
>
>     ![](https://github.com/AlexBruceLu/DAPP/wiki/rd2.png)
>
>

操作示例，示例中将使用两个`goroutine`分别担任发布者和订阅者角色进行演示：

```go
package main

import (
    "github.com/garyburd/redigo/redis"
    "fmt"
    "time"
)

func Subs() {  //订阅者
    conn, err := redis.Dial("tcp", "10.1.210.69:6379")
    if err != nil {
        fmt.Println("connect redis error :", err)
        return
    }
    defer conn.Close()
    psc := redis.PubSubConn{conn}
    psc.Subscribe("channel1") //订阅channel1频道
    for {
        switch v := psc.Receive().(type) {
        case redis.Message:
            fmt.Printf("%s: message: %s\n", v.Channel, v.Data)
        case redis.Subscription:
            fmt.Printf("%s: %s %d\n", v.Channel, v.Kind, v.Count)
        case error:
            fmt.Println(v)
            return
        }
    }
}

func Push(message string)  { //发布者
    conn, _ := redis.Dial("tcp", "10.1.210.69:6379")
    _,err1 := conn.Do("PUBLISH", "channel1", message)
    if err1 != nil {
    	fmt.Println("pub err: ", err1)
        return
    }
}

func main()  {
    go Subs()
    go Push("this is wd")
    time.Sleep(time.Second*3)
}
//channel1: subscribe 1
//channel1: message: this is wd
```

#### 事务操作

`MULTI`, `EXEC`,`DISCARD`和`WATCH`是构成Redis事务的基础，当然我们使用go语言对redis进行事务操作的时候本质也是使用这些命令。

`MULTI`：开启事务

`EXEC`：执行事务

`DISCARD`：取消事务

`WATCH`：监视事务中的键变化，一旦有改变则取消事务

<font color="red">**示例：**</font>

```go
package main

import (
	"github.com/garyburd/redigo/redis"
	"fmt"
)

func main()  {
    conn,err := redis.Dial("tcp","10.1.210.69:6379")
    if err != nil {
        fmt.Println("connect redis error :",err)
        return
    }
    defer conn.Close()
    conn.Send("MULTI")
    conn.Send("INCR", "foo")
    conn.Send("INCR", "bar")
    r, err := conn.Do("EXEC")
    fmt.Println(r)
}
//[1, 1]
```

#### 连接池使用

`redis`连接池是通过`pool`结构体实现，以下是源码定义，相关参数说明已经备注：

```go
type Pool struct {
    // Dial is an application supplied function for creating and configuring a
    // connection.
    //
    // The connection returned from Dial must not be in a special state
    // (subscribed to pubsub channel, transaction started, ...).
    Dial func() (Conn, error) //连接方法

    // TestOnBorrow is an optional application supplied function for checking
    // the health of an idle connection before the connection is used again by
	// the application. Argument t is the time that the connection was returned
    // to the pool. If the function returns an error, then the connection is
    // closed.
    TestOnBorrow func(c Conn, t time.Time) error

    // Maximum number of idle connections in the pool.
    MaxIdle int  //最大的空闲连接数，即使没有redis连接时依然可以保持N个空闲的连接，而不被清除，随时处于待命状态

    // Maximum number of connections allocated by the pool at a given time.
    // When zero, there is no limit on the number of connections in the pool.
    MaxActive int //最大的激活连接数，同时最多有N个连接

    // Close connections after remaining idle for this duration. If the value
    // is zero, then idle connections are not closed. Applications should set
    // the timeout to a value less than the server's timeout.
    IdleTimeout time.Duration  //空闲连接等待时间，超过此时间后，空闲连接将被关闭

    // If Wait is true and the pool is at the MaxActive limit, then Get() waits
    // for a connection to be returned to the pool before returning.
    Wait bool  //当配置项为true并且MaxActive参数有限制时候，使用Get方法等待一个连接返回给连接池

    // Close connections older than this duration. If the value is zero, then
    // the pool does not close connections based on age.
    MaxConnLifetime time.Duration
    // contains filtered or unexported fields
}
```

<font color="red">**示例：**</font>

```go
package main

import (
    "github.com/garyburd/redigo/redis"
    "fmt"
)

var Pool redis.Pool
func init()  {      //init 用于初始化一些参数，先于main执行
    Pool = redis.Pool{
        MaxIdle:     16,
        MaxActive:   32,
        IdleTimeout: 120,
        Dial: func() (redis.Conn, error) {
            return redis.Dial("tcp", "10.1.210.69:6379")
        },
    }
}

func main()  {

    conn :=Pool.Get()
    res,err := conn.Do("HSET","student","name","jack")
    fmt.Println(res,err)
    res1,err := redis.String(conn.Do("HGET","student","name"))
    fmt.Printf("res:%s,error:%v",res1,err)

}
//0 <nil>
//res:jack,error:<nil>
```

## Golang redis 学习指南

### 基本操作

#### 创建客户端

通过 redis.NewClient 函数即可创建一个 redis 客户端, 这个方法接收一个 `redis.Options `对象参数, 通过这个参数, 我们可以配置 redis 相关的属性, 例如 redis 服务器地址, 数据库名, 数据库密码等.
下面是一个连接的例子:

```go
// 创建 redis 客户端

func createClient() *redis.Client {

	client := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "",
		DB:       0,
})

// 通过 cient.Ping() 来检查是否成功连接到了 redis 服务器
pong, err := client.Ping().Result()
	fmt.Println(pong, err)
	return client
}
```

#### String 操作

`redis `的`String `操作有:

> set(key, value)：给数据库中名称为key的string赋予值value
>
> get(key)：返回数据库中名称为key的string的value
>
> getset(key, value)：给名称为key的string赋予上一次的value
>
> mget(key1, key2,…, key N)：返回库中多个string的value
>
> setnx(key, value)：添加string，名称为key，值为value
>
> setex(key, time, value)：向库中添加string，设定过期时间time
>
> mset(key N, value N)：批量设置多个string的值
>
> msetnx(key N, value N)：如果所有名称为key i的string都不存在
>
> incr(key)：名称为key的string增1操作
>
> incrby(key, integer)：名称为key的string增加integer
>
> decr(key)：名称为key的string减1操作
>
> decrby(key, integer)：名称为key的string减少integer
>
> append(key, value)：名称为key的string的值附加value
>
> substr(key, start, end)：返回名称为key的string的value的子串

在 `go-redis` 中, 我们可以直接找到对应的操作方法, 直接上代码:

```go
// String 操作
func stringOperation(client *redis.Client) {
	// 第三个参数是过期时间, 如果是0, 则表示没有过期时间.
	err := client.Set("name", "xys", 0).Err()
		if err != nil {
		panic(err)
	}
	val, err := client.Get("name").Result()
	
    if err != nil {
		 panic(err)
	}

	fmt.Println("name", val)
	// 这里设置过期时间.
	err = client.Set("age", "20", 1 * time.Second).Err()
	if err != nil {
		panic(err)
	}
	client.Incr("age") // 自增
	client.Incr("age") // 自增
	client.Decr("age") // 自减

	val, err = client.Get("age").Result()
	if err != nil {
		panic(err)
	}

	fmt.Println("age", val) // age 的值为21

	// 因为 key "age" 的过期时间是一秒钟, 因此当一秒后, 此 key 会自动被删除了.
	time.Sleep(1 * time.Second)
	val, err = client.Get("age").Result()
	if err != nil {
		// 因为 key "age" 已经过期了, 因此会有一个 redis: nil 的错误.
		fmt.Printf("error: %v\n", err)
	}
	fmt.Println("age", val)
}
```

#### list 操作

redis 的 list 操作有:

> rpush(key, value)：在名称为key的list尾添加一个值为value的元素
>
> lpush(key, value)：在名称为key的list头添加一个值为value的 元素
>
> llen(key)：返回名称为key的list的长度
>
> lrange(key, start, end)：返回名称为key的list中start至end之间的元素
>
> ltrim(key, start, end)：截取名称为key的list
>
> lindex(key, index)：返回名称为key的list中index位置的元素
>
> lset(key, index, value)：给名称为key的list中index位置的元素赋值
>
> lrem(key, count, value)：删除count个key的list中值为value的元素
>
> lpop(key)：返回并删除名称为key的list中的首元素
>
> rpop(key)：返回并删除名称为key的list中的尾元素
>
> blpop(key1, key2,… key N, timeout)：lpop命令的block版本。
>
> brpop(key1, key2,… key N, timeout)：rpop的block版本。
>
> rpoplpush(srckey, dstkey)：返回并删除名称为srckey的list的尾元素，并将该元素添加到名称为dstkey的list的头部

同样地, 在 go-redis 中也可以找到对应的方法, 下面是一个简单的示例:

```go
// list 操作
func listOperation(client *redis.Client) {
	client.RPush("fruit", "apple") //在名称为 fruit 的list尾添加一个值为value的元素
	client.LPush("fruit", "banana") //在名称为 fruit 的list头添加一个值为value的 元素
	length, err := client.LLen("fruit").Result() //返回名称为 fruit 的list的长度
	if err != nil {
		panic(err)
	}
	fmt.Println("length: ", length) // 长度为2
	value, err := client.LPop("fruit").Result() //返回并删除名称为 fruit 的list中的首元素
	if err != nil {
		panic(err)
	}
	fmt.Println("fruit: ", value)
	value, err = client.RPop("fruit").Result() // 返回并删除名称为 fruit 的list中的尾元素
	if err != nil {
		panic(err)
	}
	fmt.Println("fruit: ", value)
}
```

#### set 操作

> redis 的 set 操作:
>
> sadd(key, member)：向名称为key的set中添加元素member
>
> srem(key, member) ：删除名称为key的set中的元素member
>
> spop(key) ：随机返回并删除名称为key的set中一个元素
>
> smove(srckey, dstkey, member) ：移到集合元素
>
> scard(key) ：返回名称为key的set的基数
>
> sismember(key, member) ：member是否是名称为key的set的元素
>
> sinter(key1, key2,…key N) ：求交集
>
> sinterstore(dstkey, (keys)) ：求交集并将交集保存到dstkey的集合
>
> sunion(key1, (keys)) ：求并集
>
> sunionstore(dstkey, (keys)) ：求并集并将并集保存到dstkey的集合
>
> sdiff(key1, (keys)) ：求差集
>
> sdiffstore(dstkey, (keys)) ：求差集并将差集保存到dstkey的集合
>
> smembers(key) ：返回名称为key的set的所有元素
>
> srandmember(key) ：随机返回名称为key的set的一个元素

接下来是 go-redis 的 set 操作:

```go
// set 操作
func setOperation(client *redis.Client) {
	client.SAdd("blacklist", "Obama") // 向 blacklist 中添加元素
	client.SAdd("blacklist", "Hillary") // 再次添加
	client.SAdd("blacklist", "the Elder") // 添加新元素
	client.SAdd("whitelist", "the Elder") // 向 whitelist 添加元素
	// 判断元素是否在集合中
	isMember, err := client.SIsMember("blacklist", "Bush").Result()
	if err != nil {
		panic(err)
	}
	fmt.Println("Is Bush in blacklist: ", isMember)
	// 求交集, 即既在黑名单中, 又在白名单中的元素
	names, err := client.SInter("blacklist", "whitelist").Result()
	if err != nil {
		panic(err)
	}
	// 获取到的元素是 "the Elder"
	fmt.Println("Inter result: ", names)
	// 获取指定集合的所有元素
	all, err := client.SMembers("blacklist").Result()
	if err != nil {
		panic(err)
	}
	fmt.Println("All member: ", all)
}
```

#### hash 操作

redis 的 hash 操作:

> hset(key, field, value)：向名称为key的hash中添加元素field
>
> hget(key, field)：返回名称为key的hash中field对应的value
>
> hmget(key, (fields))：返回名称为key的hash中field i对应的value
>
> hmset(key, (fields))：向名称为key的hash中添加元素field 
>
> hincrby(key, field, integer)：将名称为key的hash中field的value增加integer
>
> hexists(key, field)：名称为key的hash中是否存在键为field的域
>
> hdel(key, field)：删除名称为key的hash中键为field的域
>
> hlen(key)：返回名称为key的hash中元素个数
>
> hkeys(key)：返回名称为key的hash中所有键
>
> hvals(key)：返回名称为key的hash中所有键对应的value
>
> hgetall(key)：返回名称为key的hash中所有的键（field）及其对应的value

`go-redis` 中的 hash 操作:

```go
// hash 操作
func hashOperation(client *redis.Client) {
	client.HSet("user_xys", "name", "xys"); // 向名称为 user_xys 的 hash 中添加元素 name
	client.HSet("user_xys", "age", "18"); // 向名称为 user_xys 的 hash 中添加元素 age
	// 批量地向名称为 user_test 的 hash 中添加元素 name 和 age
	client.HMSet("user_test", map[string]string{"name": "test", "age":"20"})
	// 批量获取名为 user_test 的 hash 中的指定字段的值.
	fields, err := client.HMGet("user_test", "name", "age").Result()
	if err != nil {
		panic(err)
	}
	fmt.Println("fields in user_test: ", fields)
	// 获取名为 user_xys 的 hash 中的字段个数
	length, err := client.HLen("user_xys").Result()
	if err != nil {
		panic(err)
	}
	fmt.Println("field count in user_xys: ", length) // 字段个数为2
	// 删除名为 user_test 的 age 字段
	client.HDel("user_test", "age")
	age, err := client.HGet("user_test", "age").Result()
	if err != nil {
		fmt.Printf("Get user_test age error: %v\n", err)
	} else {
		fmt.Println("user_test age is: ", age) // 字段个数为2
	}
}
```

#### 关于连接池

redis.v4 包实现了 redis 的连接池管理, 因此我们就不需要自己手动管理 redis 的连接了. 默认情况下, redis.v4 的 redis 连接池大小是10, 不过我们可以在初始化 redis 客户端时自行设置连接池的大小, 例如:

```go
client := redis.NewClient(&redis.Options{
	Addr:     "localhost:6379",
	Password: "",
	DB:       0,
	PoolSize: 5,
})
```

通过 redis.Options 的 PoolSize 属性, 我们设置了 redis 连接池的大小为5.那么接下来我们来看一下这个设置有什么效果吧:

```go
// redis.v4 的连接池管理
func connectPool(client *redis.Client) {
	wg := sync.WaitGroup{}
	wg.Add(10)

	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for j := 0; j < 100; j++ {
				client.Set(fmt.Sprintf("name%d", j), fmt.Sprintf("xys%d", j), 0).Err()
				client.Get(fmt.Sprintf("name%d", j)).Result()
			}
			fmt.Printf("PoolStats, TotalConns: %d, FreeConns: %d\n", client.PoolStats().TotalConns, client.PoolStats().FreeConns);
		}()
	}
	wg.Wait()
}
```

上面的例子启动了10个 routine 来不断向 redis 读写数据, 然后我们通过 client.PoolStats() 获取连接池的信息. 运行这个例子, 输出如下:

```bash
PoolStats, TotalConns: 5, FreeConns: 1

PoolStats, TotalConns: 5, FreeConns: 1

PoolStats, TotalConns: 5, FreeConns: 1

PoolStats, TotalConns: 5, FreeConns: 1

PoolStats, TotalConns: 5, FreeConns: 1

PoolStats, TotalConns: 5, FreeConns: 2

PoolStats, TotalConns: 5, FreeConns: 2

PoolStats, TotalConns: 5, FreeConns: 3

PoolStats, TotalConns: 5, FreeConns: 4

PoolStats, TotalConns: 5, FreeConns: 5
```

通过输出可以看到, 此时最大的连接池数量确实是 5 了, 并且一开始时, 因为 coroutine 的数量大于5, 会造成 redis 连接不足的情况(反映在 FreeConns 上就是前几次的输出 FreeConns 一直是1), 当某个 coroutine 结束后, 会释放此 redis 连接, 因此 FreeConns 会增加.

```go
//完整示例
// author xiongyongshun
// project go_redis
// version 1.0
// created 16/10/6 03:49
package main

import (
	"fmt"
	"gopkg.in/redis.v4"
	"time"
	"sync"
)

func main() {
	client := createClient()
	defer client.Close()
	stringOperation(client)
	listOperation(client)
	setOperation(client)
	hashOperation(client)
	connectPool(client)
}

// 创建 redis 客户端
func createClient() *redis.Client {
	client := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "",
		DB:       0,
		PoolSize: 5,
	})
	pong, err := client.Ping().Result()
	fmt.Println(pong, err)
	return client
}

// String 操作
func stringOperation(client *redis.Client) {
	// 第三个参数是过期时间, 如果是0, 则表示没有过期时间.
	err := client.Set("name", "xys", 0).Err()
	if err != nil {
		panic(err)
	}

	val, err := client.Get("name").Result()
	if err != nil {
		panic(err)
	}
	fmt.Println("name", val)
	// 这里设置过期时间.
	err = client.Set("age", "20", 1 * time.Second).Err()
	if err != nil {
		panic(err)
	}
	client.Incr("age") // 自增
	client.Incr("age") // 自增
	client.Decr("age") // 自减
	val, err = client.Get("age").Result()
	if err != nil {
		panic(err)
	}
	fmt.Println("age", val) // age 的值为21
	// 因为 key "age" 的过期时间是一秒钟, 因此当一秒后, 此 key 会自动被删除了.
	time.Sleep(1 * time.Second)
	val, err = client.Get("age").Result()
	if err != nil {
		// 因为 key "age" 已经过期了, 因此会有一个 redis: nil 的错误.
		fmt.Printf("error: %v\n", err)
	}
	fmt.Println("age", val)
}

// list 操作
func listOperation(client *redis.Client) {
	client.RPush("fruit", "apple") //在名称为 fruit 的list尾添加一个值为value的元素
	client.LPush("fruit", "banana") //在名称为 fruit 的list头添加一个值为value的 元素
	length, err := client.LLen("fruit").Result() //返回名称为 fruit 的list的长度
	if err != nil {
		panic(err)
	}
	fmt.Println("length: ", length) // 长度为2
	value, err := client.LPop("fruit").Result() //返回并删除名称为 fruit 的list中的首元素
	if err != nil {
		panic(err)
	}
	fmt.Println("fruit: ", value)
	value, err = client.RPop("fruit").Result() // 返回并删除名称为 fruit 的list中的尾元素
	if err != nil {
		panic(err)
	}
	fmt.Println("fruit: ", value)
}

// set 操作
func setOperation(client *redis.Client) {
	client.SAdd("blacklist", "Obama") // 向 blacklist 中添加元素
	client.SAdd("blacklist", "Hillary") // 再次添加
	client.SAdd("blacklist", "the Elder") // 添加新元素
	client.SAdd("whitelist", "the Elder") // 向 whitelist 添加元素
	// 判断元素是否在集合中
	isMember, err := client.SIsMember("blacklist", "Bush").Result()
	if err != nil {
		panic(err)
	}
	fmt.Println("Is Bush in blacklist: ", isMember)
	// 求交集, 即既在黑名单中, 又在白名单中的元素
	names, err := client.SInter("blacklist", "whitelist").Result()

	if err != nil {
		panic(err)
	}
	// 获取到的元素是 "the Elder"
	fmt.Println("Inter result: ", names)
	// 获取指定集合的所有元素
	all, err := client.SMembers("blacklist").Result()
	if err != nil {
		panic(err)
	}
	fmt.Println("All member: ", all)

}

// hash 操作
func hashOperation(client *redis.Client) {
	client.HSet("user_xys", "name", "xys"); // 向名称为 user_xys 的 hash 中添加元素 name
	client.HSet("user_xys", "age", "18"); // 向名称为 user_xys 的 hash 中添加元素 age
	// 批量地向名称为 user_test 的 hash 中添加元素 name 和 age
	client.HMSet("user_test", map[string]string{"name": "test", "age":"20"})
	// 批量获取名为 user_test 的 hash 中的指定字段的值.
	fields, err := client.HMGet("user_test", "name", "age").Result()
	if err != nil {
		panic(err)
	}
	fmt.Println("fields in user_test: ", fields)

	// 获取名为 user_xys 的 hash 中的字段个数
	length, err := client.HLen("user_xys").Result()
	if err != nil {
		panic(err)
	}
	fmt.Println("field count in user_xys: ", length) // 字段个数为2

	// 删除名为 user_test 的 age 字段
	client.HDel("user_test", "age")
	age, err := client.HGet("user_test", "age").Result()
	if err != nil {
		fmt.Printf("Get user_test age error: %v\n", err)
	} else {
		fmt.Println("user_test age is: ", age) // 字段个数为2
	}
}

// redis.v4 的连接池管理
func connectPool(client *redis.Client) {
	wg := sync.WaitGroup{}
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for j := 0; j < 100; j++ {
				client.Set(fmt.Sprintf("name%d", j), fmt.Sprintf("xys%d", j), 0).Err()
				client.Get(fmt.Sprintf("name%d", j)).Result()
			}
			fmt.Printf("PoolStats, TotalConns: %d, FreeConns: %d\n",client.PoolStats().TotalConns,client.PoolStats().FreeConns)
		}()
	}
	wg.Wait()
}
```



