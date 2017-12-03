TXI结构如下：

```go
type TXInput struct {
    Txid      []byte
    Vout      int
    ScriptSig string
}
```

如上所述，TXI与之前的某个TXO相关联：**Txid**存储输出所属的交易的ID，**Vout**存储输出的序号（一个交易可以包括多个TXO）。**ScriptSig**存储一个脚本，与之关联的TXO的**ScriptPubKey**就来源于该脚本，因此两者可以进行校验：如果校验正确，与之关联的TXO被解锁并生成新的TXO；如果校验不正确，TXO压根不能够被TXI所引用。该机制保证钱只能被其拥有者使用，而不能被其他人使用。

由于我们还没有实现地址（钱包），**ScriptSig**仅仅存储用户自定义的字符串而已，后续我们将实现公钥和签名核查。

综上所述，TXO存储交易额，同时拥有一个解锁脚本。每个交易至少拥有一个TXI和一个TXO。TXI的**ScriptSig**和TXO的**ScriptPubKey**进行校验，验证成功后可以解锁交易输出，并创建新的TXO。

那么问题来了：先有TXI还是先有TXO？

