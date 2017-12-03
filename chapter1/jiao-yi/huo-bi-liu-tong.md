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



