当完成一个智能合约\(Smart Contract\)或DApp\(Decentralization Application\)应用时需要进行测试，可以接入以太坊网络进行测试，但是一方面效率很低，需要和公有链上的其他节点进行竞争；另一方面需要消耗真实的以太币\(Ether\)，后者恐怕是最不希望看到的情况。

这种情况下，可以使用以太坊建立私有链\(Private Network\)对智能合约和DApp应用在本地进行测试，不仅高效而且不用消耗真实的以太币。

# Genesis文件

Genesis block（世纪块或始祖块）是区块链中第一个块，是唯一一个没有前序块的区块。以太坊允许通过自定义Genesis文件来创建Genesis block。拥有不同Genesis block的节点一定不会达成一致，这是由一致性算法保证的。

Genesis文件示例如下：

```
{
  "nonce": "0x0000000000000042",
  "difficulty": "0x020000",
  "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "coinbase": "0x0000000000000000000000000000000000000000",
  "timestamp": "0x00",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "extraData": "0x11bbe8db4e347b4e8c937c1c8370e4b5ed33adb3db69cbdb7a38e1e50b1b82fa",
  "gasLimit": "0x4c4b40",
  "config": {
      "chainId": 15,
      "homesteadBlock": 0,
      "eip155Block": 0,
      "eip158Block": 0
  },
  "alloc": { }
}
```

**mixhash 和nonce**

> mixhash长度为256位，nonce长度为64位，两者用于PoW系统。矿工挖矿时需要借助于两者不断计算得到一个值，当该值在数学意义上满足某种条件（参考Yellowpaper, 4.3.4. Block Header Validity, \(44\)、Yellowpager, 11.5. Mining Proof-of-Work）时则证明挖到block。

**difficulty **

> 挖矿难度等级，通过该值确定挖矿目标。挖矿目标是根据前序block的难度等级和timestamp计算得到的。难度等级越高，挖矿时间越长，进而交易确认生效时间也越长。对于私有链，可以降低难度等级便于快速测试。

**alloc **

> alloc可以包含一些钱包，这样通过以太坊特有的“预售”特性，可以提前给这些钱包充一定数量的ether。私有链中挖矿难度很低，因此不需要特别设置该选项。

**coinbase **

> The 160-bit address to which all rewards \(in Ether\) collected from the successful mining of this block have been transferred. They are a sum of the mining reward itself and the Contract transaction execution refunds. Often named “beneficiary” in the specifications, sometimes “etherbase” in the online documentation. This can be anything in the Genesis Block since the value is set by the setting of the Miner when a new Block is created.

**timestamp **

> A scalar value equal to the reasonable output of Unix time\(\) function at this block inception. This mechanism enforces a homeostasis in terms of the time between blocks. A smaller period between the last two blocks results in an increase in the difficulty level and thus additional computation required to find the next valid block. If the period is too large, the difficulty, and expected time to the next block, is reduced. The timestamp also allows verifying the order of block within the chain \(Yellowpaper, 4.3.4. \(43\)\).

**parentHash **

> 长度256位，指向前序block，Genesis block没有前序block，填充为0。

**extraData **

> 选填项，长度32字节，可以填充一些除了交易外的数据。

**gasLimit **

> A scalar value equal to the current chain-wide limit of Gas expenditure per block. High in our case to avoid being limited by this threshold during tests. Note: this does not indicate that we should not pay attention to the Gas consumption of our Contracts.

创建一个目录用于保存私有连相关信息：

```
# mkdir -p $HOME/share/q-btc/data
```

将上面内容保存到文件中，命名为PrivateGenesis.json：

```
# cd $HOME/share/q-btc
# vi PrivateGenesis.json
```

# 初始化节点

使用上面创建的Genesis文件初始化节点：

