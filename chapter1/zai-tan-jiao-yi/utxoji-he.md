在[持久化和CLI](/chapter1/chi-jiu-hua-he-cli.md)中，曾经提到比特币将block存储在数据库：block存储在**blocks**数据库；交易TXO存储在**chainstate**数据库。现在重新回顾下**chainstate**的结构：

1. **'c' + 32-byte transaction hash -&gt; unspent transaction output record for that transaction**
2. **'B' -&gt; 32-byte block hash: the block hash up to which the database represents the unspent transaction outputs**

在此之前，我们实现了交易，但是没有将交易信息存储到**chainstate**数据库。现在，我们来实现此机制。

**chainstate**数据库不存储交易本身，而是存储所有UTXO，称为UTXO集合。此外，**chainstate**数据库也会存储“the block hash up to which the database represents the unspent transaction outputs”，不过目前用不到block高度的的信息，故暂不实现。

那么，UTXO集合有什么用呢？

让我们回顾下**FindUnspentTransactions**方法：

