比特币使用[Hashcash](https://en.wikipedia.org/wiki/Hashcash)算法进行工作量证明，该算法一开始用于反垃圾邮件。该算法分为以下步骤：

> 1. 获取某种公开数据data（在反垃圾邮件场景下，使用收件人地址；在比特币中，使用block头信息）
> 2. 使用一个计数器counter，初始化为0
> 3. 计算data+counter的hash值
> 4. 检查hash值是否满足某种要求
>    1. 满足，结束
>    2. 不满足，counter加1，然后重复3-4步骤

这是个暴力型算法：改变counter值，计算得到新hash值，判断该值是否满足要求，不满足，再改变counter值，再计算，再检查…如此反复尝试直到得到满足要求为止，要求很高的算力。

来我们看看hash要满足什么要求，hashcash最初实现中，hash值要满足“头20个bit全部为0”的要求。比特币设计中，hash值要求是动态变化的，随着时间和矿工的增多，算力要求也越来越多。

为了展示算法，使用上面示例中的数据（“I like donuts”），不停的计算该数据+counter的hash值，直到找到一个满足要求（“前3个字节为0”）的hash值为止，如下图所示：![](/assets/2.2.png)

“I like donutsca07ca”中的**ca07ca**是十六机制格式的值，即counter从0递增到13240266时计算得到满足要求的hash值。