```
# cd $HOME/share/q-btc/data
# geth init PrivateGenesis.json --datadir $HOME/share/q-btc/data
WARN [xx-xx|13:26:56] No etherbase set and no accounts found as default
INFO [xx-xx|13:26:56] Allocated cache and file handles         database=$HOME/share/q-btc/data/geth/chaindata cache=16 handles=16
INFO [xx-xx|13:26:56] Writing custom genesis block
INFO [xx-xx|13:26:56] Successfully wrote genesis state         database=chaindata hash=611596…424d04
INFO [xx-xx|13:26:56] Allocated cache and file handles         database=$HOME/share/q-btc/data/geth/lightchaindata cache=16 handles=16
INFO [xx-xx|13:26:56] Writing custom genesis block
INFO [xx-xx|13:26:56] Successfully wrote genesis state         database=lightchaindata                                      hash=611596…424d04
```

初始化后，在$HOME/share/q-btc/data生成一系列目录及文件：

```
data
├── geth
│   ├── LOCK
│   ├── chaindata
│   │   ├── 000002.ldb
│   │   ├── 000003.log
│   │   ├── CURRENT
│   │   ├── LOCK
│   │   ├── LOG
│   │   └── MANIFEST-000004
│   ├── lightchaindata
│   │   ├── 000001.log
│   │   ├── CURRENT
│   │   ├── LOCK
│   │   ├── LOG
│   │   └── MANIFEST-000000
│   ├── nodekey
│   ├── nodes
│   │   ├── 000001.log
│   │   ├── CURRENT
│   │   ├── LOCK
│   │   ├── LOG
│   │   └── MANIFEST-000000
│   └── transactions.rlp
└── keystore
```

**注意：**由于使用--datadir自定义数据目录，后续geth命令使用时都需要加上该参数。默认数据目录为$HOME/Library/Ethereum。

# 创建账户

geth account命令用于管理账户，支持许多子命令：

> * geth account list         打印所有账户
> * geth account new         创建新账户
> * geth account update         更新账户信息
> * geth account import         导入私钥生成新的账户
> * geth account help         打印帮助

初始化私有链后没有任何账户：

```
# geth --datadir $HOME/share/q-btc/data account list
WARN [xx-xx|22:27:19] No etherbase set and no accounts found as default
```

生成一个包含明文密码的文件，并创建一个账户：

```
# cd $HOME/share/q-btc
# echo 123 > account.pwd
# geth --datadir $HOME/share/q-btc/data --password account.pwd account new
Address: { 205c6e56f2b809d686b4afc42b241004c985c900 }
# geth --datadir $HOME/share/q-btc/data account list
Account #0: {205c6e56f2b809d686b4afc42b241004c985c900} keystore://$HOME/share/q-btc/data/keystore/UTC--2017-xx-xxT14-50-17.692945588Z--205c6e56f2b809d686b4afc42b241004c985c900
```

此时生成了一个新账户，账户地址为**{ 205c6e56f2b809d686b4afc42b241004c985c900 }**。

# 启动节点

若之前部署过以太坊，请先删除$HOME/.ethash目录，通过如下命令启动节点：

```
# geth --datadir $HOME/share/q-btc/data \
--identity "PrivateETH" \
--nodiscover \
--maxpeers 25 \
--rpc \
--rpcapi "*" \
--rpcport 8545 \
--rpccorsdomain "*" \
--port 30303 \
--syncmode "fast" \
--cache=1024 \
--networkid 1999 \
console
```

命令行参数的含义：

--datadir "$HOME/share/q-btc/data"

> 该选项设置私有链数据存储目录，默认目录为$HOME/Library/Ethereum。建议选择一个独立于存放以太坊公有链数据目录的其它目录。

--identity

> 节点身份标识。

--nodiscover

> 若不设置该选项，当私有连节点的networkid与其他节点的networkid相同时，其他节点会将私有连节点自动加入其所在的私有链中，这是不希望看到的（当然若genesis block不同也会不添加成功，不过通过admin.peers可以看到一会有peer一会又没有了，这就是被发现添加，但是genesis block不同而添加失败）。通过该选项，可以禁止私有链节点被其他节点发现，除非手动加入。

--maxpeers 25

> 该选项指定可以加入到私有链网络的最大节点数，默认25。当设置为0时，任何节点都无法加入私有链网络。

--rpc

> 该选项将启用HTTP-RPC接口，geth默认启用。

--rpcapi "db,eth,net,web3"

