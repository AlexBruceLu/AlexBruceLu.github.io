title: protobuf

tags: 微服务

categories: microServices

---

** {{ title }}：** <Excerpt in index | 首页摘要>

微服务之protobuf

<!-- more -->
# protobuf

## 1. protobuf 简介

> <font color="red">Google Protocol Buffer(简称 Protobuf)</font>是一种轻便高效的结构化数据格式，无关平台、无关语言、可扩展，用于通讯协议和数据存储等领域。

### 1.1 protobuf 的优点

> protobuf 有如XML，不过它更小、更快、规范、更简单。可以自定义自己的数据结构，然后使用代码生成器的代码来读写这个数据结构。甚至可以在无需重新部署程序的情况下更新数据结构。只需使用protobuf 对数据结构进行一次描述，即可利用各种不同的语言或从各种不同数据流中对结构化的数据轻松读写。
>
> protobuf 向后兼容性好，不必破坏已经部署的“老”数据格式的程序就可以对数据结构进行升级。
>
> protobuf 语义更清晰，无需类似XML 解析器的东西（因为protobuf 编译器会将.proto 文件编译生成对应的数据，访问类似于对protobuf 数据进行序列化、反序列化操作）。
>
> protobuf 的编程模式比较友好，无需学习复杂的文档对象模型，简单易学。

### 1.2 protobuf 的缺点

> protobuf 与 XML 相比的不足之处有以下几点：
>
> - 功能简单，无法用来表示复杂的概念
> - XML 已经成为多种行业的标准编写工具，protobuf 只是Google 内部使用的工具，在通用性上差很多
> - 文本并不适合描述数据结构，所以 protobuf 不适合用来对基于文本的标记文档（如HTML）建模
> - 由于XML具有某种程度的字自解释性，它可以被人直接读取编辑，protobuf不行，它以二进制的方式存储，除非有 .proto 定义，否则无法直接读出 protobuf 的任何内容

## 2. 常见数据交互的格式比较

1. json：一般的web项目中，最流行的主要还是json，因为浏览器对于json数据支持非常好，有很多内建函数的支持。

2. XML：在 WebService 中应用最为广泛，但相比json 更加冗余，因为需要成对的闭合标签，而json 使用了 键值对的方式，不仅压缩了一定的数据空间，而且具有一定的可读性

3. protobuf：是谷歌开源的一种数据格式，适合高性能，对响应速度有要求的传输数据场景。protobuf 是二进制数据，需要编码和解码，数据本身并不具有可读性，只有对其进行反序列化之后才能得到可读的数据

   > 对于其他数据格式，protobuf的优势
   >
   > 1. 序列化之后体积相对于json 和 XML 很小，适合网络传输
   > 2. 支持跨平台多语言
   > 3. 消息格式升级兼容性好
   > 4. 序列化、反序列化的速度快，快于json的处理速度

## 3. protobuf 的安装

### 3.1 安装 protobuf

```shell
# 下载 protobuf
$ git clone https://github.com/protocolbuffers/protobuf.git

# 下载依赖库
$ sudo apt-get install autoconf automake libtool curl make g++ unzip libffi-dev -y

# 安装
$ cd protobuf/
$ ./autogen.sh
$ ./configure
$ make
$ sudo make install
$ sudo ldconfig # 刷新共享库（重要）
$ protoc -h
# 出现帮助列表说明protobuf 安装成功
```

### 3.2 获取 proto 包

```shell
$ go get -v -u github.com/golang/protobuf/protoc
```

### 3.3 安装 protoc-gen-go 插件

```shell
# 安装
$ go get -v -u github.com/golang/protobuf/protoc-gen-go
# 编译
$ cd $GOPATH/src/github.com/golang/protobuf/protoc-gen-go
$ go build
# 将生成的 protoc-gen-go 可执行文件放在/bin 目录下
$ sudo cp protoc-gen-go /bin/
```

## 4. protobuf 的语法

### 4.1 定义一个消息

要想使用 protobuf 必须先得定义 proto 文件，所以先得 熟悉消息定义的相关语法

```protobuf
syntax = "proto3";

message BruceRequest {
    string name = 1;
    int32 height = 2;
    repeated int32 weight = 3；
}
```

- **BruceRequest** 消息格式有三个字段，在消息中承载的数据分别对应于每一个字段，其中每一个字段都有一个名字和一种类型

- 文件第一行是指定了使用`proto3` 语法，如果没有指定，默认`proto2` 语法。指定语法行，必须是非空、非注释的第一行
- ` repeated` 关键字表示重复的，在go语言中用切片进行代表
- 在消息定义中每个字段都有一个唯一的标识符

### 4.3 定义一个消息类型

在一个 .proto 文件中可以定义多个消息类型。在定义多个相关的消息的时候，这一点特别有用——例如，如果想定义与 SearchResponse 消息类型对应的回复消息格式的话，你可以将它添加到相同的 .proto 文件中

