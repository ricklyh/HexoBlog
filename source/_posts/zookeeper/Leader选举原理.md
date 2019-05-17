---
title: Leader选举原理
date: 2019-04-03 10:22:04
tags: [zookeeper]
categories: zookeeper
---
## 前言
   - Leader崩溃或者Leader失去大多数的Follower，这时候zk进入恢复模式，恢复模式需要重新选举出一个新的Leader，让所有的Server都恢复到一个正确的状态。
   Zookeeper中Leader的选举采用了三种算法：
     - LeaderElection
     - FastLeaderElection
     - AuthFastLeaderElection
   - 并且在配置文件中是可配置的，对应的配置项为electionAlg
   
   - 成为Leader的必要条件： Leader要具有最高的zxid；当集群的规模是n时，集群中大多数的机器（至少n/2+1）得到响应并follow选出的Leader
   
   - 心跳机制：Leader与Follower利用PING来感知对方的是否存活，当Leader无法相应PING时，将重新发起Leader选举
   
   - zxid：zookeeper transaction id, 每个改变Zookeeper状态的操作都会形成一个对应的zxid，并记录到transaction log中。 这个值越大，表示更新越新
   
   - electionEpoch/logicalclock：逻辑时钟，用来判断是否为同一次选举。每调用一次选举函数，logicalclock自增1，并且在选举过程中如果遇到election比当前logicalclock大的值，就更新本地logicalclock的值

   - peerEpoch: 表示节点的Epoch
## zookeeper server状态说明
   - LOOKING:寻找Leader
   - LEADING：Leader状态，对应的节点为Leader。
   - FOLLOWING：Follower状态，对应的节点为Follower。
   - OBSERVING：Observer状态，对应节点为Observer，该节点不参与Leader选举
    
## LeaderElection选举算法
   - LeaderElection是Fast Paxos最简单的一种实现，每个Server启动以后都询问其它的Server它要投票给谁，收到所有Server回复以后，就计算出zxid最大的哪个Server，并将这个Server相关信息设置成下一次要投票的Server。
     该算法于Zookeeper 3.4以后的版本废弃
     
   - #### 选举流程：
     - 1 选举线程首先向所有的server发起一次询问（包括自己）
     - 2 选举线程收到回复，验证是否是自己发起询问（验证zxid是否一致），然后获取对方的server Id(MyId),并存储到当前询问对象列表中，最后获取对方提议的leader相关信息（id,zxid）,并将这些信息存储到当次选举的投票记录表中
     - 3 收到所有的server回复后，就计算出zxid最大的那个server,并将这个server的相关信息设置成下一次要投票的server
     - 4 选举线程将当前zxid最大的server设置为当前server要推举的leader，如果此时（zxid最大的server）获取多数server投票,就设置当前推荐的leader为获胜的server
         将根据获胜的server相关信息设置自己的状态，否则继续这个过程，直到leader被选举出来
   
   - #### 流程图展示      
   ![LeaderElectoin](LeaderElectoin.png "Optional title")
   - 通过流程分析我们可以得出：要使Leader获得多数Server的支持，则Server总数必须是奇数2n+1，且存活的Server的数目不得少于n+1
   
   - #### Leader选举异常情况
     - 1 选举过程中，Server的退出，只要保证n/2+1个Server存活就没有任何问题，如果少于n/2+1个Server 存活就没办法选出Leader
     
     
  
   