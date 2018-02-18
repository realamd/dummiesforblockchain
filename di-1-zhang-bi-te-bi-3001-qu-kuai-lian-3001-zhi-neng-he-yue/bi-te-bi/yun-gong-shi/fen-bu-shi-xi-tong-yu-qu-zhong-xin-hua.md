# 分布式系统与去中心化

## 分布式系统

分布式系统的起源，早在上个世纪80年代，就已经有计算机科学家开始围绕多台计算机协同工作的方式展开研究。什么是分布式系统呢？计算机发展和普及已经接近半个世纪，人们习惯了使用一台计算机操作系统后，理解多台计算机分散工作可能有些困难，从简单的概念上来说：

* 广义概念：一组计算机协同工作，对用户展现的是一个完整的系统。
* 狭义概念：一个由网络连接的分散的物理和逻辑资源的系统。

根据这两组概念不难看出，分布式系统的重要特点是多节点，多资源以及高性能。

![](/assets/1.1.1.png)

如上图所示，在分布式系统下，Node作为一个系统的单元节点，它拥有完整的“最小系统“功能，每个节点之间都是相互连接、数据共享形成一个Cluster。因此对于用户来说，无论访问集群中哪个节点，都能够执行一套完整的系统命令；区别在于，一个集群的规模大小，以及对性能的影响。

实际上，摩尔定律的建立与失效、分布式系统的理论与产生，这两者之间的命运是息息相关的。从1965年，由Intel创始人之一的戈登.摩尔提出了著名的摩尔定律——Moore’s Law，并且整个计算机行业始终以缩小晶体管体积作为计算机性能提升的手段。因为受到物理条件限制，这种方式的性能提升始终会有“终点”，而此时人类对于计算速度的要求可能远远不够。

![](/assets/1.1.2.jpg)

2002年1月Moore在Intel开发者论坛上声明，定律预计在2017年终结，但实际上此时Intel仍在推进更小工艺技术，但成本急剧攀升。

主要的阻碍：

而有先见之明的计算机科学家，早在上世纪70年代开始就已经开始研究多处理器并行技术，到现在分布式系统的百花齐放，特别是Google的三大论文的发表，奠定了分布式系统的架构的大格局，之后该类型应用如雨后春笋般涌现。如，Google File System、MapReduce、Bigtable、Hadoop、Zookeeper、Spark、Storm、Hive、Hbase、Kafka、Redis等等，这些应用无一不是为了解决单一服务器解决不了的大量计算、海量存储、高效通信等问题。

分布式系统由于需要多台“计算机”协同工作，在逻辑处理、资源分配、信息安全以及性能上与我们常见单一计算机有很大的不同，而在这些问题中重要的就是各个节点之间的一致性，如何保证整个集群的一致性（Consistency），是确定一个系统是否是分布式系统的关键。

为此计算机科学家埃瑞克·布鲁尔在1999年提出了CAP原理（Consistency、Availability、Partition tolerance），分布式系统的一致性、可用性以及分区容错性。

* 一致性：分布式系统不同节点的备份数据，在同一时刻值是否相同。
* 可用性：分布式系统良好、高效的响应性能。
* 分区容错性：分布式系统中，若节点出现故障，是否能保证整个系统正常运行的可靠性。

![](/assets/1.1.3.png)

他认为在一个分布式系统中，系统的这三个参数只能同时满足其中两个；也就是说，如果提高一致性和分区容错性，则可用性就会降低，同理为了可用性，需要降低一致性或分区容错性其中之一。三者相辅相成，告诫架构师不要企图构建“完美”的分布式系统，要根据不同场景相互取舍，这一理论很快被业界所接受并广泛采用。

## 去中心化

去中心化作为新型词汇，与前期的云计算、大数据一样被推上了网络信息时代的舞台，在这个时代扮演者“改变世界的”角色，它被誉为区块链存在的最核心因素，但到底什么是去中心化，各界众说纷纭，存在很多异议。

抛开应用场景，从概念上来讲：

* 广义概念：在多个参与者的系统中，不存在特殊权值的个体，信息的采纳由多数个体确认并被集体认可。
* 狭义概念：在多个节点的系统中，每个节点的数据相同，且系统操作具有强一致性。

![](/assets/1.1.4.jpg)

如上图可以看出，分布式系统中的连接使用虚线，因为每次请求被执行或分配的节点有限，为了保证最大资源利用，不会使用到所有节点，例如HDFS、Spark等等。

而去中心化图中的实线代表，每一个请求（Update、Delete等）的执行会波及整个系统的所有节点，如：Zookeeper，又或着是HyperLedger和Ethereum等基于区块链的服务。

需要注意的是，在广义概念中添加了“共识”的理念。实际上，对于现阶段区块链发展，就是因为有了良好的共识机制才会被世界广泛关注。

去中心化和共识是两个不同的概念，两者结合才是真正意义上的“去中心化”，去中心化有了共识才具备了完全的可信度，否则只是狭义概念的数据共享，没人知道这些共享的数据是否真实，或者被集体认可，仅仅只是拥有这些信息；共识有了去中心化，就具备了安全性，系统很难被攻破，信息被篡改；再加上特殊的数据结构，而这一切的实现就是区块链。
