需要找到所有未消费的TXO（下文称为UTXO）。未消费表示这些TXO未被任何TXI所引用，在上图中一下TXO是UTXO：

> 1. tx0, output 1;
> 2. tx1, output 0;
> 3. tx3, output 0;
> 4. tx4, output 0.

下面是加锁/解锁方法：

```go
func (in *TXInput) CanUnlockOutputWith(unlockingData string) bool {
	return in.ScriptSig == unlockingData
}

func (out *TXOutput) CanBeUnlockedWith(unlockingData string) bool {
	return out.ScriptPubKey == unlockingData
}
```