> 该选项指定可以通过HTTP-RPC接口访问哪些API。默认情况下，geth启用所有IPC接口的所有API，以及RPC接口的db,eth,net,web3这几类API。

--rpcport "8545"

> HTTP-RPC服务监听端口，默认为8545。

--rpccorsdomain "\*"

> 该选项指定接收来自于哪些URL连接的初始请求。真实环境中，应该设置一个特定的URL。由于私有链或私有链不涉及真实的以太币，因此可以设置为\*通配符便于测试，这样可以使用站点（如Browser Solidity）或者DApps（如Notareth）连接网络。

--port "30303"

> P2P网络监听端口，通过该端口可以手动地连接到其它节点。

--syncmode "fast"

> blockchain数据同步模式：full，fast或light。

--cache=1024

> 缓存大小，单位MB，最小16，默认120。

--networkid 1999

> 网络ID，用于标识私有链网络，不同私有链的网络ID不同。默认为1，1表示太坊公有链；2表示Morden私有链网络，已不使用；3表示Ropsten私有链网路；4表示Rinkeby私有链网络。

私有链启动后，会打印如下信息：

```
INFO [xx-xx|23:40:46] Starting peer-to-peer node               instance=Geth/PrivateETH/v1.7.3-stable/darwin-amd64/go1.9.2
INFO [xx-xx|23:40:46] Allocated cache and file handles         database=$HOME/share/q-btc/data/geth/chaindata cache=1024 handles=1024
INFO [xx-xx|23:40:46] Initialised chain configuration          config="{ChainID: 15 Homestead: 0 DAO: <nil> DAOSupport: false EIP150: <nil> EIP155: 0 EIP158: 0 Byzantium: <nil> Engine: unknown}"
INFO [xx-xx|23:40:46] Disk storage enabled for ethash caches   dir=$HOME/share/q-btc/data/geth/ethash count=3
INFO [xx-xx|23:40:46] Disk storage enabled for ethash DAGs     dir=$HOME/.ethash                      count=2
INFO [xx-xx|23:40:46] Initialising Ethereum protocol           versions="[63 62]" network=1999
INFO [xx-xx|23:40:46] Loaded most recent local header          number=0 hash=611596…424d04 td=131072
INFO [xx-xx|23:40:46] Loaded most recent local full block      number=0 hash=611596…424d04 td=131072
INFO [xx-xx|23:40:46] Loaded most recent local fast block      number=0 hash=611596…424d04 td=131072
INFO [xx-xx|23:40:46] Loaded local transaction journal         transactions=0 dropped=0
INFO [xx-xx|23:40:46] Regenerated local transaction journal    transactions=0 accounts=0
INFO [xx-xx|23:40:46] Starting P2P networking
INFO [xx-xx|23:40:47] RLPx listener up                         self="enode://2d59b92a845dc555276ae565a5b26d43c2c4e1505efc84666b1bd893d7a0e050d0a26af72618d32aa33b35c7034d9f041bbbc019b19c4d21c23393af0dad6ef0@[::]:30303?discport=0"
INFO [xx-xx|23:40:47] IPC endpoint opened: $HOME/share/q-btc/data/geth.ipc
INFO [xx-xx|23:40:47] HTTP endpoint opened: http://127.0.0.1:8545
Welcome to the Geth JavaScript console!

instance: Geth/PrivateETH/v1.7.3-stable/darwin-amd64/go1.9.2
coinbase: 0x205c6e56f2b809d686b4afc42b241004c985c900
at block: 0 (Thu, 01 Jan 1970 08:00:00 CST)
 datadir: $HOME/share/q-btc/data
 modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0

> INFO [xx-xx|23:40:49] Mapped network port                      proto=tcp extport=30303 intport=30303 interface="UPNP IGDv1-PPP1"
```

打印的信息主要包含私有链的配置信息，如缓存存放目录、数据存放目录、DAG存放目录、IPC地址等。随后进入Geth JavaScript控制台（&gt;为控制台提示符），该控制台可执行JavaScript代码，同时内置的许多以太坊对象：

> * eth            用于操作区块链
> * net            用于查看p2p网络状态
> * admin        用于管理节点
> * miner        用于启动&停止挖矿
> * personal    用于管理账户
> * txpool        用于查看交易内存池
> * web3        上述对象的父对象，此外包括一些单位换算对象