```protobuf
syntax = "proto3";

message BruceRequest {
    string name = 1;
    int32 height = 2;
    repeated int32 weight = 3；
}

message AlexRequest {
    ...
}
```

### 4.4 添加注释 

如一般的编程语言一直使用 '//'

```protobuf
syntax = "proto3";

message BruceRequest {
    string name = 1; // 姓名
    int32 height = 2; // 身高
    repeated int32 weight = 3；// 体重
}

message AlexRequest {
    ...
}
```

### 4.5 .proto 文件生成文件

当用protocol buffer编译器来运行.proto文件时，编译器将生成所选择语言的代码，这些代码可以操作在.proto文件中定义的消息类型，包括获取、设置字段值，将消息序列化到一个输出流中，以及从一个输入流中解析消息。

> 对C++来说，编译器会为每个.proto文件生成一个.h文件和一个.cc文件，.proto文件中的每一个消息有一个对应的类。 对Python来说，有点不太一样——Python编译器为.proto文件中的每个消息类型生成一个含有静态描述符的模块，，该模块与一个元类（metaclass）在运行时（runtime）被用来创建所需的Python数据访问类。 对go来说，编译器会为每个消息类型生成了一个.pd.go文件。

### 4.6 标准数据类型

一个标量消息字段可以含有一个如下的类型:

> 该表格展示了定义于.proto文件中的类型，以及与之对应的、在自动生成的访问类中定义的类型

|  .proto  |                            Notes                             |  C++   |   Python    |   Go    |
| :------: | :----------------------------------------------------------: | :----: | :---------: | :-----: |
|  double  |                                                              | double |    float    | float64 |
|  float   |                                                              | float  |    float    | float32 |
|  int32   | 使用变长编码，对于负值的效率很低,如果你的域有可能有负值，请使用sint64替代 | int32  |     int     |  int32  |
|  uint32  |                         使用变长编码                         | uint32 |  int/long   | uint32  |
|  uint64  |                         使用变长编码                         | uint64 |  int/long   | uint64  |
|  sint32  |        使用变长编码，这些编码在负值时比int32高效的多         | int32  |     int     |  int32  |
|  sint64  |    使用变长编码，有符号的整型值。编码时比通常的int64高效     | int64  |  int/long   |  int64  |
| fixed32  | 总是4个字节，如果数值总是比总是比228大的话，这个类型会比uint32高效 | uint32 |     int     | uint32  |
| fixed64  | 总是8个字节，如果数值总是比总是比256大的话，这个类型会比uint64高效 | uint64 |  int/long   | uint64  |
| sfixed32 |                         总是4个字节                          | int32  |     int     |  int32  |
| sfixed32 |                         总是4个字节                          | int32  |     int     |  int32  |
| sfixed64 |                         总是8个字节                          | int64  |  int/long   |  int64  |
|   bool   |                                                              |  bool  |    bool     |  bool   |
|  string  |      一个字符串必须是UTF-8编码或者7-bit ASCII编码的文本      | string | str/unicode | string  |
|  bytes   |                  可能包含任意顺序的字节数据                  | string |     str     | []byte  |

#### 4.6.1 默认值

当一个消息被解析的时候，如果被编码的信息不包含一个特定的元素，被解析的对象锁对应的域被设置位一个默认值，对于不同类型指定如下： 

- 对于strings，默认是一个空string 
- 对于bytes，默认是一个空的bytes 
- 对于bool，默认是false 
- 对于数值类型，默认是0

### 4.7 使用其他消息类型

可以将其他消息类型用作字段类型。

> 例如，假设在每一个PersonInfo消息中包含Person消息，此时可以在相同的 .proto 文件中定义一个Result消息类型，然后在PersonInfo消息中指定一个Person类型的字段

```protobuf
syntax = "proto3";

message BruceRequest {
    string name = 1;
    int32 height = 2;
    repeated int32 weight = 3；
}
```

### 4.8 使用proto2消息类型

在你的proto3消息中导入proto2的消息类型也是可以的，反之亦然，然后proto2枚举不可以直接在proto3的标识符中使用（如果仅仅在proto2消息中使用是可以的）。

#### 4.8.1 嵌套类型

你可以在其他消息类型中定义、使用消息类型，在下面的例子中，Person消息就定义在PersonInfo消息内，如：

```protobuf
message PersonInfo {
	message Person {
		string name = 1;
		int32 shengao = 2;
		repeated int32 tizhong = 3;
	} 
	repeated Person info = 1;
}
```

如果你想在它的父消息类型的外部重用这个消息类型，你需要以PersonInfo.Person的形式使用它，如：

```protobuf
message PersonMessage {
	PersonInfo.Person info = 1;
}
```

当然，你也可以将消息嵌套任意多层，如：

