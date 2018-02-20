# Paxos算法

Paxos算法是解决分布式系统共识问题的重要算法，由Leslie Lamport在1989年提出这个名字，其实它的出生要早几年，对于它的历史，这里不做详述。而Paxos算法对于现代互联网的发展作用，借用Google在2006年Chubby论文中的一句话：“事实上，到目前为止我们所遇到用于解决异步共识问题的算法，其核心都是Paxos”。

为了很好的讲述Paxos算法的内容，我们从简单结构模型开始——客户端/服务器。

![](/assets/1.1.5.png)

上图的功能是，Client给Server发送请求，Server接收到信息后，执行并返回结果。

但如何应对C/S之间的消息传递过程中出现消息丢失呢？如何避免重复发送同一个消息呢？

实际上如果我们在传输过程中添加确认机制（Acknowledgements），也就是说，每次Client在发送请求后，若Server接收到请求则会发送一个AckMessage返回给Client，如果超时还未接收到该消息，则Client补发请求；同时为了避免Server重复执行同一个消息，需要Client在发送的消息中添加一个MessageID，用于识别是否是重复消息。

> Acknowledgements：
>
> * Client向Server发送一条带MessageID的命令。
> * Server接收到命令，会返回一条确认消息AckMessage。
> * 如果Client在一段时间内没有收到确认消息，则重新发送该命令。

这个算法的应用非常广泛，常见的有TCP协议，应用服务有Storm确认消息是否发送的Ack函数等等。

但是这由出现了另一个问题，如果有多个Client呢？

![](/assets/1.1.6.png)

如果出现消息延迟，那Server端接收到的命令顺序也会不一样。

为了解决这个问题，Server端提供了一个锁机制（Locking）。每一个Client只有在拿到Server端的锁（lock），才能够发送请求消息，没有拿到锁的Client需要等待其他Client释放锁，并重新抢占锁。这就是两段式协议（Two Phase Protocol——2PL）的雏形，保证了在同一时间只能有一个Client可以操作Server，避免了消息延迟、多客户端抢占Server带来的问题。



实际上，这个问题也涉及一致性问题的解决，在经典数据库的系统中，两段提交协议（2PC）就是这种机制的实现。





