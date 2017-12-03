比特币中，TXI关联TXO的逻辑和经典的“先有鸡还是先有蛋”的问题是一样的。比特币是先有蛋（TXO）后有鸡（TXI），TXO先于TXI出现。

当blockchain的首个block（即genesis block）被挖到后，会生成一个coinbase交易。coinbase交易是一种特殊的交易，该TXI不会引用任何TXO，而会直接生成一个TXO，这是作为奖励给矿工的。

Blockchain以genesis block开头，该block生成blockchain中第一个TXO。由于之前没有任何交易，因此该TXI不会与任何TXO关联。

下面创建一个coinbase交易：

```go
func NewCoinbaseTX(to, data string) *Transaction {
	if data == "" {
		data = fmt.Sprintf("Reward to '%s'", to)
	}

	txin := TXInput{[]byte{}, -1, data}
	txout := TXOutput{subsidy, to}
	tx := Transaction{nil, []TXInput{txin}, []TXOutput{txout}}
	tx.SetID()

	return &tx
}
```



