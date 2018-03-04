# Paxos算法

Paxos算法是解决分布式系统共识问题的重要算法，同样也衍生了大多数区块链共识算法，由Leslie Lamport在1989年提出这个名字，其实它的出生要早几年，对于它的历史，在这里不做详述。而Paxos算法对于现代互联网的发展作用，借用Google在2006年Chubby论文中的一句话：“事实上，到目前为止我们所遇到用于解决异步共识问题的算法，其核心都是Paxos”。

为了很好的讲述Paxos算法的内容，我们从简单结构模型开始——客户端/服务器（C/S）。

![](/assets/1.1.5.png)

上图的功能是，Client给Server发送请求，Server接收到信息后，执行并返回结果。

但如何应对C/S之间的消息传递过程中出现消息丢失呢？如何避免重复发送同一个消息呢？

实际上如果我们在传输过程中添加确认机制（Acknowledgements），也就是说，每次Client在发送请求后，若Server接收到请求则会发送一个AckMessage返回给Client，如果超时还未接收到该消息，则Client补发请求；同时为了避免Server重复执行同一个消息，需要Client在发送的消息中添加一个MessageID，用于识别是否是重复消息。

> Acknowledgements：
>
> * Client向Server发送一条带MessageID的命令。
> * Server接收到命令，会返回一条确认消息AckMessage。
> * 如果Client在一段时间内没有收到确认消息，则重新发送该命令。

该算法的应用非常广泛，常见的有TCP协议，应用服务有Storm确认消息是否发送的Ack函数等等。

但是这又出现了另一个问题，如果有多个Client呢？

![](/assets/1.1.6.png)

这样的结构带来一个新的问题，如果出现消息延迟，那Server端接收到的命令顺序也会不一样。

为了解决这个问题，Server端提供了一个锁机制（Locking）。每一个Client只有在拿到Server端的锁（Lock），才能够发送请求消息，没有拿到锁的Client需要等待其他Client释放锁，并重新抢占锁。这就是两段式协议（Two Phase Protocol）的雏形，通过两段才能发送/执行消息，保证了在同一时间只能有一个Client可以操作Server。避免了消息延迟、多客户端抢占Server带来的问题。

> Locking：
>
> * Client向Server请求Lock。（第一段）
> * 如果CLient获得Lock，则发送消息给Server。（第二段）
> * 如果Client没有获得Lock，则等带其他Client释放Lock，并重新获取Lock。

实际上，这个问题也是一致性问题的解决，在经典数据库的系统中，两段提交协议（2PC）就是这种机制的实现。

但是，我们再次扩展模型，增加Server的个数，一个Client，需要发送消息到不同的Server时，这个两段式协议就出现了很大的问题。

![](/assets/1.1.7.png)

每个Server都有各自的Lock，一个Client在提交时，需要获取所有的Lock才能发送消息。

如果Client宕机还未完全释放所有的Lock，如果Client只能获取到部分Lock，会不会出现死锁？

在这样的情况下如何保证各个Server的消息一致性呢？或者又如何保证数据的同步呢？

伴随着这些问题，Paxos算法中提出了一个替代Lock的概念：票据（Ticket），其特性如下。

> Ticket：
>
> * 由Server端发出最新的t，无论之前是否已发过。
> * t可以过期，过期作废。

正常情况下，一个Client会先获取每个Server的最新的Ticket——t，并在请求的消息中携带该t，消息结构是最新t和命令c（t，c）。如果消息中的t不是该Server最新的t，Client会重新获取t。

所以无论Client是否宕机都不会影响其他Client的操作，因为每个Server只接收携带最新t的消息。

同时，为了避免Server端发生宕机，我们认为当一个Client收到超过一半以上的Server确认的最新t，就可以向所有Server发送确认执行的命令消息。

