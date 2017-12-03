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

**ProofOfWork**结构包括一个block指针和一个target指针。“target”表示困难度，其类型为big.Int，之所以使用big结构是为了方便和hash值做比较检查是否满足要求。

**NewProofOfWork**方法中，将target初始化为1，然后将其左移（256-target），其值为：

```
0x10000000000000000000000000000000000000000000000000000000000
```

该值在内存中占用29字节，接下来将其与之前示例中的hash值进行比较：

```
0fac49161af82ed938add1d8725835cc123a1a87b1b196488360e58d4bfb51e3
0000010000000000000000000000000000000000000000000000000000000000
0000008b0f41ec78bab747864db66bcb9fb89920ee75f43fdaaeb5544f7f76ca
```

上述有3个hash值，第1行“I like donuts”计算得到hash值，第2行是目标值，第3行是基于“I like donutsca07ca”计算得到的hash值。第1行值大于目标值，不满足要求；第3行值小于目标值，满足要求，结束证明。

总结一下，hash值满足要求的规则：当hash值大于目标值，表示不满足要求；当hash值小于等于目标值，表示满足要求。

prepareData方法将block各个字段和nonce（counter值）作为输入，计算得到hash值作为输出：

```go
func (pow *ProofOfWork) prepareData(nonce int) []byte {
    data := bytes.Join(
        [][]byte{
            pow.block.PrevBlockHash,
            pow.block.Data,
            IntToHex(pow.block.Timestamp),
            IntToHex(int64(targetBits)),
            IntToHex(int64(nonce)),
        },
        []byte{},
    )

    return data
}
```

准备工作一切完毕，接下来实现PoW核心算法：

```go
func (pow *ProofOfWork) Run() (int, []byte) {
    var hashInt big.Int
    var hash [32]byte
    nonce := 0

    fmt.Printf("Mining the block containing \"%s\"\n", pow.block.Data)
    for nonce < maxNonce {
        data := pow.prepareData(nonce)
        hash = sha256.Sum256(data)
        fmt.Printf("\r%x", hash)
        hashInt.SetBytes(hash[:])

        if hashInt.Cmp(pow.target) == -1 {
            break
        } else {
            nonce++
        }
    }
    fmt.Print("\n\n")

    return nonce, hash[:]
}
```

最初，初始化变量：**hashInt**是hash值得整数形式；**nonce**是一个递增计数器，初始化为0。接下来，开始进行循环计算， **nonce**和**maxNonce（**设置为math.MaxInt64）进行比较，直到找到符合要求的hash值为止。

循环中进行如下操作：

> 1. 准备数据
> 2. 使用SHA-256算法计算hash值
> 3. 将hash值转换为整数
> 4. 整型hash值与目标值作比较

接下来，删除**SetHash**方法，并修改**NewBlock**方法：

```go
func NewBlock(data string, prevBlockHash []byte) *Block {
	block := &Block{time.Now().Unix(), []byte(data), prevBlockHash, []byte{}, 0}
	pow := NewProofOfWork(block)
	nonce, hash := pow.Run()

	block.Hash = hash[:]
	block.Nonce = nonce

	return block
}
```



