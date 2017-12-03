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



