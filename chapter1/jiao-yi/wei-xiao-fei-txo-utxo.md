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

目前，仅仅将**ScriptSig、ScriptPubKey**与**unlockingData**作比较，待后续实现地址（钱包）后，会做进一步改进。

接下来查找包含UTXO的交易，过程有些复杂：

```go
func (bc *Blockchain) FindUnspentTransactions(address string) []Transaction {
  var unspentTXs []Transaction
  spentTXOs := make(map[string][]int)
  bci := bc.Iterator()

  for {
    block := bci.Next()

    for _, tx := range block.Transactions {
      txID := hex.EncodeToString(tx.ID)

    Outputs:
      for outIdx, out := range tx.Vout {
        // Was the output spent?
        if spentTXOs[txID] != nil {
          for _, spentOut := range spentTXOs[txID] {
            if spentOut == outIdx {
              continue Outputs
            }
          }
        }

        if out.CanBeUnlockedWith(address) {
          unspentTXs = append(unspentTXs, *tx)
        }
      }

      if tx.IsCoinbase() == false {
        for _, in := range tx.Vin {
          if in.CanUnlockOutputWith(address) {
            inTxID := hex.EncodeToString(in.Txid)
            spentTXOs[inTxID] = append(spentTXOs[inTxID], in.Vout)
          }
        }
      }
    }

    if len(block.PrevBlockHash) == 0 {
      break
    }
  }

  return unspentTXs
}
```

交易存储在block中，我们需要遍历blockchain中的每一个block：

```go
if out.CanBeUnlockedWith(address) {
    unspentTXs = append(unspentTXs, tx)
}
```

如果TXO是被指定地址锁定的，该TXO会作为候选TXO继续进行处理：

```go
if spentTXOs[txID] != nil {
    for _, spentOut := range spentTXOs[txID] {
        if spentOut == outIdx {
            continue Outputs
        }
    }
}
```

对于已经被TXI引用的TXO，不做处理；对于未被TXI引用的TXO，即UTXO，将包含其的交易保存到交易列表中。对于coinbase交易，不需要遍历TXI，因为其TXI不会引用任何TXO。

```go
if tx.IsCoinbase() == false {
    for _, in := range tx.Vin {
        if in.CanUnlockOutputWith(address) {
            inTxID := hex.EncodeToString(in.Txid)
            spentTXOs[inTxID] = append(spentTXOs[inTxID], in.Vout)
        }
    }
}
```

最终，该函数返回包含UTXO的交易列表。通过接下来的函数，进一步处理，最终返回TXO列表。

```go
func (bc *Blockchain) FindUTXO(address string) []TXOutput {
       var UTXOs []TXOutput
       unspentTransactions := bc.FindUnspentTransactions(address)

       for _, tx := range unspentTransactions {
               for _, out := range tx.Vout {
                       if out.CanBeUnlockedWith(address) {
                               UTXOs = append(UTXOs, out)
                       }
               }
       }

       return UTXOs
}
```



