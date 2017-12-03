现在，让我们来实现blockchain吧。本质上，blockchain仅仅是具备某种特殊结构的数据库：有序，反向链接链表。这意味着，block按照插入的顺序存放，同时每个block都保存指向上一个block的链接。这种结构保证可以快速获取最新插入的block同时获取它的hash值。

Go语言中，可以通过slice和map来实现该结构。slice\(有序\)用于保存有序的hash值，map\(无序\)用于保存hash-&gt;block对。对于我们的blockchain原型，目前不需要根据hash值来获取block，因此仅仅使用slice即可满足需求。blockchain结构如下：

```go
type Blockchain struct {
	blocks []*Block
}
```

这就是我们的Blockchain，怎么样够简单吧😉

接下来，我们事先AddBlock方法，用于将block添加到blockchain中：

```go
func (bc *Blockchain) AddBlock(data string) {
	prevBlock := bc.blocks[len(bc.blocks)-1]
	newBlock := NewBlock(data, prevBlock.Hash)
	bc.blocks = append(bc.blocks, newBlock)
}
```

新增一个block的前提是另一个block已经存在，但是一开始blockchain中并没有任何block。因此，在任何blockchain中都必须有一个特殊的block存在，称之为**GenesisBlock。**下面实现**NewGenesisBlock**方法用于创建**GenesisBlock**：

```go
func NewGenesisBlock() *Block {
	return NewBlock("Genesis Block", []byte{})
}
```

接下来，实现**NewBlockchain**方法，该方法会创建一个包含Genesis Block的blockchain：

```go
func NewBlockchain() *Blockchain {
	return &Blockchain{[]*Block{NewGenesisBlock()}}
}
```

最后让我们看看Blockchain是否可以正常工作吧：

```go
func main() {
	bc := NewBlockchain()

	bc.AddBlock("Send 1 BTC to Ivan")
	bc.AddBlock("Send 2 more BTC to Ivan")

	for _, block := range bc.blocks {
		fmt.Printf("Prev. hash: %x\n", block.PrevBlockHash)
		fmt.Printf("Data: %s\n", block.Data)
		fmt.Printf("Hash: %x\n", block.Hash)
		fmt.Println()
	}
}
```

输出：

```
Prev. hash:
Data: Genesis Block
Hash: aff955a50dc6cd2abfe81b8849eab15f99ed1dc333d38487024223b5fe0f1168

Prev. hash: aff955a50dc6cd2abfe81b8849eab15f99ed1dc333d38487024223b5fe0f1168
Data: Send 1 BTC to Ivan
Hash: d75ce22a840abb9b4e8fc3b60767c4ba3f46a0432d3ea15b71aef9fde6a314e1

Prev. hash: d75ce22a840abb9b4e8fc3b60767c4ba3f46a0432d3ea15b71aef9fde6a314e1
Data: Send 2 more BTC to Ivan
Hash: 561237522bb7fcfbccbc6fe0e98bbbde7427ffe01c6fb223f7562288ca2295d1
```

大功告成！

