一个交易由多个输入和输出：

```go
type Transaction struct {
    ID   []byte
    Vin  []TXInput
    Vout []TXOutput
}
```

交易输入（下文称TXI）均与之前交易的输出（下文称TXO）相关联（存在一个例外情况，后续讨论）；TXO存储交易额。下图展示了交易间相互关联的情况：



