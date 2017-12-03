若前所述，BoltDB仅仅存储字节数组，因此我们实现Seriable方式（基于encoding/gob库）用于序列化**Block**结构：

```go
func (b *Block) Serialize() []byte {
	var result bytes.Buffer
	encoder := gob.NewEncoder(&result)

	err := encoder.Encode(b)

	return result.Bytes()
}
```



