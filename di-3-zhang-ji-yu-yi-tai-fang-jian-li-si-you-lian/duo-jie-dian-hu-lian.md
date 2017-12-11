私有链中多个节点互联需要确保多个节点：

* 拥有相同的genesis block
* 拥有相同的networkid

# 同一机器内节点互联

可以在同一台机器上创建多个节点，只要节点间的端口不冲突即可，下面以创建两个节点为例。

目录规划如下：

```
# mkdir -p $HOME/share/q-btc/local-cluster/{data1,data2}
# cd $HOME/share/q-btc/local-cluster
# vi PrivateGenesis.json
```

打开一个终端，初始化、启动节点1：

```
# cd $HOME/share/q-btc/local-cluster
# geth init PrivateGenesis.json --datadir $HOME/share/q-btc/local-cluster/data1
# geth --datadir $HOME/share/q-btc/local-cluster/data1 \
--identity "PrivateETH Node1" \
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
打开另一个终端，初始化、启动节点2：
# cd $HOME/share/q-btc/local-cluster
# geth init PrivateGenesis.json --datadir $HOME/share/q-btc/local-cluster/data2
# geth --datadir $HOME/share/q-btc/local-cluster/data2 \
--identity "PrivateETH Node2" \
--nodiscover \
--maxpeers 25 \
--rpc \
--rpcapi "*" \
--rpcport 8546 \
--rpccorsdomain "*" \
--port 30304 \
--syncmode "fast" \
--cache=1024 \
--networkid 1999 \
console
```

分别在节点1/2查看peer列表，此时均为空：

```
> admin.peers
[]
```

分别在节点1/2查看block数量，此时均为0：

```
> eth.blockNumber
0
```

分别在节点1/2查看账户情况，此时均没有账户：

```
> eth.accounts
[]
```

为了验证节点间可以同步信息，在节点1上进行挖矿获取一些block。先创建一个账户（之前通过geth命令创建，下面使用console来创建），然后启动挖矿：

```
> personal.newAccount()
Passphrase:输入密码
Repeat passphrase:输入密码
"0x9f6a75647cc8d382f1c236b87c4d37003fa54dd5"
> eth.accounts
["0x9f6a75647cc8d382f1c236b87c4d37003fa54dd5"]
> miner.start(1)
NFO [xx-xx|00:40:07] Successfully sealed new block            number=1 hash=28a8d2…ce7f9f
INFO [xx-xx|00:40:07] 🔨 mined potential block                  number=1 hash=28a8d2…ce7f9f
INFO [xx-xx|00:40:07] Commit new mining work                   number=2 txs=0 uncles=0 elapsed=797.4µs
INFO [xx-xx|00:40:08] Successfully sealed new block            number=2 hash=ac7d21…abc77e
………………
> miner.stop()
> eth.blockNumber
12
```

最终挖到12个block。

由于启动节点时增加了--nodiscover选项，因此需要手动添加，在console中通过admin.addPeer\(nodeurl\)函数添加peer。首先，需要知道两个peer的nodeurl。

分别在节点1/2查看node信息：

节点1 node信息：

```
> admin.nodeInfo.enode
"enode://01dfc3a6cec378232668fddd07479e5607d4e1dd0b50d7ddb7f06c89003e8dc44e2b930db18d0daeccdaec3fdddb3ea1dace2ca022b516834d957c635187d399@[::]:30303?discport=0"
```

节点2 node信息：

```
> admin.nodeInfo.enode
"enode://eb6901ddf5f563e8fe42e744c3c63e96de5417531b2194da3d0dbf8ee68773d024261330ed6aeeebe3ee1eea60be4947676f421f03f64bf875ee5e968ab45449@[::]:30304?discport=0"
```

在节点2上添加节点1的信息：

