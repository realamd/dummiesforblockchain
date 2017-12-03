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



