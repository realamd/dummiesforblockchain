**Wallet**结构如下：

```go
type Wallet struct {
    PrivateKey ecdsa.PrivateKey
    PublicKey  []byte
}

type Wallets struct {
    Wallets map[string]*Wallet
}

func NewWallet() *Wallet {
    private, public := newKeyPair()
    wallet := Wallet{private, public}

    return &wallet
}

func newKeyPair() (ecdsa.PrivateKey, []byte) {
    curve := elliptic.P256()
    private, err := ecdsa.GenerateKey(curve, rand.Reader)
    pubKey := append(private.PublicKey.X.Bytes(), private.PublicKey.Y.Bytes()...)

    return *private, pubKey
}
```

钱包就是一个密钥对。**Wallets**代表钱包集合，**Wallets**可以被存储到文件中，同时也可以从文件加载到内存中。NewWallet函数用于生成一个新的钱包。**newKeyPair**函数使用椭圆曲线算法生成私钥，紧接着通过私钥生成公钥。需要注意一点：椭圆曲线算法中，公钥是曲线上的点集合。因此，公钥由X,Y坐标混合而成。比特币中，这些坐标被组合在一起，生成公钥。

现在，创建一个地址：

```go
func (w Wallet) GetAddress() []byte {
    pubKeyHash := HashPubKey(w.PublicKey)

    versionedPayload := append([]byte{version}, pubKeyHash...)
    checksum := checksum(versionedPayload)

    fullPayload := append(versionedPayload, checksum...)
    address := Base58Encode(fullPayload)

    return address
}

func HashPubKey(pubKey []byte) []byte {
    publicSHA256 := sha256.Sum256(pubKey)

    RIPEMD160Hasher := ripemd160.New()
    _, err := RIPEMD160Hasher.Write(publicSHA256[:])
    publicRIPEMD160 := RIPEMD160Hasher.Sum(nil)

    return publicRIPEMD160
}

func checksum(payload []byte) []byte {
    firstSHA := sha256.Sum256(payload)
    secondSHA := sha256.Sum256(firstSHA[:])

    return secondSHA[:addressChecksumLen]
}
```

通过Base58将公钥转换成可读格式，包括以下步骤：

> 1. 使用 RIPEMD160\(SHA256\(PubKey\)\)对公钥进行两次哈希，生成pubKeyHash
> 2. 将版本信息追加到pubKeyHash之前，生成versionedPayload，此时versionedPayload=version+pubKeyHash
> 3. 使用SHA256\(SHA256\(versionedPayload\)\)进行两次哈希得到一个hash值，取该值的前n个字节最终生成checksum
> 4. 将checksum追加到versionedPayload之后，生成编码前的地址，此时地址= version+pubKeyHash+checksum
> 5. 最后使用Base58对version+pubKeyHash+checksum编码生成最终的地址。

最终，我们得到一个真实的比特币地址，甚至可以在[blockchain.info](https://blockchain.info/)上查询该地址的余额。当然，无论查询多少次，该地址的余额都会是0。好的公钥加密算法是非常关键，它使产生相同私钥的概率尽可能低，低到不可能发生。同时，比特币地址生成算法使用一组开放的算法集合，因此不需要连接到比特币网络就可以生成比特币地址。

现在，修改TXI和TXO结构，在其中加入地址：

```go
type TXInput struct {
    Txid      []byte
    Vout      int
    Signature []byte
    PubKey    []byte
}

func (in *TXInput) UsesKey(pubKeyHash []byte) bool {
    lockingHash := HashPubKey(in.PubKey)

    return bytes.Compare(lockingHash, pubKeyHash) == 0
}

type TXOutput struct {
    Value      int
    PubKeyHash []byte
}

func (out *TXOutput) Lock(address []byte) {
    pubKeyHash := Base58Decode(address)
    pubKeyHash = pubKeyHash[1 : len(pubKeyHash)-4]
    out.PubKeyHash = pubKeyHash
}

func (out *TXOutput) IsLockedWithKey(pubKeyHash []byte) bool {
    return bytes.Compare(out.PubKeyHash, pubKeyHash) == 0
}
```

由于我们不打算实现一个脚本语言，因此不再使用**ScriptPubKey**和**ScriptSig。ScriptSig**被**Signature**和**PubKey**代替；**ScriptPubKey**被**PubKeyHash**代替。通过下面的方法中实现比特币中TXO的加解锁和TXI的签名逻辑：

**UsesKey**方法核查TXI是否可以使用特定的未哈希的公钥来解锁一个TXO。**IsLockedWithKey**核查TXO是否被特定的哈希后的公钥锁定。**UsesKey**和**IsLockedWithKey**是互补函数，**FindUnspentTransactions**函数使用这两个函数来构建交易间的关系。

**Lock**方法是对TXO进行加锁。当发给某人货币时，仅仅知道他们的地址，因此该方法唯一入参就是地址信息。从地址中从解码出哈希后的公钥，将其保存到**PubKeyHash**中。

现在，让我们看看是否能正常工作：

```
$ blockchain_go createwallet
Your new address: 13Uu7B1vDP4ViXqHFsWtbraM3EfQ3UkWXt

$ blockchain_go createwallet
Your new address: 15pUhCbtrGh3JUx5iHnXjfpyHyTgawvG5h

$ blockchain_go createwallet
Your new address: 1Lhqun1E9zZZhodiTqxfPQBcwr1CVDV2sy

$ blockchain_go createblockchain -address 13Uu7B1vDP4ViXqHFsWtbraM3EfQ3UkWXt
0000005420fbfdafa00c093f56e033903ba43599fa7cd9df40458e373eee724d

Done!

$ blockchain_go getbalance -address 13Uu7B1vDP4ViXqHFsWtbraM3EfQ3UkWXt
Balance of '13Uu7B1vDP4ViXqHFsWtbraM3EfQ3UkWXt': 10

$ blockchain_go send -from 15pUhCbtrGh3JUx5iHnXjfpyHyTgawvG5h -to 13Uu7B1vDP4ViXqHFsWtbraM3EfQ3UkWXt -amount 5
2017/09/12 13:08:56 ERROR: Not enough funds

$ blockchain_go send -from 13Uu7B1vDP4ViXqHFsWtbraM3EfQ3UkWXt -to 15pUhCbtrGh3JUx5iHnXjfpyHyTgawvG5h -amount 6
00000019afa909094193f64ca06e9039849709f5948fbac56cae7b1b8f0ff162

Success!

$ blockchain_go getbalance -address 13Uu7B1vDP4ViXqHFsWtbraM3EfQ3UkWXt
Balance of '13Uu7B1vDP4ViXqHFsWtbraM3EfQ3UkWXt': 4

$ blockchain_go getbalance -address 15pUhCbtrGh3JUx5iHnXjfpyHyTgawvG5h
Balance of '15pUhCbtrGh3JUx5iHnXjfpyHyTgawvG5h': 6

$ blockchain_go getbalance -address 1Lhqun1E9zZZhodiTqxfPQBcwr1CVDV2sy
Balance of '1Lhqun1E9zZZhodiTqxfPQBcwr1CVDV2sy': 0
```



