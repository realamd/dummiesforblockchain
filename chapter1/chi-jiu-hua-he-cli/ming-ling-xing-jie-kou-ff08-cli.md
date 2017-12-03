目前为止，程序还没有提供任何交互接口，仅仅在main函数中调用**NewBlockchain**, **bc.AddBlock**等函数。现在让我们实现以下命令：

```
blockchain_go addblock "Pay 0.031337 for a coffee"
blockchain_go printchain
```

所有命令行命令通过**CLI**结构处理：

```go
type CLI struct {
	bc *Blockchain
}
```



