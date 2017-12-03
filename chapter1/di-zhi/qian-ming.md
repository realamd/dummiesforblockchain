对交易进行签名是保证某人只能花费属于自己货币的唯一方法。签名无效代表交易无效，该交易就无法被加入到blockchain中。

对于实现交易签名，万事俱备只欠东风：基于什么数据进行签名。是基于部分交易进行签名还是对整个交易进行签名？选择数据非常重要。签名的数据中必须包含可以唯一标识该数据的信息。例如，仅仅对TXO的交易额进行签名，这就不合理，因为签名中没有考虑发送者和接受者信息。

交易会解锁之前的TXO，生成新的TXO，因此必须将对以下数据进行签名：

> 1. 被解锁的TXO的公钥hash值。该值标识“发送者”
> 2. 新生成的，被加锁的TXO的公钥hash值。该值标识“接收者”
> 3. 新TXO的交易额
>
> 比特币的加锁、解锁逻辑存放在脚本中，分别存储在TXI的ScriptSig和TXO的ScriptPubKey字段。由于比特币支持多种类型的脚本，因此ScriptPubKey的整体也需要被签名

我们并不需要对TXI中的公钥进行签名。因此并非对“原始”交易进行签名，而是对TrimmedCopy交易进行签名。什么是TrimmedCopy交易？还是让我们来看代码理解下，从**Sign**方法开始：

> _TrimmedCopy交易的处理过程参见_[_此文_](https://en.bitcoin.it/wiki/File:Bitcoin_OpCheckSig_InDetai)_，可能有些过程，但是还是可以看出些端倪的。_

```go
func (tx *Transaction) Sign(privKey ecdsa.PrivateKey, prevTXs map[string]Transaction) {
    if tx.IsCoinbase() {
        return
    }

    txCopy := tx.TrimmedCopy()

    for inID, vin := range txCopy.Vin {
        prevTx := prevTXs[hex.EncodeToString(vin.Txid)]
        txCopy.Vin[inID].Signature = nil
        txCopy.Vin[inID].PubKey = prevTx.Vout[vin.Vout].PubKeyHash
        txCopy.ID = txCopy.Hash()
        txCopy.Vin[inID].PubKey = nil

        r, s, err := ecdsa.Sign(rand.Reader, &privKey, txCopy.ID)
        signature := append(r.Bytes(), s.Bytes()...)

        tx.Vin[inID].Signature = signature
    }
}
```

该方法接受两个参数：私钥以及所引用的之前交易的集合。对交易进行签名时，需要获取该交易所有TXI引用的TXO列表，因此需要存储这些TXO对应的交易。

我们一步一步来看：

```go
if tx.IsCoinbase() {
    return
}
```

Coinbase交易没有真实的TXI，因此此交易不进行签名。

```
txCopy := tx.TrimmedCopy()
```

基于TrimmedCopy交易进行签名：

```go
func (tx *Transaction) TrimmedCopy() Transaction {
    var inputs []TXInput
    var outputs []TXOutput

    for _, vin := range tx.Vin {
        inputs = append(inputs, TXInput{vin.Txid, vin.Vout, nil, nil})
    }

    for _, vout := range tx.Vout {
        outputs = append(outputs, TXOutput{vout.Value, vout.PubKeyHash})
    }

    txCopy := Transaction{tx.ID, inputs, outputs}

    return txCopy
}
```

TrimmedCopy交易包括复制原始交易中所有的TXI和TXO，区别在于TrimmedCopy中TXI的**Signature**和**PubKey**被设置为nil。

接下来对TrimmedCopy交易中所有的TXI进行遍历：

```go
for inID, vin := range txCopy.Vin {
    prevTx := prevTXs[hex.EncodeToString(vin.Txid)]
    txCopy.Vin[inID].Signature = nil
    txCopy.Vin[inID].PubKey = prevTx.Vout[vin.Vout].PubKeyHash
```

对于每个TXI，为了确保万无一失，再次将**Signature**字段设置为nil，同时将**PubKey**设置为其所引用的TXO的**PubKeyHash**。此时，除了当前被处理的交易外，其他所有交易都是“空”交易（**Signature**和**PubKey**字段为nil）。由于比特币允许交易包含来自于不同地址的TXI，因此需要**单独地对每个TXI进行签名**，虽然对于我们的应用来说没必要这么做（在我们的应用中，一个交易包中的TXI均来自同一地址）。

```go
    txCopy.ID = txCopy.Hash()
    txCopy.Vin[inID].PubKey = nil
```

**Hash**方法将交易序列化并通过SHA-256进行哈希。生成的哈希结果就是待签名的数据。此后，我们将**PubKey**字段重新设置为nil避免影响后续的迭代。下面是核心部分：

```go
	r, s, err := ecdsa.Sign(rand.Reader, &privKey, txCopy.ID)
	signature := append(r.Bytes(), s.Bytes()...)

	tx.Vin[inID].Signature = signature
```



