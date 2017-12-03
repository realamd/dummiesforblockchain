为了保证blockchain的一致性和可靠性，PoW算法必须要考虑block中的交易信息，**ProofOfWork.prepareData**需要做如下修改：

```go
func (pow *ProofOfWork) prepareData(nonce int) []byte {
    data := bytes.Join(
        [][]byte{
            pow.block.PrevBlockHash,
            pow.block.HashTransactions(), // This line was changed
            IntToHex(pow.block.Timestamp),
            IntToHex(int64(targetBits)),
            IntToHex(int64(nonce)),
        },
        []byte{},
    )

    return data
}
```

现在使用**pow.block.HashTransactions\(\)**代替**pow.block.Data**：

```go
func (b *Block) HashTransactions() []byte {
    var txHashes [][]byte
    var txHash [32]byte

    for _, tx := range b.Transactions {
        txHashes = append(txHashes, tx.ID)
    }
    txHash = sha256.Sum256(bytes.Join(txHashes, []byte{}))

    return txHash[:]
}
```

我们遍历所有交易，获取每个交易的hash值，然后将所有交易的hash值组合后最终得到一个代表block中整个交易的hash值，计算整个block的hash值时会引用该值。

> _比特币使用了一种更加优雅的技术：以_[_Merkle Tree（默克尔树）_](%5Cl%20%22_Merkle_Tree)_表示block中的所有交易，PoW算法中使用树根的hash值。该方法仅仅需要树根hash值而不用下载所有交易信息，因此可以非常快速检查block中是否包含某些交易，_

使用Ivan创建一个blockchain：

```
$ blockchain_go createblockchain -address Ivan
00000093450837f8b52b78c25f8163bb6137caf43ff4d9a01d1b731fa8ddcc8a

Done!
```

我们收到了首次挖矿的奖励，但是我们如何知道余额信息呢？

