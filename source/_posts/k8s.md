title: Kubernetes（K8s） 

tags: k8s

categories: docker

------

** {{ title }}：** <Excerpt in index | 首页摘要>

Kubernetes 的简介和基本操作

<!-- more -->

[TOC]

## 为什么叫k8s

Kubernetes（K8s）是Google在2014年发布的一个开源项目 。而且`k`和`s`之间有八个字母间隔，所以叫做k8s。

## k8s 的基本概念

- **Cluster（集群 )**

  > Cluster是计算、存储和网络资源的集合，Kubernetes利用这些资源运行各种基于容器的应用。 

- **Master（控制主节点）**

  > Master是Cluster的大脑，它的主要职责是调度，即决定将应用放在哪里运行。Master运行Linux操作系统，可以是物理机或者虚拟机。为了实现高可用，可以运行多个Master。调度应用程序、维护应用程序的所需状态、扩展应用程序和滚动更新都是master的主要工作。 

- **Node（节点）** 

  > Node的职责是运行容器应用。Node由Master管理，Node负责监控并汇报容器的状态，同时根据Master的要求管理容器的生命周期。
  > Node是 Kubernetes 集群中的工作机器，可以是物理机或虚拟机。每个工作节点都有一个 kubelet，它是管理节点并与 Kubernetes Master 节点进行通信的代理。节点上还应具有处理容器操作的容器运行时，例如 Docker。
  > 一个 Kubernetes 工作集群至少有三个节点。 Master 管理集群，而 Node（节点）用于托管正在运行的应用程序。
  > 当你在 Kubernetes 上部署应用程序时，你可以告诉 master 启动应用程序容器。Master 调度容器在集群的节点上运行。 节点使用 Master 公开的 Kubernetes API 与 Master 通信。用户也可以直接使用 Kubernetes 的 API 与集群交互 。

![](https://github.com/AlexBruceLu/DAPP/wiki/k8s.png)

- **Pod（资源对象） **

  > Pod是Kubernetes的最小工作单元。每个Pod包含一个或多个容器。Pod中的容器会作为一个整体被Master调度到一个Node上运行。
  > Kubernetes引入Pod主要基于下面两个目的： 
  >
  > （1）可管理性。 有些容器天生就是需要紧密联系，一起工作。Pod提供了比容器更高层次的抽象，将它们封装到一个部署单元中。Kubernetes以Pod为最小单位进行调度、扩展、共享资源、管理生命周期。 
  >
  > （2）通信和资源共享。 Pod中的所有容器使用同一个网络namespace，即相同的IP地址和Port空间。它们可以直接用localhost通信。同样的，这些容器可以共享存储，当Kubernetes挂载volume到
  > Pod，本质上是将volume挂载到 Pod中的每一个容器。Pods有两种使用方式：
  > **（1）运行单一容器。**
  > one-container-per-Pod是Kubernetes最常见的模型，这种情况下，只是将单个容器简单封装成Pod。即便是只有一个容器，Kubernetes管理的也是Pod而不是直接管理容器。
  > **（2）运行多个容器。**
  > 问题在于：哪些容器应该放到一个Pod中？ 答案是：这些容器联系必须非常紧密，而且需要直接共享资源。举个例子，如图 所示，这个Pod包含两个容器：一个是File Puller(文件拉取器)，一个是Web Server。 