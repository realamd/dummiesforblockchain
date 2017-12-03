当Bitcoin Core首次运行时，需要从某些节点下载最新状态的blockchain，从哪些节点下载呢？

对于节点地址绝不能采用硬编码方式，因为一旦节点被攻击或者关停，会导致新的节点无法加入网络。Bitcoin Core中会硬编码一些[DNS Seeds](https://bitcoin.org/en/glossary/)地址。这些不是某种节点而是DNS服务器，这些DNS服务器知道节点地址。当启动一次完全干净的Bitcoin Core时，将连接到某个seed，然后获取所有节点的列表，最终从这些节点下载blockchain。

然而，我们将采用集中化的实现方式，将包括以下节点：
