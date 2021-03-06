当Bitcoin Core首次运行时，需要从某些节点下载最新状态的blockchain，从哪些节点下载呢？

对于节点地址绝不能采用硬编码方式，因为一旦节点被攻击或者关停，会导致新的节点无法加入网络。Bitcoin Core中会硬编码一些[DNS Seeds](https://bitcoin.org/en/glossary/)地址。这些不是某种节点而是DNS服务器，这些DNS服务器知道节点地址。当启动一次完全干净的Bitcoin Core时，将连接到某个seed，然后获取所有节点的列表，最终从这些节点下载blockchain。

然而，我们将采用集中化的实现方式，将包括以下节点：

> 1. 中心节点：所有其他节点均连接到此节点；该节点向其他节点发送数据
> 2. 矿工节点：将交易存储在mempool中，当交易数量满足要求时，开始挖新的block
> 3. 钱包节点：用于在钱包之间发送钱币。与SPV节点不同，该节点存储完整的blockchain

# **场景**

本文的目标是实现以下场景：

> 1. 中心节点创建blockchain
> 2. 其他钱包节点连接到中心节点，并下载blockchain
> 3. 一个或多个矿工节点连接到中心节点，并下载blockchain
> 4. 钱包节点创建一个交易
> 5. 矿工节点收到此交易并保存在mempool中
> 6. 当mempool有足够交易时，矿工开始挖矿
> 7. 当新的block生成时，其将被发送到中心节点
> 8. 钱包节点与中心节点进行同步
> 9. 钱包节点的拥有者将检查支付是否成功

整个过程与比特币很像，虽然没有实现一个真实的P2P网络，但是该场景仍然是比特币中最重要、最主要的用例。

# **version消息**

节点间借助于消息进行通信。当新节点启动后，将从一个DNS seed获取节点列表，并向这些节点发送**version**消息。version结构如下：

```go
type version struct {
    Version    int
    BestHeight int
    AddrFrom   string
}
```

由于仅仅有一个blockchain版本，因此**Version**字段不存储任何重要信息。**BestHeight**存放该节点的blockchain的长度。**AddFrom**存放消息发送者的地址。

当一个节点收到**version**消息后，需要做些什么呢？它将发送自身的**version**消息给对方。这是一种握手协议：不握手不能做其他任何事情。**version**用于发现一个更长的blockchain。当一个节点收到**version**消息后，将会检查发送节点的**version.BestHeight**值是否更大（即blockchain更长），如果发送节点的blockchain更长的话，接收节点将会发送请求获取缺失的block。

创建一个Server用于接收消息：

```go
var nodeAddress string
var knownNodes = []string{"localhost:3000"}

func StartServer(nodeID, minerAddress string) {
    nodeAddress = fmt.Sprintf("localhost:%s", nodeID)
    miningAddress = minerAddress
    ln, err := net.Listen(protocol, nodeAddress)
    defer ln.Close()

    bc := NewBlockchain(nodeID)

    if nodeAddress != knownNodes[0] {
        sendVersion(knownNodes[0], bc)
    }

    for {
        conn, err := ln.Accept()
        go handleConnection(conn, bc)
    }
}
```

首先，将中心节点地址硬编码到程序中：一开始每个节点必须要知道该地址。**minerAddress**参数表示接收挖矿奖励的地址。

```go
if nodeAddress != knownNodes[0] {
    sendVersion(knownNodes[0], bc)
}
```

若当前节点不是中心节点，需要向中心节点发送**version**消息，用于检查当前节点的blockchain是否过时。

```go
func sendVersion(addr string, bc *Blockchain) {
    bestHeight := bc.GetBestHeight()
    payload := gobEncode(version{nodeVersion, bestHeight, nodeAddress})

    request := append(commandToBytes("version"), payload...)

    sendData(addr, request)
}
```

消息由字节数组构成：前12个字节表示命令名（此时是“version”），紧接着是gob编码的消息体。

```go
func commandToBytes(command string) []byte {
    var bytes [commandLength]byte

    for i, c := range command {
        bytes[i] = byte(c)
    }

    return bytes[:]
}
```

**commandToBytes**函数创建长度为12的字节数组，然后将命令名填充到其中。

```go
func bytesToCommand(bytes []byte) string {
    var command []byte

    for _, b := range bytes {
        if b != 0x0 {
            command = append(command, b)
        }
    }

    return fmt.Sprintf("%s", command)
}
```

**bytesToCommand**与**commandToBytes**作用正好相反：当一个节点接收到一个命令时，通过**bytesToCommand**函数从中解析出命令名和消息体。**handleConnection**函数会根据解析出的命令名来调用不同的消息处理函数。

```go
func handleConnection(conn net.Conn, bc *Blockchain) {
    request, err := ioutil.ReadAll(conn)
    command := bytesToCommand(request[:commandLength])
    fmt.Printf("Received %s command\n", command)

    switch command {
    ...
    case "version":
        handleVersion(request, bc)
    default:
        fmt.Println("Unknown command!")
    }

    conn.Close()
}
```

**version**的消息处理函数如下：

```go
func handleVersion(request []byte, bc *Blockchain) {
    var buff bytes.Buffer
    var payload verzion

    buff.Write(request[commandLength:])
    dec := gob.NewDecoder(&buff)
    err := dec.Decode(&payload)

    myBestHeight := bc.GetBestHeight()
    foreignerBestHeight := payload.BestHeight

    if myBestHeight < foreignerBestHeight {
        sendGetBlocks(payload.AddrFrom)
    } else if myBestHeight > foreignerBestHeight {
        sendVersion(payload.AddrFrom, bc)
    }

    if !nodeIsKnown(payload.AddrFrom) {
        knownNodes = append(knownNodes, payload.AddrFrom)
    }
}
```

首先，对请求中的数据进行解码获取消息体，这对所有消息处理函数都是一样的，后续介绍其他消息处理函数时将不再进行介绍。

随后，接收节点将自身的**BestHeight**与消息体（即发送节点）中的**BestHeight**进行比较。若接收节点的blockchain更长，将发送**version**消息；若发送节点的blockchain更长，将发送**getblocks**消息。

# **getblocks消息**

```go
type getblocks struct {
    AddrFrom string
}
```

**getblocks**消息（比特币实现中更加复杂）：返回当前节点所拥有的block的hash值列表，而不是所有block的详细信息。出于降低网络负荷的目的，若需要下载block，可以从多个节点同时下载，没必要从单个节点下载。相应的消息处理函数也很简单：

```go
func handleGetBlocks(request []byte, bc *Blockchain) {
    ...
    blocks := bc.GetBlockHashes()
    sendInv(payload.AddrFrom, "block", blocks)
}
```

我们的实现中，该消息处理函数通过发送inv消息返回所有block的hash值。

# **inv消息**

```go
type inv struct {
    AddrFrom string
    Type     string
    Items    [][]byte
}
```

**inv**消息包含发送节点所拥有的block或交易的hash值列表。**Type**用于表示是block还是交易。该消息处理函数如下：

```go
func handleInv(request []byte, bc *Blockchain) {
    ...
    fmt.Printf("Recevied inventory with %d %s\n", len(payload.Items), payload.Type)

    if payload.Type == "block" {
        blocksInTransit = payload.Items

        blockHash := payload.Items[0]
        sendGetData(payload.AddrFrom, "block", blockHash)

        newInTransit := [][]byte{}
        for _, b := range blocksInTransit {
            if bytes.Compare(b, blockHash) != 0 {
                newInTransit = append(newInTransit, b)
            }
        }
        blocksInTransit = newInTransit
    }

    if payload.Type == "tx" {
        txID := payload.Items[0]

        if mempool[hex.EncodeToString(txID)].ID == nil {
            sendGetData(payload.AddrFrom, "tx", txID)
        }
    }
}
```

为了跟踪已下载block信息，当block的hash值完成传输后，将被保存到**blocksInTransit**中，这样做可以使我们从不同的节点下载block。一旦block置为transit状态后，将向inv消息的发送节点发送**getdata**消息并更新**blocksInTransit**。在真实的P2P网络中，将从不同的节点传输block信息。

在我们的实现中，inv消息中仅有一个block或交易的hash值，这也就是当**payload.Type == "tx"**时仅仅获取第一个hash值的原因。当mempool没有找到对应block或交易时，将发送**getdata**消息获取相关信息。

# getdata**消息**

```go
type getdata struct {
    AddrFrom string
    Type     string
    ID       []byte
}
```

**getdata**消息用于获取某个block或交易信息

```go
func handleGetData(request []byte, bc *Blockchain) {
    ...
    if payload.Type == "block" {
        block, err := bc.GetBlock([]byte(payload.ID))

        sendBlock(payload.AddrFrom, &block)
    }

    if payload.Type == "tx" {
        txID := hex.EncodeToString(payload.ID)
        tx := mempool[txID]

        sendTx(payload.AddrFrom, &tx)
    }
}
```

该消息处理函数根据消息中的数据类型，返回block或者交易。需要注意的是，该函数没有检查对应的block或交易是否存在。

# **block和tx消息**

```go
type block struct {
    AddrFrom string
    Block    []byte
}

type tx struct {
    AddFrom     string
    Transaction []byte
}
```

这些消息表示实际的数据（block或交易信息）。

**block**消息处理函数很简单：

```go
func handleBlock(request []byte, bc *Blockchain) {
    ...

    blockData := payload.Block
    block := DeserializeBlock(blockData)

    fmt.Println("Recevied a new block!")
    bc.AddBlock(block)

    fmt.Printf("Added block %x\n", block.Hash)

    if len(blocksInTransit) > 0 {
        blockHash := blocksInTransit[0]
        sendGetData(payload.AddrFrom, "block", blockHash)

        blocksInTransit = blocksInTransit[1:]
    } else {
        UTXOSet := UTXOSet{bc}
        UTXOSet.Reindex()
    }
}
```

当收到一个新block时，将该block加入blockchain中。如果还有其他待下载的block，将向相同的节点发送消息来获取block信息。当所有block均已下载，UTXO将被重建。

> _待完善:目前实现认为新block是有效的，后续在新block加入blockchain之前，应该对其进行有效性验证。_
>
> _待完善:目前实现使用UTXOSet.Reindex\(\)对UTXO进行重建，后续应该使用UTXOSet.Update\(block\)，避免重建整个blockchain带来的性能影响。_

**tx**消息处理函数更复杂一些：

```go
func handleTx(request []byte, bc *Blockchain) {
    ...
    txData := payload.Transaction
    tx := DeserializeTransaction(txData)
    mempool[hex.EncodeToString(tx.ID)] = tx

    if nodeAddress == knownNodes[0] {
        for _, node := range knownNodes {
            if node != nodeAddress && node != payload.AddFrom {
                sendInv(node, "tx", [][]byte{tx.ID})
            }
        }
    } else {
        if len(mempool) >= 2 && len(miningAddress) > 0 {
        MineTransactions:
            var txs []*Transaction

            for id := range mempool {
                tx := mempool[id]
                if bc.VerifyTransaction(&tx) {
                    txs = append(txs, &tx)
                }
            }

            if len(txs) == 0 {
                fmt.Println("All transactions are invalid! Waiting for new ones...")
                return
            }

            cbTx := NewCoinbaseTX(miningAddress, "")
            txs = append(txs, cbTx)

            newBlock := bc.MineBlock(txs)
            UTXOSet := UTXOSet{bc}
            UTXOSet.Reindex()

            fmt.Println("New block is mined!")

            for _, tx := range txs {
                txID := hex.EncodeToString(tx.ID)
                delete(mempool, txID)
            }

            for _, node := range knownNodes {
                if node != nodeAddress {
                    sendInv(node, "block", [][]byte{newBlock.Hash})
                }
            }

            if len(mempool) > 0 {
                goto MineTransactions
            }
        }
    }
}
```

首先，将新交易放入mempool（放入前应该进行有效性验证）。

```go
if nodeAddress == knownNodes[0] {
    for _, node := range knownNodes {
        if node != nodeAddress && node != payload.AddFrom {
            sendInv(node, "tx", [][]byte{tx.ID})
        }
    }
}
```

检查当前节点是否为中心节点，在我们的视线中由于中心节点不进行挖矿操作，因此若是中心节点则将新交易转发给网络中的其他节点。

若当前节点为矿工节点，将进行如下操作。让我们一步一步来看：

```go
if len(mempool) >= 2 && len(miningAddress) > 0 {
```

仅仅矿工节点设置 **miningAddress**，若当前矿工节点的mempool包含两个以上的交易时，开始挖矿：

```go
for id := range mempool {
    tx := mempool[id]
    if bc.VerifyTransaction(&tx) {
        txs = append(txs, &tx)
    }
}

if len(txs) == 0 {
    fmt.Println("All transactions are invalid! Waiting for new ones...")
    return
}
```

首先，对mempool中的所有交易进行验证，忽略所有无效交易，若没有任何有效交易，则停止挖矿。

```go
cbTx := NewCoinbaseTX(miningAddress, "")
txs = append(txs, cbTx)

newBlock := bc.MineBlock(txs)
UTXOSet := UTXOSet{bc}
UTXOSet.Reindex()

fmt.Println("New block is mined!")
```

所有有效交易和一个用于奖励的coinbase交易将被放到一个block中，当完成挖矿后，UTXO将被重建。

> _待完善:与之前一样，后续应该使用UTXOSet.Update\(block\)，避免重建整个blockchain带来的性能影响。_

```go
for _, tx := range txs {
    txID := hex.EncodeToString(tx.ID)
    delete(mempool, txID)
}

for _, node := range knownNodes {
    if node != nodeAddress {
        sendInv(node, "block", [][]byte{newBlock.Hash})
    }
}

if len(mempool) > 0 {
    goto MineTransactions
}
```

当挖矿结束后，从mempool中移除交易信息，同时当前节点将向其他节点发送**inv**消息，其中包含最新挖到的block的hash值。

