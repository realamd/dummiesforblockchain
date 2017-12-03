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



