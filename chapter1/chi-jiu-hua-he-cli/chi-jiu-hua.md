让我们来实现 **NewBlockchain**函数，该函数创建一个blockchain实例同时添加genesis block，步骤如下：

> 1. 打开数据库文件
> 2. 检查是否有blockchain
> 3. 如果有blockchain
>    1. 创建一个blockchain实例
>    2. 将blockchain的tip指向最新的block
> 4. 如果没有blockchain
>    1. 创建genesis block
>    2. 存储在数据库中
>    3. 将genesis block的hash值保存为最新的block hash值
>    4. 创建一个新的blockchain实例，同时tip指向最新的block

代码如下：

```go
func NewBlockchain() *Blockchain {
	var tip []byte
	db, err := bolt.Open(dbFile, 0600, nil)

	err = db.Update(func(tx *bolt.Tx) error {
		b := tx.Bucket([]byte(blocksBucket))

		if b == nil {
			genesis := NewGenesisBlock()
			b, err := tx.CreateBucket([]byte(blocksBucket))
			err = b.Put(genesis.Hash, genesis.Serialize())
			err = b.Put([]byte("l"), genesis.Hash)
			tip = genesis.Hash
		} else {
			tip = b.Get([]byte("l"))
		}

		return nil
	})

	bc := Blockchain{tip, db}

	return &bc
}
```



