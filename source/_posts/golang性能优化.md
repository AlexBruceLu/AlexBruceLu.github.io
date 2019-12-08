title: golang性能优化

tags: golang

categories: golang

------

** {{ title }}：** <Excerpt in index | 首页摘要>

go语言养成记之golang性能优化

<!-- more -->

[TOC]

## string & profiling

### string 连接

#### fmt VS "+"

1. 如图所示：

   ![](https://github.com/AlexBruceLu/DAPP/wiki/you1.png)

   <font color="red">**比对结果：**</font>

   <font color="red">**BenchmarkFmt 1000000 1617 ns/op **</font>

   <font color="red">**BenchmarkPlus 5000000 393 ns/op  **</font>

2. 如图所示：

   ![](https://github.com/AlexBruceLu/DAPP/wiki/you2.png)

   <font color="red">**比对结果：**</font>

   <font color="red">**BenchmarkFmt 1000000 1324 ns/op **</font>

   <font color="red">**BenchmarkPlus 5000000 751 ns/op**</font>

#### strings.join VS "+"

1. 如图所示：

   ![](https://github.com/AlexBruceLu/DAPP/wiki/you3.png)

2. 如图所示：

   ![](https://github.com/AlexBruceLu/DAPP/wiki/you4.png)

   <font color="red">**比对结果：**</font>

   <font color="red">**BenchmarkPlus 500000 4659 ns/op  **</font>

   <font color="red">**BenchmarkJoin 1000000 1491 ns/op **</font>

#### strings.Join VS bytes.Buffer 

1. 如图所示：

   ![](https://github.com/AlexBruceLu/DAPP/wiki/you5.png)

2. 如图所示：

   ![](https://github.com/AlexBruceLu/DAPP/wiki/you6.png)

   <font color="red">**比对结果：**</font>

   <font color="red">**BenchmarkJoin 1000000 1505 ns/op   **</font>

   <font color="red">**BenchmarkBuffer 500000 2886 ns/op  **</font>

3. 如图所示：

   ![](https://github.com/AlexBruceLu/DAPP/wiki/you7.png)

4. 如图所示：

   ![](https://github.com/AlexBruceLu/DAPP/wiki/you8.png)

   <font color="red">**比对结果：**</font>

   <font color="red">**BenchmarkJoin 1000000 1824 ns/op   **</font>

   <font color="red">**BenchmarkBuffer 500000 2588 ns/op **</font>

5. 如图所示：

   ![](https://github.com/AlexBruceLu/DAPP/wiki/you9.png)

6. 如图所示：

   ![](https://github.com/AlexBruceLu/DAPP/wiki/you10.png)

   <font color="red">**比对结果：**</font>

   <font color="red">**BenchmarkJoin 1000000 1500 ns/op   **</font>

   <font color="red">**BenchmarkBuffer 1000000 1482 ns/op  **</font>

7. 如图所示：

   ![](https://github.com/AlexBruceLu/DAPP/wiki/you11.png)

8. 如图所示：

   ![](https://github.com/AlexBruceLu/DAPP/wiki/you12.png)

   <font color="red">**比对结果：**</font>

   <font color="red">**BenchmarkJoin 1000000 1792 ns/op   **</font>

   <font color="red">**BenchmarkBuffer 1000000 1162 ns/op  **</font>

## string & []byte 

1. 如果可以的话，尽量多用`[]byte` ，少用`string `
2. 尽可能的少在两者之间做转换
3. append([]byte,string...)
4. copy([]byte,string)

### strconv 

  1. func AppendBool(dst []byte, b bool) []byte
  2. func AppendFloat(dst []byte, f float64, fmt byte, prec int,bitSize int) []byte
  3. func AppendInt(dst []byte, i int64, base int) []byte
  4. func AppendUint(dst []byte, i uint64, base int) []byte
  5. func FormatBool(b bool) string
  6. func FormatFloat(f float64, fmt byte, prec, bitSize int) string
  7. func FormatInt(i int64, base int) string
  8. func FormatUint(i uint64, base int) string 

## slice & array 

array：值传递

slice：引用传递

![](https://github.com/AlexBruceLu/DAPP/wiki/you13.png)