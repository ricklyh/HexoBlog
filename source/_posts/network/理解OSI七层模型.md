---
title: 理解OSI七层模型
date: 2018-11-14 12:42:21
tags: [network]
categories: network
---
## 概念
 - OSI模型，即开放式通信系统互联参考模型(Open System Interconnection,OSI/RM,Open Systems InterconnectionReference Model)，是国际标准化组织(ISO)提出的一个试图使各种计算机在世界范围内互连为网络的标准框架，简称OSI
 
 
## 七层模型
 - 0SI／RM协议是由IS0(国际标准化组织)制定的，它有三个基本的功能：提供给开发者一个必须的、通用的概念以便开发完善、可以用来解释连接不同系统的框架
 ![OSI Mode](networkMode.jpg "Optional title")
 - #### 应用层（Application Layer）
   - 提供网络与用户应用软件之间的接口服务
 - #### 表示层 (Presentation Layer)
   - 提供格式化的表示和转换数据服务，如加密和压缩
 - #### 会话层 (Session Layer)
   - 提供包括访问验证和会话管理在内的建立和维护应用之间通信的机制
 - #### 传输层（Transimission Layer）
   - 提供建立、维护和取消传输连接功能，负责可靠地传输数据(PC)
 - #### 网络层（NetWork Layer）
   - 处理网络间路由，确保数据及时传送(路由器)
 - #### 数据链路层（Data Link Layer）
   - 负责无错传输数据，确认帧、发错重传等(交换机)
 - #### 物理层（Physics Layer）
   - 提供机械、电气、功能和过程特性(网卡、网线、双绞线、同轴电缆、中继器)

 - 七层中应用层、表示层和会话层由软件控制，传输层、网络层和数据链路层由操作系统控制，物理层有物理设备控制