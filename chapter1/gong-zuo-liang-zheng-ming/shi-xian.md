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

我们将**nonce**将入**Block**结构，这是因为**nonce**需要用于后续验证操作。**Block**结构结构如下：

```go
type Block struct {
    Timestamp     int64
    Data          []byte
    PrevBlockHash []byte
    Hash          []byte
    Nonce         int
}
```

大功告成！让我们运行看看效果吧：

```
Mining the block containing "Genesis Block"
00000041662c5fc2883535dc19ba8a33ac993b535da9899e593ff98e1eda56a1

Mining the block containing "Send 1 BTC to Ivan"
00000077a856e697c69833d9effb6bdad54c730a98d674f73c0b30020cc82804

Mining the block containing "Send 2 more BTC to Ivan"
000000b33185e927c9a989cc7d5aaaed739c56dad9fd9361dea558b9bfaf5fbe

Prev. hash:
Data: Genesis Block
Hash: 00000041662c5fc2883535dc19ba8a33ac993b535da9899e593ff98e1eda56a1

Prev. hash: 00000041662c5fc2883535dc19ba8a33ac993b535da9899e593ff98e1eda56a1
Data: Send 1 BTC to Ivan
Hash: 00000077a856e697c69833d9effb6bdad54c730a98d674f73c0b30020cc82804

Prev. hash: 00000077a856e697c69833d9effb6bdad54c730a98d674f73c0b30020cc82804
Data: Send 2 more BTC to Ivan
Hash: 000000b33185e927c9a989cc7d5aaaed739c56dad9fd9361dea558b9bfaf5fbe
```

可以看到，所有block的hash值前3个字节均为0，同时生成block时需要花费一定时间。

目前还有一件事需要做，就是对工作量进行验证：

```go
func (pow *ProofOfWork) Validate() bool {
    var hashInt big.Int

    data := pow.prepareData(pow.block.Nonce)
    hash := sha256.Sum256(data)
    hashInt.SetBytes(hash[:])

    isValid := hashInt.Cmp(pow.target) == -1

    return isValid
}
```

可以看到，验证过程需要nonce数据。

再运行一次，一切OK：

```
func main() {
    ...

    for _, block := range bc.blocks {
        ...
        pow := NewProofOfWork(block)
        fmt.Printf("PoW: %s\n", strconv.FormatBool(pow.Validate()))
        fmt.Println()
    }
}
```

输出：

```
...

Prev. hash:
Data: Genesis Block
Hash: 00000093253acb814afb942e652a84a8f245069a67b5eaa709df8ac612075038
PoW: true

Prev. hash: 00000093253acb814afb942e652a84a8f245069a67b5eaa709df8ac612075038
Data: Send 1 BTC to Ivan
Hash: 0000003eeb3743ee42020e4a15262fd110a72823d804ce8e49643b5fd9d1062b
PoW: true

Prev. hash: 0000003eeb3743ee42020e4a15262fd110a72823d804ce8e49643b5fd9d1062b
Data: Send 2 more BTC to Ivan
Hash: 000000e42afddf57a3daa11b43b2e0923f23e894f96d1f24bfd9b8d2d494c57a
PoW: true
```



