在[持久化和CLI](/chapter1/chi-jiu-hua-he-cli.md)中，曾经提到比特币将block存储在数据库：block存储在**blocks**数据库；交易TXO存储在**chainstate**数据库。现在重新回顾下**chainstate**的结构：

1. **'c' + 32-byte transaction hash -&gt; unspent transaction output record for that transaction**
2. **'B' -&gt; 32-byte block hash: the block hash up to which the database represents the unspent transaction outputs**

在此之前，我们实现了交易，但是没有将交易信息存储到**chainstate**数据库。现在，我们来实现此机制。

**chainstate**数据库不存储交易本身，而是存储所有UTXO，称为UTXO集合。此外，**chainstate**数据库也会存储“the block hash up to which the database represents the unspent transaction outputs”，不过目前用不到block高度的的信息，故暂不实现。

那么，UTXO集合有什么用呢？

让我们回顾下**FindUnspentTransactions**方法：

```go
func (bc *Blockchain) FindUnspentTransactions(pubKeyHash []byte) []Transaction {
    ...
    bci := bc.Iterator()

    for {
        block := bci.Next()

        for _, tx := range block.Transactions {
            ...
        }

        if len(block.PrevBlockHash) == 0 {
            break
        }
    }
    ...
}
```

该方法返回所有UTXO。由于交易存储在block中，因此需要遍历blockchain中的所有block，对block中的所有交易进行检查。截止2017/9/18，比特币有485860个block，整个数据库大小超过140GB。这意味着需要遍历整个数据库中的所有block对交易进行检查。可想而知效率会非常低！

为了解决此问题，我们需要对UTXO建立索引，这就是所谓的UTXO集合。UTXO集合其实是一个缓存，遍历blockchain中所有交易得到的，用于计算余额以及交易验证。截止2017/9，比特币的UTXO集合大小约为2.7GB。

目前，下述方法用于查找交易：

> 1. Blockchain.FindUnspentTransactions – 遍历所有block返回UTXO
> 2. Blockchain.FindSpendableOutputs – 新交易创建时，通过此函数找到满足要求的可以供消费的交易。此函数使用到了Blockchain.FindUnspentTransactions
> 3. Blockchain.FindUTXO – 返回指定公钥hash值的所有UTXO，用于计算余额。此函数使用到了Blockchain.FindUnspentTransactions
> 4. Blockchain.FindTransaction  - 根据交易ID查找交易。该函数会遍历所有block进行查找。

上述方法均会遍历整个数据库中所有block，但由于UTXO集合仅仅存储UTXO不存储所有交易，因此UTXO集合无法为上述所有方法带来改善，如Blockchain.**FindTransaction**方法。

所以，我们考虑使用下面的新方法：

> 1. Blockchain.FindUTXO – 遍历block返回所有UTXO
> 2. UTXOSet.Reindex – 将Blockchain.FindUTXO结果存储到数据库
> 3. UTXOSet.FindSpendableOutputs  - 和Blockchain. FindSpendableOutputs类似，只不过基于UTXO集合进行计算
> 4. UTXOSet.FindUTXO - 和Blockchain.FindUTXO 类似，只不过基于UTXO集合进行计算
> 5. Blockchain.FindTransaction保持不变

现在，两个使用最频繁的方法均使用了UTXO集合做缓存：

```go
type UTXOSet struct {
    Blockchain *Blockchain
}
```

我们仍使用同一个数据库，将UTXO集合存储在不同的bucket中。因此，**UTXOSet**和**Blockchain**相伴而生。

```go
func (u UTXOSet) Reindex() {
    db := u.Blockchain.db
    bucketName := []byte(utxoBucket)

    err := db.Update(func(tx *bolt.Tx) error {
        err := tx.DeleteBucket(bucketName)
        _, err = tx.CreateBucket(bucketName)
    })

    UTXO := u.Blockchain.FindUTXO()

    err = db.Update(func(tx *bolt.Tx) error {
        b := tx.Bucket(bucketName)

        for txID, outs := range UTXO {
            key, err := hex.DecodeString(txID)
            err = b.Put(key, outs.Serialize())
        }
    })
}
```

首先，若UTXO集合存在，将其删除；然后获取所有UTXO；最后将其保存到bucket中。

