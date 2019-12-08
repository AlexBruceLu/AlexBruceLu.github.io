title: kafka  

tags: Windows

categories: others

------

** {{ title }}：** <Excerpt in index | 首页摘要>

windows 安装及配置kafka  

<!-- more -->

[TOC]

## kafka安装与设置

### 1. Kafka是一种高吞吐量的分布式发布订阅消息系统。详细介绍可查阅官网：kafka官网



### 2.环境搭建

#### 2.1 安装JDK

​    下载地址：[jre下载](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

   默认安装文件夹，C:\Program Files\Java\jre1.8.0_202

#### 2.1.1 设置环境变量

新建系统变量`JAVA_HOME`：C:\Program Files\Java\jre1.8.0_202;

在`path`里面添加：`%JAVA_HOME%\bin`；

### 2.2 安装Zookeeper

​     下载地址：https://zookeeper.apache.org/releases.html

    下载后，解压放在目录D:\kafka（本文所用的目录）下，关于zookeeper以及kafka的目录，路径中最好不要出现空格，比如D:\Program Files，尽量别用，运行脚本时会有问题。
    
    1. 进入zookeeper的相关设置所在的文件目录，例如本文的：     D:\bigdata\zookeeper-3.4.10\conf
    
    2. 将"zoo_sample.cfg"重命名为"zoo.cfg"
    
    3. 打开zoo.cfg(至于使用什么编辑器，根据自己喜好选即可)，找到并编辑：
    
    dataDir=/tmp/zookeeper  to  D:/kafka/zookeeper-3.4.10/data或 D:\\kafka\\zookeeper-3.4.10\\data(路径仅为示例，具体可根据需要配置)
    
    这里注意，路径要么是"/"分割，要么是转义字符"\\"，这样会生成正确的路径(层级，子目录)。
    
    4. 与配置jre类似，在系统环境变量中添加：
    
        a.系统变量中添加ZOOKEEPER_HOME=D:\kafka\zookeeper-3.4.10
    
        b.编辑系统变量中的path变量，增加%ZOOKEEPER_HOME%\bin
    
    5. 在zoo.cfg文件中修改默认的Zookeeper端口(默认端口2181)
    6. 打开cmd窗口，输入zkserver，运行Zookeeper，运行结果如下：
![img](https://img-blog.csdn.net/20170812132732633?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDI4Mzg5NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

恭喜，Zookeeper已经安装完成，已在2181端口运行。

### 2.3 安装kafka

下载地址：http://kafka.apache.org/downloads

    要下载Binary downloads这个类型，不要下载源文件，这种方便使用。下载后，解压放在D:\bigdata目录下。

①进入kafka配置文件所在目录，D:\bigdata\kafka_2.11-0.9.0.1\config

②编辑文件"server.properties"，找到并编辑：

 log.dirs=/tmp/kafka-logs  to  log.dirs=D:/bigdata/kafka_2.11-0.9.0.1/kafka-logs 或者 D:\\bigdata\\kafka_2.11-0.9.0.1\\kafka-logs

同样注意：路径要么是"/"分割，要么是转义字符"\\"，这样会生成正确的路径(层级，子目录)。错误路径情况可自行尝试，文件夹名为这种形式：bigdatakafka_2.11-0.9.0.1kafka-logs

③在server.properties文件中，zookeeper.connect=localhost:2181代表kafka所连接的zookeeper所在的服务器IP以及端口，可根据需要更改。本文在同一台机器上使用，故不用修改。

④kafka会按照默认配置，在9092端口上运行，并连接zookeeper的默认端口2181。

### 2.4 运行kafka

提示：请确保启动kafka服务器前，Zookeeper实例已经在运行，因为kafka的运行是需要zookeeper这种分布式应用程序协调服务。

①进入kafka安装目录D:\bigdata\kafka_2.11-0.9.0.1

②按下shift+鼠标右键，选择"在此处打开命令窗口"，打开命令行。

③在命令行中输入：.\bin\windows\kafka-server-start.bat .\config\server.properties   回车。

④正确运行的情况为：

![img](https://img-blog.csdn.net/20170812132906662?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDI4Mzg5NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

到目前为止，zookeeper以及kafka都已正确运行。保持运行状态，不要关闭。

<font color="red">重要（操作日志的处理）：</font>


kafka启动后，如果你去查看kafka所在的根目录，或者是kafka本身的目录，会发现已经默认生成一堆操作日志(这样看起来真心很乱)：

而且会不断生成不同时间戳的操作日志。刚开始不知所措，一番研究后，看了启动的脚本内容，发现启动的时候是会默认使用到这个log4j.properties文件中的配置，而在zoo.cfg是不会看到本身的启动会调用到这个，还以为只有那一个日志路径：

![img](https://img-blog.csdn.net/20170812133057808?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDI4Mzg5NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

在这里配置一下就可以了，找到config下的log4j.properties：

![img](https://img-blog.csdn.net/20170812133131159?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDI4Mzg5NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

将路径更改下即可，这样就可以归档在一个文件夹下边了，路径根据自己喜好定义：

![img](https://img-blog.csdn.net/20170812133222818?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDI4Mzg5NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

另外如何消除不断生成日志的问题，就是同一天的不同时间会不停生成。

修改这里，还是在log4j.properties中：

![img](https://img-blog.csdn.net/20170812133300278?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDI4Mzg5NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

本身都为trace，字面理解为会生成一堆跟踪日志，将其改为INFO即可。

## 3. 创建主题

①创建主题，命名为"test0811"，replicationfactor=1(因为只有一个kafka服务器在运行)。可根据集群中kafka服务器个数来修改replicationfactor的数量，以便提高系统容错性等。

②在D:\kafka\kafka_2.11-0.9.0.1\bin\windows目录下打开新的命令行

③输入命令：

kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test0811回车。 

![img](https://img-blog.csdn.net/20170812133330959?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDI4Mzg5NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

<font color="red">该窗口可以关闭。</font>

### 3.1 创建生产者、消费者

①在D:\bigdata\kafka_2.11-0.9.0.1\bin\windows目录下打开新的命令行。

②输入命令，启动producer：

kafka-console-producer.bat --broker-list localhost:9092 --topic test0811 

**该窗口不要关闭。**

③同样在该目录下打开新的命令行。

④输入命令，启动consumer：

kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test0811 --from-beginning

**<font color="red">参数 --from-beginning 是表示消费者从生产者消息源头开始接收，去掉之后就只是接收新消息</font>**

![img](https://img-blog.csdn.net/20170812133419255?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDI4Mzg5NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

现在生产者、消费者均已创建完成。

⑤在producer命令行窗口中任意输入内容，回车  在consumer命令行窗口中即可看到相应的内容。

![img](https://img-blog.csdn.net/20170812133447425?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDI4Mzg5NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

