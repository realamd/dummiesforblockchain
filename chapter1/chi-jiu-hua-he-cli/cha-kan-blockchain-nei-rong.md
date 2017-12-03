现在所有block都保存在数据库文件中，因此我们可以重新打开一个blockchain并且新增一个block。但我们现在失去一个很好的特性：我们不能输出blockchain的内容。让我们修改这个缺陷！

BoltDB允许遍历bucket中的所有key，但是key按照字节序的顺序来存储的。我们想按照block被加入的顺序来打印blockchain内容。此外，我们不想将所有block全部加载到内存，因此将实现一个迭代器来逐个遍历block：

```go
type BlockchainIterator struct {
	currentHash []byte
	db          *bolt.DB
}
```

当想遍历blockchain时，我们会创建一个迭代器。迭代器包括当前遍历到的block的hash值和数据库链接。

```go
func (bc *Blockchain) Iterator() *BlockchainIterator {
	bci := &BlockchainIterator{bc.tip, bc.db}

	return bci
}
```

一开始迭代器指向blockchain的tip，意味着将从顶到底、从新到旧的遍历blockchain。实际上，选择tip就好比blockchain投票。Blockchain可能有多个分支，最长的那个分支被当做主分支。获得tip后，就可以重建整个blockchain并且得到blockchain的长度。因此，某种程度上可以说tip是blockchain的标识。

**BlockchainIterator**仅仅做一件事：返回先一个block。

```go
func (i *BlockchainIterator) Next() *Block {
	var block *Block

	err := i.db.View(func(tx *bolt.Tx) error {
		b := tx.Bucket([]byte(blocksBucket))
		encodedBlock := b.Get(i.currentHash)
		block = DeserializeBlock(encodedBlock)

		return nil
	})

	i.currentHash = block.PrevBlockHash

	return block
}
```



