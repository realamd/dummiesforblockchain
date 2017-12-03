让我们从“ blockchain”中的“block”开始整个旅程。block存储价值信息，例如，比特币中的block存储交易信息。除此之外，block中包含一些其他信息，如版本、时间戳和上一个Block的hash值等

本文中，我们并不会按照比特币规范实现一个完整的blockchain，而是实现一个简化版本，仅仅包含一些重要的信息。block结构如下所示：

```go
type Block struct {
    Timestamp     int64 
    Data          []byte
    PrevBlockHash []byte
    Hash          []byte
}
```

**Timestamp**表示Block被创建的时间戳，**Data**表示block中实际的价值信息，**PrevBlockHash**表示上一个Block的hash值，**Hash**表示该block本身的Hash值。比特币规范中，Timestamp、PrevBlockHash、Hash存储在独立的数据结构中，作为block的头信息；Data存储在另一个独立的数据结构种。本文中，我们将它们放在一个数据结构中，从而简化实现。

Hash值用于确保blockchain的安全。Hash计算是计算敏感的操作，即使在高性能电脑也需要花费一段时间来完成计算\(这也就是为什么人们购买高性能GPU进行比特币挖矿的原因\)。blockchain架构设计有意使Hash计算变得困难，这样做是为了加大新增一个block的难度，进而防止block在增加后被随意修改。

现在，我们首先**SetHash**方法，将Block中的字段信息组合后，生成该block的SHA-256 Hash值：

```go
func (b *Block) SetHash() {
	timestamp := []byte(strconv.FormatInt(b.Timestamp, 10))
	headers := bytes.Join([][]byte{b.PrevBlockHash, b.Data, timestamp}, []byte{})
	hash := sha256.Sum256(headers)

	b.Hash = hash[:]
}
```



