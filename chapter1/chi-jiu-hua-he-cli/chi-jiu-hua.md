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

让我们一段一段来看实现。

```go
db, err := bolt.Open(dbFile, 0600, nil)
```

上面代码是打开BoltDB文件的标准方式，如果文件不存在不会反馈错误。

```go
err = db.Update(func(tx *bolt.Tx) error {
...
})
```

在BoltDB中，数据库操作都在一个事务中运行。BoltDB有两种事务：只读事务和读写事务。在这里，由于可能会存在添加genesis block的操作，因此我们使用读写事务（**db.Update\(...\)**）。

```go
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
```

上述代码是核心逻辑。首先，我们尝试获取一个bucket，如果bucket存在，则将tip设置为key\(**l**\)的值；如果bucket不存在，则创建genesis block、创建bucket、存储genesis block、将tip设置为genesis block的hash值。

此外，注意创建blockchain的方法有了变化：

```go
bc := Blockchain{tip, db}
```

现在不再存储任何block，而仅仅存储blockchain的tip，同时为了避免程序运行过程中数据库反复被打开，因此blockchain中会存储已打开的数据库链接，**blockchain**的结构修改如下：

```go
type Blockchain struct {
    tip []byte
    db  *bolt.DB
}
```

接下来，修改**AddBlock**方法：新增一个block会变得复杂一些：

```go
func (bc *Blockchain) AddBlock(data string) {
    var lastHash []byte

    err := bc.db.View(func(tx *bolt.Tx) error {
        b := tx.Bucket([]byte(blocksBucket))
        lastHash = b.Get([]byte("l"))

        return nil
    })

    newBlock := NewBlock(data, lastHash)

    err = bc.db.Update(func(tx *bolt.Tx) error {
        b := tx.Bucket([]byte(blocksBucket))
        err := b.Put(newBlock.Hash, newBlock.Serialize())
        err = b.Put([]byte("l"), newBlock.Hash)
        bc.tip = newBlock.Hash

        return nil
    })
}
```

我们来一段一段来看：

```go
err := bc.db.View(func(tx *bolt.Tx) error {
    b := tx.Bucket([]byte(blocksBucket))
    lastHash = b.Get([]byte("l"))

    return nil
})
```

首先使用只读事务获取当前数据库中最新block的hash值。

```go
newBlock := NewBlock(data, lastHash)
b := tx.Bucket([]byte(blocksBucket))
err := b.Put(newBlock.Hash, newBlock.Serialize())
err = b.Put([]byte("l"), newBlock.Hash)
bc.tip = newBlock.Hash
```

在挖到一个新block后，我们将序列化后的结果存储到数据库文件中同时将key\(**l**\)值更新为最新block的hash值。

完成！一切简单明了。



