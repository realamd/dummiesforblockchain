目前为止，程序还没有提供任何交互接口，仅仅在main函数中调用**NewBlockchain**, **bc.AddBlock**等函数。现在让我们实现以下命令：

```
blockchain_go addblock "Pay 0.031337 for a coffee"
blockchain_go printchain
```

所有命令行命令通过**CLI**结构处理：

```go
type CLI struct {
    bc *Blockchain
}
```

CLI的入口是**Run**函数：

```go
func (cli *CLI) Run() {
    cli.validateArgs()

    addBlockCmd := flag.NewFlagSet("addblock", flag.ExitOnError)
    printChainCmd := flag.NewFlagSet("printchain", flag.ExitOnError)

    addBlockData := addBlockCmd.String("data", "", "Block data")

    switch os.Args[1] {
    case "addblock":
        err := addBlockCmd.Parse(os.Args[2:])
    case "printchain":
        err := printChainCmd.Parse(os.Args[2:])
    default:
        cli.printUsage()
        os.Exit(1)
    }

    if addBlockCmd.Parsed() {
        if *addBlockData == "" {
            addBlockCmd.Usage()
            os.Exit(1)
        }
        cli.addBlock(*addBlockData)
    }

    if printChainCmd.Parsed() {
        cli.printChain()
    }
}
```

我们使用flag标准库处理命令行参数：

```go
addBlockCmd := flag.NewFlagSet("addblock", flag.ExitOnError)
printChainCmd := flag.NewFlagSet("printchain", flag.ExitOnError)
addBlockData := addBlockCmd.String("data", "", "Block data")
```

首先，创建两个子命令：**addblock**、**printchain，addblock**具有一个命令参数**-data，printchain**没有任何命令参数。

```go
switch os.Args[1] {
case "addblock":
    err := addBlockCmd.Parse(os.Args[2:])
case "printchain":
    err := printChainCmd.Parse(os.Args[2:])
default:
    cli.printUsage()
    os.Exit(1)
}
```

接下来，我们检查用户输入的命令以及与之相关的命令参数：

```go
f addBlockCmd.Parsed() {
    if *addBlockData == "" {
        addBlockCmd.Usage()
        os.Exit(1)
    }
    cli.addBlock(*addBlockData)
}

if printChainCmd.Parsed() {
    cli.printChain()
}
```

对于解析成功的应命令执行对应的函数。

```go
func (cli *CLI) addBlock(data string) {
	cli.bc.AddBlock(data)
	fmt.Println("Success!")
}

func (cli *CLI) printChain() {
	bci := cli.bc.Iterator()

	for {
		block := bci.Next()

		fmt.Printf("Prev. hash: %x\n", block.PrevBlockHash)
		fmt.Printf("Data: %s\n", block.Data)
		fmt.Printf("Hash: %x\n", block.Hash)
		pow := NewProofOfWork(block)
		fmt.Printf("PoW: %s\n", strconv.FormatBool(pow.Validate()))
		fmt.Println()

		if len(block.PrevBlockHash) == 0 {
			break
		}
	}
}
```