```
> admin.addPeer("enode://01dfc3a6cec378232668fddd07479e5607d4e1dd0b50d7ddb7f06c89003e8dc44e2b930db18d0daeccdaec3fdddb3ea1dace2ca022b516834d957c635187d399@[::]:30303?discport=0")
true
> admin.peers
[{
    caps: ["eth/63"],
    id: "01dfc3a6cec378232668fddd07479e5607d4e1dd0b50d7ddb7f06c89003e8dc44e2b930db18d0daeccdaec3fdddb3ea1dace2ca022b516834d957c635187d399",
    name: "Geth/PrivateETH Node1/v1.7.3-stable/darwin-amd64/go1.9.2",
    network: {
      localAddress: "[::1]:60802",
      remoteAddress: "[::1]:30303"
    },
    protocols: {
      eth: {
        difficulty: 1707456,
        head: "0xf7d4e393ec7963da14aaeb1ef4e74e1b365a4da8c666cf3c954fef7268597d1b",
        version: 63
      }
    }
}]
> INFO [xx-xx|00:45:07] Block synchronisation started
INFO [xx-xx|00:45:07] Imported new state entries               count=1 elapsed=297.665µs processed=1 pending=0 retry=0 duplicate=0 unexpected=0
INFO [xx-xx|00:45:08] Imported new block headers               count=12 elapsed=1.374s    number=12 hash=f7d4e3…597d1b ignored=0
INFO [xx-xx|00:45:08] Imported new chain segment               blocks=12 txs=0 mgas=0.000 elapsed=3.237ms   mgasps=0.000 number=12 hash=f7d4e3…597d1b
INFO [xx-xx|00:45:08] Fast sync complete, auto disabling

> eth.blockNumber
12
```

在节点2上先通过admin.addPeer添加节点1；然后使用admin.peer查看节点信息，可以看到节点已经添加；接下来输出一系列信息，_**“Fast sync complete, auto disabling”**_表示node2从node1同步信息结束；通过eth.blockNumber可以看到同步了12个block。

此时，再在节点1上启动挖矿，当挖到新的block时，节点2会打印如下信息，表示节点2在同步更新私有链：

```
INFO [xx-xx|00:50:09] Imported new chain segment               blocks=1  txs=0 mgas=0.000 elapsed=5.147ms   mgasps=0.000 number=13 hash=f0e25d…3fc77f
INFO [xx-xx|00:50:19] Imported new chain segment               blocks=1  txs=0 mgas=0.000 elapsed=3.802ms   mgasps=0.000 number=14 hash=998152…644f12
INFO [xx-xx|00:50:19] Imported new chain segment               blocks=1  txs=0 mgas=0.000 elapsed=3.908ms   mgasps=0.000 number=15 hash=d6d0f4…d51980
```

# 跨机器间节点互联

跨机器间的节点互联和同一台机器内的节点互联一样，区别在于添加节点，例如机器2的IP为192.168.1.2，节点标识为enode://999fc3a6cec378232668fddd07479e5607d4e1dd0b50d7ddb7f06c89003e8dc44e2b930db18d0daeccdaec3fdddb3ea1dace2ca022b516834d957c635187d399@\[::\]:30303?discport=0，机器1添加节点调用如下：

```
> admin.addPeer("enode://01dfc3a6cec378232668fddd07479e5607d4e1dd0b50d7ddb7f06c89003e8dc44e2b930db18d0daeccdaec3fdddb3ea1dace2ca022b516834d957c635187d399@192.168.1.2:30303?discport=0")
true
```

只需要将机器2节点标识中的\[::\]替换为机器2的IP即可。

> **添加节点通常会出现失败的情况，通常是由于：**
>
> * 机器间时间不同步。以太坊网络对于时间同步有很高的要求，两台机器即使有10几秒的差异也会产生一定问题
> * 防火墙禁止UDP协议
>
> **静态节点**
>
> 上面通过admin.addPeer\(\)来联机到这些节点，也可以通过配以一个文件static-node.js来声明静态节点，当节点启动后，会自动添加该文件中的节点。static-node.js放置于${datadir}中，如$HOME/share/q-btc/local-cluster/data1中，文件内容格式为：
>
> \[
>
> enodeurl,
>
> enodeurl,
>
> ...
>
> \]

# 目前构成了一个2机器3节点的私有链网络：![](/assets/3.1.20.png)



