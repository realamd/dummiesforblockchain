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

coinbase仅有一个TXI，该TXI的**Txid**为空，**Vout**设置为-1，同时**ScriptSig**中存储的不是脚本，而仅仅是一个普通字符串。

> _比特币中，第一个coinbase交易包含如下信息“The Times 03/Jan/2009 Chancellor on brink of second bailout for banks”_

**Subsidy**是挖矿的奖励值，比特币中，该奖励值是基于总block数量计算得到的。挖出genesis奖励50BTC，每挖出**210000**个block，奖励值减半。我们的实现中，该奖励值是一个常量。