```protobuf
message Grandpa { // Level 0
	message Father { // Level 1
		message son { // Level 2
			string name = 1;
			int32 age = 2;
		}
	} 

	message Uncle { // Level 1
		message Son { // Level 2
			string name = 1;
			int32 age = 2;
		}
	}
}
```

#### 4.8.2 定义服务(Service)

果想要将消息类型用在RPC(远程方法调用)系统中，可以在.proto文件中定义一个RPC服务接口，protocol buffer编译器将会根据所选择的不同语言生成服务接口代码及存根。如，想要定义一个RPC服务并具有一个方法，该方法能够接收 SearchRequest并返回一个SearchResponse，此时可以在.proto文件中进行如下定义：

```protobuf
service SearchService {
	//rpc 服务的函数名 （传入参数）返回（返回参数）
	rpc Search (SearchRequest) returns (SearchResponse);
}
```

最直观的使用protocol buffer的RPC系统是gRPC一个由谷歌开发的语言和平台中的开源的RPC系统，gRPC在使用protocl buffer时非常有效，如果使用特殊的protocol buffer插件可以直接为您从.proto文件中产生相关的RPC代码。

如果你不想使用gRPC，也可以使用protocol buffer用于自己的RPC实现，你可以从proto2语言指南中找到更多信息

#### 4.8.3 生成访问类

可以通过定义好的.proto文件来生成Java,Python,C++, Ruby, JavaNano, Objective-C,或者C# 代码，需要基于.proto文件运行protocol buffer编译器protoc。如果你没有安装编译器，下载安装包并遵照README安装。对于Go,你还需要安装一个特殊的代码生成器件。你可以通过GitHub上的protobuf库找到安装过程
通过如下方式调用protocol编译器:

```shell
$ protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR path/to/file.proto
```

IMPORT_PATH声明了一个.proto文件所在的解析import具体目录。如果忽略该值，则使用当前目录。如果有多个目录则可以多次调用--proto_path，它们将会顺序的被访问并执行导入。-I=IMPORT_PATH是--proto_path的简化形式。

**当然也可以提供一个或多个输出路径：**

 --cpp_out 在目标目录DST_DIR中产生C++代码，可以在C++代码生成参考中查看更多。 

--python_out 在目标目录 DST_DIR 中产生Python代码，可以在Python代码生成参考中查看更多。

--go_out 在目标目录 DST_DIR 中产生Go代码，可以在GO代码生成参考中查看更多。

> 作为一个方便的拓展，如果DST_DIR以.zip或者.jar结尾，编译器会将输出写到一个ZIP格式文件或者符合JAR标准的.jar文件中。注意如果输出已经存在则会被覆盖，编译器还没有智能到可以追加文件。 - 你必须提议一个或多个.proto文件作为输入，多
> 个.proto文件可以只指定一次。虽然文件路径是相对于当前目录的，每个文件必须位于其IMPORT_PATH下，以便每个文件可以确定其规范的名称。

#### 4.8.4 测试

protobuf的使用方法是将数据结构写入到 .proto文件中，使用 protoc编译器编译(间接使用了插件）得到一个新的go包，里面包含 go中可以使用的数据结构和一些辅助方法。

- 编写 test.proto文件

  - $GOPATH/src/创建 myproto文件夹

    ```shell
    $ cd $GOPATH/src/
    $ mkdir myproto
    ```

  - myproto文件夹中创建 test.proto文件 (protobuf协议文件)

    ```sh
    $ vim test.proto
    ```

  - 文件内容

    ```protobuf
    syntax = "proto3";
    package myproto;
    
    message Test {
    	string name = 1;
    	int32 stature = 2 ;
    	repeated int64 weight = 3;
    	string motto = 4;
    }
    ```

- 编译 :执行

  ```sh
  $ protoc --go_out=./ *.proto 
  # 生成 test.pb.go文件
  ```

- 使用 protobuf 做数据格式转换

  ```go
  package main
  
  import (
  	"fmt"
  	"github.com/golang/protobuf/proto"
  	"myproto"
  ) 
  
  func main() {
  	test := &myproto.Test{
  		Name : "panda",
  		Stature : 180,
  		Weight : []int64{120,125,198,180,150,180},
  		Motto : "天行健，地势坤",
  	} 
      //将Struct test 转换成 protobuf
  	data,err:= proto.Marshal(test)
  	if err!=nil{
  		fmt.Println("转码失败",err)
  	} 
      //得到一个新的Test结构体 newTest
  	newtest:= &myproto.Test{}
      //将data转换为test结构体
  	err = proto.Unmarshal(data,newtest)
  	if err!=nil {
  		fmt.Println("转码失败",err)
  	} 
      fmt.Println(newtest.String())
  	//得到name字段
  	fmt.Println("newtest->name",newtest.GetName())
  	fmt.Println("newtest->Stature",newtest.GetStature())
  	fmt.Println("newtest->Weight",newtest.GetWeight())
  	fmt.Println("newtest->Motto",newtest.GetMotto())
  }
  ```