# 挖矿

启动节点进入控制台后，就可以开始挖矿：

```
> miner.start(1)
INFO [xx-xx|00:01:06] Updated mining threads                   threads=1
INFO [xx-xx|00:01:06] Transaction pool price threshold updated price=18000000000
INFO [xx-xx|00:01:06] Starting mining operation
null
> INFO [xx-xx|00:01:06] Commit new mining work                   number=1 txs=0 uncles=0 elapsed=2.244ms
INFO [xx-xx|00:01:21] Generating DAG in progress               epoch=0 percentage=1 elapsed=13.218s
INFO [xx-xx|00:01:22] Generating DAG in progress               epoch=0 percentage=2 elapsed=14.693s
INFO [xx-xx|00:01:24] Generating DAG in progress               epoch=0 percentage=3 elapsed=16.200s
…………
INFO [xx-xx|00:06:54] Generating DAG in progress               epoch=1 percentage=97 elapsed=3m0.262s
INFO [xx-xx|00:06:55] Generating DAG in progress               epoch=1 percentage=98 elapsed=3m1.942s
INFO [xx-xx|00:06:57] Generating DAG in progress               epoch=1 percentage=99 elapsed=3m3.726s
INFO [xx-xx|00:06:57] Generated ethash verification cache      epoch=1 elapsed=3m3.729s
INFO [xx-xx|00:07:07] 🔨 mined potential block                  number=2 hash=af3ac8…f49b4d
INFO [xx-xx|00:07:07] Commit new mining work                   number=3 txs=0 uncles=0 elapsed=2.094ms
INFO [xx-xx|00:07:07] Successfully sealed new block            number=3 hash=7498af…379059
INFO [xx-xx|00:07:07] 🔨 mined potential block                  number=3 hash=7498af…379059
INFO [xx-xx|00:07:07] Commit new mining work                   number=4 txs=0 uncles=0 elapsed=192.358µs
INFO [xx-xx|00:07:08] Successfully sealed new block            number=4 hash=c3e532…36a2ba
…………
```

通过设置mine.start\(n\)的参数n来控制同时挖矿的线程数，默认使用创建的第一个账户进行挖矿。启动挖矿之初，先要创建DAG文件，当创建完成后（percentage=99）会打印DAG验证信息（上面红色字体），然后就会开始挖矿。此时会发现挖矿速度非常快，远远高于以太坊公有链的速度，这是因为有意在Genesis文件中将挖矿难度变得非常低：_**"nonce": "0x0000000000000042"**_，这样便于在测试网络中快速挖到以太币进行测试。

> 什么是DAG？
>
> Ethash是一个工作量证明\(PoW\)的系统，为了实现PoW需要准备大小约为1GB的数据集，称为DAG。由于DAG数据集的生成需要花费一定的时间，因此第一次生成后该数据集将被缓存起来便于其他客户端共享该数据集。
>
> DAG数文件存放在：
>
> * Mac/Linux：$HOME/.ethash/full-R&lt;REVISION&gt;-&lt;SEEDHASH&gt;
> * Windows：$HOME/AppData/Local/Ethash/full-R&lt;REVISION&gt;-&lt;SEEDHASH&gt;
>
> DAG文件以8字节的魔数0xfee1deadbaddcafe开头（大端存储，fe ca dd ba ad de e1 fe）。

可以使用如下命令停止挖矿，成功会返打印true：

```
> miner.stop()
true
```

查看当前账户列表，目前仅有一个账户：

```
> eth.accounts
["0x205c6e56f2b809d686b4afc42b241004c985c900"]
```

查看该用户/矿工的余额，eth.getBalance返回值是以太币最小单位（Wei，Ether = 1018Wei）的数量，可以使用web3.fromWei转换成实际的以太币数量：

```
> eth.getBalance(eth.accounts[0])
1.59e+21
> primary = eth.accounts[0]
"0x205c6e56f2b809d686b4afc42b241004c985c900"
> balance = web3.fromWei(eth.getBalance(primary), "ether")
1590
```



