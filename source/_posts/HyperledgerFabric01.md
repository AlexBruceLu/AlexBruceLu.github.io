title: HyperLedger Fabric

tags: Fabric

categories: hyperledger

------

** {{ title }}：** <Excerpt in index | 首页摘要>

HyperLedger基础概念简介，以及核心模块的介绍

<!-- more -->

[TOC]

# Fabric基本概念 

## 1. 逻辑架构 

![](https://github.com/AlexBruceLu/DAPP/wiki/fen.png)



- **成员管理（MemberShip）**
  - 会员注册
    - 注册成功一个账号得到的不是用户名密码
    - 使用证书作用身份认证的标志
  - 身份保护
  - 交易审计
  - 内容保密
    - 可以多条区块链, 通过通道来区分的
- **账本管理**
  - 区块链
    - 保存所有的交易记录
  - 世界状态
    - 数据的最新状态
    - 数据存储在当前节点的数据库中
      - 自带的默认数据库: levelDB, 也可以使用couchdb
      - 以键值对的方式进行存储 的

- **交易管理**
  - 部署交易
    - 部署的是链码, 就是给节点安装链码 - chaincode
  - 调用交易
    - invoke
- **智能合约**
  - 一段代码， 处理网络成员所同意的业务逻辑
  - 真正实现了链码和账本的分离（逻辑和数据分离）