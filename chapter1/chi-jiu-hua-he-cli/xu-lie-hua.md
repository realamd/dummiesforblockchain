若前所述，BoltDB仅仅存储字节数组，因此我们实现Seriable方式（基于encoding/gob库）用于序列化**Block**结构：

```go
func (b *Block) Serialize() []byte {
    var result bytes.Buffer
    encoder := gob.NewEncoder(&result)

    err := encoder.Encode(b)

    return result.Bytes()
}
```

过程很简单：首先声明一个buffer；然后初始化一个gob编码block；最后返回编码后的字节数组。

接下来，我们需要一个反序列化函数，用于将一个字节数组解码为一个**block**：

```go
func DeserializeBlock(d []byte) *Block {
	var block Block

	decoder := gob.NewDecoder(bytes.NewReader(d))
	err := decoder.Decode(&block)

	return &block
}
```



