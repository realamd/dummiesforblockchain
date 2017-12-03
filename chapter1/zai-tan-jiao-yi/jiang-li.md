前文中，我们没有讨论挖矿奖励。现在让我们来实现此机制。

所谓的奖励其实是一个coinbase交易。当矿工挖到一个新的block时，除了一个通用交易外，还会包含一个coinbase交易。Coinbase的TXO中包括矿工的公钥。

修改**send**方法，实现挖矿奖励：

```go
func (cli *CLI) send(from, to string, amount int) {
    ...
    bc := NewBlockchain()
    UTXOSet := UTXOSet{bc}
    defer bc.db.Close()

    tx := NewUTXOTransaction(from, to, amount, &UTXOSet)
    cbTx := NewCoinbaseTX(from, "")
    txs := []*Transaction{cbTx, tx}

    newBlock := bc.MineBlock(txs)
    fmt.Println("Success!")
}
```

我们的实现中，交易创建者进行挖矿，挖到一个新block时，就会收到一个奖励。

