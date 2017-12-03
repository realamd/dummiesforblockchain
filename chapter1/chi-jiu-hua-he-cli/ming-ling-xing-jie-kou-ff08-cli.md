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