**Blockchain.FindUTXO**几乎与**Blockchain.FindUnspentTransactions**完全一致，只不过现在该方法返回**TransactionID → TransactionOutputs**对。

现在，发送货币过程可以用到UTXO集合：

```go
func (u UTXOSet) FindSpendableOutputs(pubkeyHash []byte, amount int) (int, map[string][]int) {
    unspentOutputs := make(map[string][]int)
    accumulated := 0
    db := u.Blockchain.db

    err := db.View(func(tx *bolt.Tx) error {
        b := tx.Bucket([]byte(utxoBucket))
        c := b.Cursor()

        for k, v := c.First(); k != nil; k, v = c.Next() {
            txID := hex.EncodeToString(k)
            outs := DeserializeOutputs(v)

            for outIdx, out := range outs.Outputs {
                if out.IsLockedWithKey(pubkeyHash) && accumulated < amount {
                    accumulated += out.Value
                    unspentOutputs[txID] = append(unspentOutputs[txID], outIdx)
                }
            }
        }
    })

    return accumulated, unspentOutputs
}
```

获取余额过程也可以用到UTXO集合：

```go
func (u UTXOSet) FindUTXO(pubKeyHash []byte) []TXOutput {
    var UTXOs []TXOutput
    db := u.Blockchain.db

    err := db.View(func(tx *bolt.Tx) error {
        b := tx.Bucket([]byte(utxoBucket))
        c := b.Cursor()

        for k, v := c.First(); k != nil; k, v = c.Next() {
            outs := DeserializeOutputs(v)

            for _, out := range outs.Outputs {
                if out.IsLockedWithKey(pubKeyHash) {
                    UTXOs = append(UTXOs, out)
                }
            }
        }

        return nil
    })

    return UTXOs
}
```

之前**Blockchain**的那些方法已经不需要了，现在开始使用这些新方法。

UTXO集合的出现意味着交易信息分为两部分存储：实际交易数据存储在blockchain中，UTXO数据存储在UTXO集合中。为了保持数据一致性的同时避免每次挖到一个新的block都重建UTXO集合（不希望频繁扫描blockchain），需要一个数据同步机制：

```go
func (u UTXOSet) Update(block *Block) {
    db := u.Blockchain.db

    err := db.Update(func(tx *bolt.Tx) error {
        b := tx.Bucket([]byte(utxoBucket))

        for _, tx := range block.Transactions {
            if tx.IsCoinbase() == false {
                for _, vin := range tx.Vin {
                    updatedOuts := TXOutputs{}
                    outsBytes := b.Get(vin.Txid)
                    outs := DeserializeOutputs(outsBytes)

                    for outIdx, out := range outs.Outputs {
                        if outIdx != vin.Vout {
                            updatedOuts.Outputs = append(updatedOuts.Outputs, out)
                        }
                    }

                    if len(updatedOuts.Outputs) == 0 {
                        err := b.Delete(vin.Txid)
                    } else {
                        err := b.Put(vin.Txid, updatedOuts.Serialize())
                    }

                }
            }

            newOutputs := TXOutputs{}
            for _, out := range tx.Vout {
                newOutputs.Outputs = append(newOutputs.Outputs, out)
            }

            err := b.Put(tx.ID, newOutputs.Serialize())
        }
    })
}
```

该方法看上去很长，但实际很简单。当新的block生成时，需要更新UTXO集合。更新意味着删除已消费的TXO，同时将新生成的交易中的UTXO添加进来。若某个交易的UTXO全部被删除，该交易也会随之被移除。

```go
func (cli *CLI) createBlockchain(address string) {
    ...
    bc := CreateBlockchain(address)
    defer bc.db.Close()

    UTXOSet := UTXOSet{bc}
    UTXOSet.Reindex()
    ...
}
```

当blockchain被创建时，需要创建UTXO集合。现在，这是唯一需要进行**Reindex**的地方。虽然blockchain一开始仅有一个block以及一个交易，这时仅需要执行**Update**没必要进行**Reindex**，但是将来可能需要这个重建机制。

```go
func (cli *CLI) send(from, to string, amount int) {
    ...
    newBlock := bc.MineBlock(txs)
    UTXOSet.Update(newBlock)
}
```



