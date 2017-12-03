在[基本原型](/chapter1/ji-ben-yuan-xing.md)章节中，我们利用非常简单的数据结构构建了blockchain数据库。Block之间存在链式关系：每个block都保存上一个block的链接。但是，我们的blockchain实现存在一个严重的问题：添加block操作过于简单。而区块链和比特币系统的主旨是让新增block成为一个高成本的操作。接下来，我们将修改这个问题。

