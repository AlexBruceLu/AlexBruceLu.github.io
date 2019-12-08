title: Go Test

tags: golang

categories: golang

------

** {{ title }}：** <Excerpt in index | 首页摘要>

Go Test相关介绍

<!-- more -->

## Go Test

### go test 和 go test -v

### t.SkipNow()的作用

### go test的顺序执行

> go test 明确指出并不会按照顺序来执行，需要顺序执行某些`test`时使用`t.Run()`

```go
func testPrint1(t *testing.T) {
	fmt.Println("print 1...")
}
func testPrint2(t *testing.T) {
	fmt.Println("print 2...")
}
func TestAll(t *testing.T) {
	t.Run("testPrint1",testPrint1)
	t.Run("testPrint2",testPrint2)
}
```

### TestMain 用来初始化

```go
func TestMain(m *testing.M) {
	fmt.Println("This is a testmain")
	m.Run()
}
```

使用TestMain来初始化test，并使用m.Run()来调用其他tests，可以完成需要初始化的一些testing，如：数据库连接，打开文件，REST服务登录等。

如果整个TestMain里面没有执行m.Run()，整个test文件中的其他tests将不会被执行，只有TestMain会被执行。

### benchmark的书写

> go test -bench=.    // 一般使用`.`就可以了

```go
func BenchmarkAll(b *testing.B) { // 测试函数性能,函数首字母大写
	for i := 0; i < b.N; i++ {
		PrintHello()
	}
}
```

测试性能，每次平均时间，执行次数， b.N 表示函数稳定执行的最大次数，若函数执行时间并不稳定，测试函数就会一直执行下去，永远跑不完，没有输出结果。

```go
func aaa(i int) {
	if i > 0 {
		i--
	}
}
func BenchmarkAll(b *testing.B) { // 测试函数性能
	for i := 0; i < b.N; i++ {
		aaa(i)
	}
}
```

