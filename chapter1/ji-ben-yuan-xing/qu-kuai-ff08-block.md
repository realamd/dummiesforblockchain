让我们从“ blockchain”中的“block”开始整个旅程。block存储价值信息，例如，比特币中的block存储交易信息。除此之外，block中包含一些其他信息，如版本、时间戳和上一个Block的hash值等

本文中，我们并不会按照比特币规范实现一个完整的blockchain，而是实现一个简化版本，仅仅包含一些重要的信息。block结构如下所示：

```
type Block struct {
    Timestamp     int64 
    Data          []byte
    PrevBlockHash []byte
    Hash          []byte
}
```



