货币流动起来才创造价值，为了达成此目的，我们需要创建一个交易，将该交易放到一个block中，然后通过挖矿挖到（PoW）该block。目前为止，我们仅仅实现了coinbase这种特殊的交易，下面我们实现通用的交易：

```go
func NewUTXOTransaction(from, to string, amount int, bc *Blockchain) *Transaction {
    var inputs []TXInput
    var outputs []TXOutput

    acc, validOutputs := bc.FindSpendableOutputs(from, amount)

    if acc < amount {
        log.Panic("ERROR: Not enough funds")
    }

    // Build a list of inputs
    for txid, outs := range validOutputs {
        txID, err := hex.DecodeString(txid)

        for _, out := range outs {
            input := TXInput{txID, out, from}
            inputs = append(inputs, input)
        }
    }

    // Build a list of outputs
    outputs = append(outputs, TXOutput{amount, to})
    if acc > amount {
        outputs = append(outputs, TXOutput{acc - amount, from}) // a change
    }

    tx := Transaction{nil, inputs, outputs}
    tx.SetID()

    return &tx
}
```

创建TXO前，我们需要找到该账户之前所有交易中的UTXO（**FindSpendableOutputs**函数），然后对于每一个UTXO创建一个TXI引用它，直到交易额满足要求为止。此时所有被引用的UTXO变为TXO。随后，在当前交易我们需要创建TXO：

* 如果引用的UTXO的交易额小于所需，则当前交易创建失败；
* 如果引用的UTXO的交易额等于所需，则当前交易仅有一个TXO；
* 如果引用的UTXO的交易额大于所需，则当前交易有两个TXO。

**FindSpendableOutputs**函数基于**FindUnspentTransactions**实现：

```go
func (bc *Blockchain) FindSpendableOutputs(address string, amount int) (int, map[string][]int) {
    unspentOutputs := make(map[string][]int)
    unspentTXs := bc.FindUnspentTransactions(address)
    accumulated := 0

Work:
    for _, tx := range unspentTXs {
        txID := hex.EncodeToString(tx.ID)

        for outIdx, out := range tx.Vout {
            if out.CanBeUnlockedWith(address) && accumulated < amount {
                accumulated += out.Value
                unspentOutputs[txID] = append(unspentOutputs[txID], outIdx)

                if accumulated >= amount {
                    break Work
                }
            }
        }
    }

    return accumulated, unspentOutputs
}
```

此方法遍历账户所有UTX，遍历过程中进行交易额累加，同时计算累加交易额是否满足需求，当满足需求时停止遍历，并返回UTXO列表以及累加交易额。

接下来修改**Blockchain.MineBlock**方法：

```go
func (bc *Blockchain) MineBlock(transactions []*Transaction) {
    ...
    newBlock := NewBlock(transactions, lastHash)
    ...
}
```

最后，实现**send**命令：

```go
func (cli *CLI) send(from, to string, amount int) {
    bc := NewBlockchain(from)
    defer bc.db.Close()

    tx := NewUTXOTransaction(from, to, amount, bc)
    bc.MineBlock([]*Transaction{tx})
    fmt.Println("Success!")
}
```

消费意味着创建一个交易，然后挖一个block存储该交易，并将block添加到blockchain中。但是比特币的做法不同：比特币不会为一个新的交易马上去挖矿，而是会先将新交易缓存内存池mempool，当矿工即将挖矿时，从内存池中将所有交易取出，整体放到block中，并添加到blockchain。

让我们尝试进行一些交易：

```
$ blockchain_go send -from Ivan -to Pedro -amount 6
00000001b56d60f86f72ab2a59fadb197d767b97d4873732be505e0a65cc1e37

Success!

$ blockchain_go getbalance -address Ivan
Balance of 'Ivan': 4

$ blockchain_go getbalance -address Pedro
Balance of 'Pedro': 6
```

Ivan给了Pedro 6元，此时Ivan余4元，Pedro余6元。然后，Pedro给Helen 2元，Ivan给Helen 2元：

```
$ blockchain_go send -from Pedro -to Helen -amount 2
00000099938725eb2c7730844b3cd40209d46bce2c2af9d87c2b7611fe9d5bdf

Success!

$ blockchain_go send -from Ivan -to Helen -amount 2
000000a2edf94334b1d94f98d22d7e4c973261660397dc7340464f7959a7a9aa

Success!
```

此时，Helen锁定两个TXO：一个来自Pedro，一个来自Ivan。Helen给Rachel 3元，此时Ivan余2元，Pedro余4元，Helen余1元，Rachel余3元：

```
$ blockchain_go send -from Helen -to Rachel -amount 3
000000c58136cffa669e767b8f881d16e2ede3974d71df43058baaf8c069f1a0

Success!

$ blockchain_go getbalance -address Ivan
Balance of 'Ivan': 2

$ blockchain_go getbalance -address Pedro
Balance of 'Pedro': 4

$ blockchain_go getbalance -address Helen
Balance of 'Helen': 1

$ blockchain_go getbalance -address Rachel
Balance of 'Rachel': 3
```

一切OK！尝试一种异常情况：Pedro给Ivan5元，但是Pedro只有4元，消费失败。交易失败前后，Pedro和Ivan的余额未发生变化。

```
$ blockchain_go send -from Pedro -to Ivan -amount 5
panic: ERROR: Not enough funds

$ blockchain_go getbalance -address Pedro
Balance of 'Pedro': 4

$ blockchain_go getbalance -address Ivan
Balance of 'Ivan': 2
```