> 简化Paxos算法：
>
> T，C为Client端的ticket和command参数；Ticket由每个Client维护，每次增加，由Server端验证、保存。
>
> Server端最大ticket T.max，存储的命令携带ticket T.store，命令 C.store。
>
> 初始化，
>
> | Client | Server |
> | :--- | :--- |
> | T=0，C=c0 | T.max=t，T.store=t，C.store=c1 |
>
> 请求 ，
>
> | Client | Server |
> | :--- | :--- |
> | 向所有Server发送请求票据，T=T+1，并发送给所有Server。 |  |
> |  | if \(T&gt;T.max\) {                                                                                                   T.max = T;                                                                                  return prepare\(T.store, C.store\);                                           } |
> | if\(获得半数以上Server回复prepare\){                                                                     选择最大的T.store的\(T.store，C.store\);                                           if\(T.store&gt;0\){                                                                                    C=C.store;                                                                                 }                                                                                 向回复的Server发送消息propose\(T,C\);                   } |  |
> |  | 获得propose请求。                                                                  if\(T=T.max\){                                                                                                    C.store=C;                                                                                  T.store=T;                                                                                   return success;                                                                       } |
>
> 执行，
>
> | Client | Server |
> | :--- | :--- |
> | if\(获得半数以上Server回复success\){                                                                 向所有Server发送execute\(C\);                                                 } |  |
> |  | 获得execute请求。                                                                   if\(C=C.store\){                                                                                                    执行命令C;                                                                                T.store = 0;                                                                                return  result\(T.store, C.store\);                                               } |

以上是Paxos算法的骨干逻辑，其中T.store作为是否已有准备好的命令存储的标志，若T.store不为零，则需要当前客户端提交执行已存储的命令C.store。这样如果一个Client执行到一半，另一个Client介入时就会继续执行上一个存储的Client命令，根据最终返回result来判断自己的请求是否被执行。

实际上，完整的Paxos算法中包含了Proposer、Accepter和Learner角色，它们拥有着不同的功能。

Proposer：提案者或申请人，提出要表决的提案（Value 对应信息）。

Acceptor：接受者，接受了一个提案后就会把它记录下来。

Learner：学者，接受者接受了那个提案，他就会学习并执行那个提案。  
![](/assets/1.1.8.png)  
步骤如下：

1. Proposer向有一定法定人数（quorum）的Acceptor发出新提案请求。
2. Acceptor确定该新提案的编号大于已记录的提案编号（如果没有已记录的提案则编号为0），记录新提案编号，并返回已记录的提案。
3. Proposer如果有一半以上的Acceptor响应，选择其中已记录的提案的最大编号的提案并使用新提案的编号，如果编号为0则使用新提案，并向返回该信息的Acceptor发出Prepare提案请求。
4. Acceptor判断准备请求的提案编号是否等于最大提案编号（新提案编号），如果是则接受该提案，并返回已接受该提案信息。
5. Proposer接收到超过一半Acceptor发出的接受提案消息，则确定有提案需要被执行（此时可能是提案也有可能是前一个已接受的提案）；发出Execute命令。
6. Acceptor执行提案，发送给Learner学习提案内容。

算法总体上可以分为两大步骤prepare和commit（发出准备提案和执行提交操作），而在实现过程中需要考虑很多问题，如出现多个Proposer同时争抢最大提案编号（ticket），这样Acceptor中记录的最大提案编号永远无法确定，甚至陷入死循环；大多数分布式系统也不会把Proposer定义为Client，处于安全等问题考虑，提供Clients访问Proposer，由Proposer提出提案；同时，Paxos算法也没有提供信息内容的可靠性保障等等。  
Paxos算法是一个保证分布式系统一致性的基础算法，在应用过程中，面对各种各样的问题，我们借鉴该算法，并在此之上进行改进。在区块链中，Paxos算法同样也是借鉴的基础算法之一，但是仅仅只能让私有链能获得最大性能，对于联盟链和公有链则不能适用，因此，这将引出本章第二节和第三节的内容。

