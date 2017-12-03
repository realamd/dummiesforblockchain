一个交易由多个输入和输出：

```go
type Transaction struct {
    ID   []byte
    Vin  []TXInput
    Vout []TXOutput
}
```

交易输入（下文称TXI）均与之前交易的输出（下文称TXO）相关联（存在一个例外情况，后续讨论）；TXO存储交易额。下图展示了交易间相互关联的情况：![](/assets/4.1.png)

需注意的是：

> 1. 有些TXO没有与任何其他TXI相关联
> 2. 某个TXI可能与之前多个TXO相关联
> 3. 一个TXI仅与一个TXO相关联

本文中，我们将使用“money”, “coins”, “spend”, “send”, “account”等术语用于描述交易过程，但比特币中就没有相关概念存在的。比特币通过一个脚本对交易额进行加锁，而仅仅加锁者才能解锁。

一个交易包括付款方和收款方，因此交易中的TXI都是付款方，而交易中的TXO有两种情况：**若TXI的总额正好和所需值相等，那么交易只有一个TXO，该TXO属于收款方；若TXI的总额大于所需值，那么交易有两个TXO，一个属于收款方，一个属于付款方。**（FindUnspentTransactions方法利用这个特点来获取某个地址（账户）的UTXO）：

![](/assets/4.2.png)![](/assets/4.3.png)



