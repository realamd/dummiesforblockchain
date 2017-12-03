  
OK，纸上谈兵到此为止，接下来实现该机制。首先，让我们来定义挖矿的难度：

```go
const targetBits = 24
```

比特币中，“target bits”在block头信息中，表示该block被挖到的难度。我们并不实现难度自调整算法，而是定义一个全局的难度。

“target bits”只要小于256（因为我们要使用sha256算法计算hash值）即可，我们设置为24。这意味着，当hash值的前24位均为0即满足要求。当“target bits”过大时，满足要求将非常困难，因为符合条件的数据总数量变得非常小，因此需要更长时间计算才有可能得到满足要求的hash值。“target bits”的值越大难度越高，值越小难度越低。

```go
type ProofOfWork struct {
	block  *Block
	target *big.Int
}

func NewProofOfWork(b *Block) *ProofOfWork {
	target := big.NewInt(1)
	target.Lsh(target, uint(256-targetBits))

	pow := &ProofOfWork{b, target}

	return pow
}
```



