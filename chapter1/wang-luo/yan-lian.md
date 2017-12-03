让我们对之前的场景进行演练。

首先，打开一个终端，通过**export NODE\_ID=3000**命令将**NODE\_ID**设置为3000。后续将使用**NODE 3000**或**NODE 3001**来表示哪个节点。

```
$ blockchain_go createblockchain -address CENTREAL_NODE
```

在节点上创建blockchain之后，blockchain中包含一个genesis block，该genesis block作为该节点的blockchain的标识符（在Bitcoin Core中，genesis block是硬编码的）。我们需要将该block单独保存，其他节点会用到该信息：

```
$ cp blockchain_3000.db blockchain_genesis.db
```

**在NODE 3001节点上**

打开一个新终端，将node ID设置为3001，该节点为钱包节点。使用**createwallet**命令创建3个钱包地址：**WALLET\_1**, **WALLET\_2**, **WALLET\_3。**

**在NODE 3000节点上**

向这些钱包地址发送一些货币：

```
$ blockchain_go send -from CENTREAL_NODE -to WALLET_1 -amount 10 -mine
$ blockchain_go send -from CENTREAL_NODE -to WALLET_2 -amount 10 -mine
```

**-mine**参数表示将立即开始进行挖矿，由于初始时网络中没有矿工节点，因此需要该命令行参数。

启动节点：

```
$ blockchain_go startnode
```

该节点将一直运行直到该场景结束。

**在NODE 3001节点上**

将上面保存的genesis block作为该节点的blockchain的初始化信息：

```
$ cp blockchain_genesis.db blockchain_3001.db
```

启动节点：

```
$ blockchain_go startnode
```

该节点将从中心节点下载所有block。为了检查一切是否正常，停止该节点，然后查看钱包地址的余额：

```
$ blockchain_go getbalance -address WALLET_1
Balance of 'WALLET_1': 10

$ blockchain_go getbalance -address WALLET_2
Balance of 'WALLET_2': 10
```

由于**NODE 3001**中已经有blockchain，因此也可以查看**CENTRAL\_NODE**地址的余额：

```
$ blockchain_go getbalance -address CENTRAL_NODE
Balance of 'CENTRAL_NODE': 10
```

**在NODE 3002节点上**

打开一个新终端，将node ID设置为3002，并生成一个钱包，该节点作为矿工节点使用。同理，将上面保存的genesis block作为该节点的blockchain的初始化信息：

```
$ cp blockchain_genesis.db blockchain_3002.db
```

启动节点：

```
 blockchain_go startnode -miner MINER_WALLET
```

**在NODE 3001节点上**

发送一些货币：

```
$ blockchain_go send -from WALLET_1 -to WALLET_3 -amount 1
$ blockchain_go send -from WALLET_2 -to WALLET_4 -amount 1
```

**在NODE 3002节点上**

然后，快速切换到矿工节点所在终端，可以看到正在挖新的block！同样，也可以查看中心节点所在终端有何信息输出。

**在NODE 3001节点上**

切换到钱包节点所在终端，然后启动该节点：

```
$ blockchain_go startnode
```

该节点将下载新的block！

停止该节点并检查余额：

```
$ blockchain_go getbalance -address WALLET_1
Balance of 'WALLET_1': 9

$ blockchain_go getbalance -address WALLET_2
Balance of 'WALLET_2': 9

$ blockchain_go getbalance -address WALLET_3
Balance of 'WALLET_3': 1

$ blockchain_go getbalance -address WALLET_4
Balance of 'WALLET_4': 1

$ blockchain_go getbalance -address MINER_WALLET
Balance of 'MINER_WALLET': 10
```

一切如预期！



