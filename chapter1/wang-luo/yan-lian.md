让我们对之前的场景进行演练。

首先，打开一个终端，通过**export NODE\_ID=3000**命令将**NODE\_ID**设置为3000。后续将使用**NODE 3000**或**NODE 3001**来表示哪个节点。

```
$ blockchain_go createblockchain -address CENTREAL_NODE
```

在节点上创建blockchain之后，blockchain中包含一个genesis block，该genesis block作为该节点的blockchain的标识符（在Bitcoin Core中，genesis block是硬编码的）。我们需要将该block单独保存，其他节点会用到该信息：

```
$ cp blockchain_3000.db blockchain_genesis.db
```



