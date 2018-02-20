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

> Paxos算法：
>
> | Client | Server |
> | :--- | :--- |
> | T=0，C=c | T=t，C=0 |
> | 向Server请求T=t |  |
> |  |  |



