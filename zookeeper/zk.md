# Summary_About_Java


### 

### zk节点以及基本操作参考:	O'Reilly ZooKeeper 



### 分布式基础理论

####CAP理论

```
CAP理论的核心观点是任何软件系统都无法同时满足一致性、可用性以及分区容错性。
值得一提的是，作为一个分布式系统，分区容错性是一个必须要考虑的关键点。一个分布式系统一旦丧失了分区容错性，也就表示放弃了扩展性。因为在分布式系统中，网络故障是经常出现的，一旦出现在这种问题就会导致整个系统不可用是绝对不能容忍的。所以，大部分分布式系统都会在保证分区容错性的前提下在一致性和可用性之间做权衡。

在CAP这三个关键的性质中，同时满足CA两点的是著名的数据库中ACID、同时满足AP两点的是著名的BASE理论。
```

#### BASE理论

```
BASE理论是指，Basically Available（基本可用）、Soft-state（ 软状态/柔性事务）、Eventual Consistency（最终一致性）。是基于CAP定理演化而来，是对CAP中一致性和可用性权衡的结果。

　　核心思想：即使无法做到强一致性，但每个业务根据自身的特点，采用适当的方式来使系统达到最终一致性。

　 （1）Basically Available：基本可用，指分布式系统在出现故障的时候，允许损失部分可用性，保证核心可用。但不等价于不可用。

　 （2）Soft-state：软状态/柔性事务，即状态可以有一段时间的不同步。

　 （3）Eventual consistency：最终一致性，系统中的所有数据副本经过一定时间后，最终能够达到一致的状态，不需要实时保证系统数据的强一致性。最终一致性是弱一致性的一种特殊情况。

　　BASE是反ACID的，它完全不同于ACID模型，牺牲强一致性，获得基本可用性和柔性可靠性并要求达到最终一致性。

　　BASE理论面向的是大型高可用可扩展的分布式系统，通过牺牲强一致性来获得可用性。
```



### ZK算法



#### [Paxos算法原理以及推导](https://www.zhihu.com/question/19787937)

```
阶段一 
预提案阶段： 
	提议者Proposer：向接受者Acceptor广播预提案，附带接下来提案Proposal的proposal_id 
	接受者Acceptor：收到预提案后更新a_proposal_id = max(proposal_id,a_proposal_id)，如果预提案的proposal_id>a_proposal_id，Acceptor回复记录的接受过的proposal_id最大的提案。 

阶段二 
提案阶段： 
	提议者Proposer：等待直到收到大多数接受者对预提案的回复，从所有回复的提案组成的法定数目的提案集合K中挑选proposal_id最大的提案，以该提案的值作为本次提案的值。如果K是空集，那么可以给提案任意赋值。然后把该提案广播给接受者们，提案和预提案共享同一个proposal_id。 
	接受者Acceptor：如果收到的提案的proposal_id>= a.proposal_id，那么接受这个提案，更新a_proposal_id = max(proposal_id,a_proposal_id)，更新记录的提案。
```

#### [ZAB](http://blog.csdn.net/chen77716/article/details/7309915)

```
1. Atomic Broadcast
同一时刻存在一个Leader节点，其他节点称为“Follower”，如果是更新请求，如果客户端连接到Leader节点，则由Leader节点执行其请求；如果连接到Follower节点，则需转发请求到Leader节点执行。但对读请求，Client可以直接从Follower上读取数据，如果需要读到最新数据，则需要从Leader节点进行，Zookeeper设计的读写比例是2：1。

Leader通过一个简化版的二段提交模式向其他Follower发送请求，但与二段提交有两个明显的不同之处：
	因为只有一个Leader，Leader提交到Follower的请求一定会被接受（没有其他Leader干扰）
	不需要所有的Follower都响应成功，只要一个多数派即可
通俗地说，如果有2f+1个节点，允许f个节点失败。因为任何两个多数派必有一个交集，当Leader切换时，通过这些交集节点可以获得当前系统的最新状态。如果没有一个多数派存在（存活节点数小于f+1）则，算法过程结束。但有一个特例：
如果有A、B、C三个节点，A是Leader，如果B Crash，则A、C能正常工作，因为A是Leader，A、C还构成多数派；如果A Crash则无法继续工作，因为Leader选举的多数派无法构成。

2.	Leader Election
Leader选举主要是依赖Paxos算法
```

