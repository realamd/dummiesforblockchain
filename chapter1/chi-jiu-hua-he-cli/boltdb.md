为什么选用BoltDB？理由如下：

> * 足够简洁
> * 基于Go实现
> * 不需要服务端

BoltDB在Github上的说明：

> _Bolt是基于纯Go语言开发的KV存储，灵感来自于Howard Chu的LMDB项目。该项目目标是开发一个简单、快速、可靠的无服务端的数据库。API非常小巧和简洁，仅仅关注如何获取或设置数据，这就是全部。_

听起来和我们的需求非常匹配！让我们花几分钟研究一下。

BoltDB是K-V存储，没有关系型数据库中类似表、行、列结构，数据以key-value对存储（类似GO语言中的map）。相似的key-value对存储在同一bucket中，类似于关系型数据库中的Table。因此，为了获取一个value，需要知道其所在的bucket以及对应的key。

BoltDB中没有数据类，key和value都是字节数组。我们的blockchain数据均为struct，因此为了在BoltDB中存取数据，需要对struct进行序列化和反序列化。虽然**JSON**, **XML**, **Protocol Buffers**均可以到达相同的目的，我们还是决定采用encoding/gob库来完成此工作，一方面是因为使用简单，另一方面是因为该模块是Go标准库的一部分。



