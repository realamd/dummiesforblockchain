比特币拥有一个称作script的脚本语言。通过script，可以对TXO加锁，同时支持TXI解锁TXO。Script非常简单，就是一些代码和操作符的序列，例如：

```
5 2 OP_ADD 7 OP_EQUAL
```

**5, 2, 7**是数据，**OP\_ADD**和**OP\_EQUAL**是操作符。script从左至右执行代码：遇到数据放入堆栈（先进后出）；遇到操作符就从堆栈顶部取出所需的数据，并将结果放入堆栈。

上面的代码执行过程如下：![](/assets/6.2.png)  
**OP\_ADD**从堆栈中获取两个数据然后求和，将结果放入堆栈中。**OP\_EQUAL**从堆栈获取两个数据进行比较：如果相等将**true**放入堆栈；否则将**false**放入堆栈。栈顶数据就是代码的执行结果。

比特币中用于执行交付的脚本：

```
<signature> <pubKey> OP_DUP OP_HASH160 <pubKeyHash> OP_EQUALVERIFY OP_CHECKSIG
```

该脚本是比特币种最常用的脚本，称为P2PKH（Pay to Public Key Hash）。此脚本用于向一个公钥hash值进行支付，例如使用一个公钥对货币进行加锁。比特币支付的核心就在于此：无账户，无基金转移；仅有一个脚本用于检查所提供的签名和公钥是否正确。

此脚本数据分为两部分：

> 1. 第一部分，&lt;signature&gt; &lt;pubKey&gt;，被存放在TXI的ScriptSig 中
> 2. 第二部分，OP\_DUP OP\_HASH160 &lt;pubKeyHash&gt; OP\_EQUALVERIFY OP\_CHECKSIG，被存放在TXO的ScriptPubKey中

TXO定义解锁逻辑，TXI提供数据解锁TXO，脚本执行如下：

> 1. Stack: **empty**
>     Script: **&lt;signature&gt; &lt;pubKey&gt; OP\_DUP OP\_HASH160 &lt;pubKeyHash&gt; OP\_EQUALVERIFY OP\_CHECKSIG**
> 2. Stack: **&lt;signature&gt;**
>     Script:** &lt;pubKey&gt; OP\_DUP OP\_HASH160 &lt;pubKeyHash&gt; OP\_EQUALVERIFY OP\_CHECKSIG**
> 3. Stack: **&lt;signature&gt; &lt;pubKey&gt;**
>     Script: **OP\_DUP OP\_HASH160 &lt;pubKeyHash&gt; OP\_EQUALVERIFY OP\_CHECKSIG**
> 4. Stack: **&lt;signature&gt; &lt;pubKey&gt; &lt;pubKey&gt;**
>     Script: **OP\_HASH160 &lt;pubKeyHash&gt; OP\_EQUALVERIFY OP\_CHECKSIG**
> 5. Stack: **&lt;signature&gt; &lt;pubKey&gt; &lt;pubKeyHash&gt;**
>     Script: **&lt;pubKeyHash&gt; OP\_EQUALVERIFY OP\_CHECKSIG**
> 6. Stack: **&lt;signature&gt; &lt;pubKey&gt; &lt;pubKeyHash&gt; &lt;pubKeyHash&gt;**
>     Script: **OP\_EQUALVERIFY OP\_CHECKSIG**
> 7. Stack: **&lt;signature&gt; &lt;pubKey&gt;**
>     Script: **OP\_CHECKSIG**
> 8. Stack: **true or false. Script: empty.**

**OP\_DUP**复制复制栈顶数据，**OP\_HASH160**获取栈顶数据、使用**RIPEMD160**生成hash值并放入堆栈。**OP\_EQUALVERIFY**比较两个栈顶数据，不相等则结束脚本。**OP\_CHECKSIG**对交易进行哈希处理并使用**&lt;signature&gt;、&lt;pubKey&gt;**验证签名。

正因为有了script脚本语言，比特币可以成为智能合约平台，如该脚本可以支持多种支付模式，不仅仅是单个数字了。

