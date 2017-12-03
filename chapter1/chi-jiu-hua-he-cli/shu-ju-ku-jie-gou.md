实现持久化逻辑前，需要先确定在数据库中如何存储数据。对此，我们计划参考比特币的实现。

简单来说，比特币使用下面两个bucket来存储数据：

1. **blocks**存储用于描述所有block的元数据信息。
2. **chainstate**用于存储blockchain的状态，包括所有交易元数据和部分元数据。

同时，出于性能考虑，每个block存放在独立的文件，这样获取单个block时不需要加载所有block。但为了简化，我们还是把所有数据存储在一个文件中。

在blocks bucket中，存储如下k-v对：

1. 'b' + 32-byte block hash -&gt; block index record
2. 'f' + 4-byte file number -&gt; file information record
3. 'l' -&gt; 4-byte file number: the last block file number used
4. 'R' -&gt; 1-byte boolean: whether we're in the process of reindexing
5. 'F' + 1-byte flag name length + flag name string -&gt; 1 byte boolean: various flags that can be on or off
6. 't' + 32-byte transaction hash -&gt; transaction index record

在chainstate bucket中，存储如下k-v对：

1. 'c' + 32-byte transaction hash -&gt; unspent transaction output record for that transaction
2. 'B' -&gt; 32-byte block hash: the block hash up to which the database represents the unspent transaction outputs

（设计原理请参考[这里](https://en)）

由于目前实现还没有设计交易，因此我们仅需要实现**blocks** bucket。同时，就如上面所说，所有数据存储在一个单一的文件中，因此不需要任何和文件号相关的信心。所以，我们仅需要一下k-v数据：

1. 32-byte block-hash -&gt; Block structure \(serialized\)
2. 'l' -&gt; the hash of the last block in a chain

这就是我们实现持久化所需的相关信息。

