从现在开始，每个block至少包含一个交易，**Block**结构中的**Data**字段江永**Transactions**字段代替。

```go
type Block struct {
    Timestamp     int64
    Transactions  []*Transaction
    PrevBlockHash []byte
    Hash          []byte
    Nonce         int
}
```

**NewBlock**和**NewGenesisBlock**方法也需要修改：

```go
func NewBlock(transactions []*Transaction, prevBlockHash []byte) *Block {
    block := &Block{time.Now().Unix(), transactions, prevBlockHash, []byte{}, 0}
    ...
}

func NewGenesisBlock(coinbase *Transaction) *Block {
    return NewBlock([]*Transaction{coinbase}, []byte{})
}
```

**CreateBlockchain**也需要修改：

```go
func CreateBlockchain(address string) *Blockchain {
	...
	err = db.Update(func(tx *bolt.Tx) error {
		cbtx := NewCoinbaseTX(address, genesisCoinbaseData)
		genesis := NewGenesisBlock(cbtx)

		b, err := tx.CreateBucket([]byte(blocksBucket))
		err = b.Put(genesis.Hash, genesis.Serialize())
		...
	})
	...
}
```



