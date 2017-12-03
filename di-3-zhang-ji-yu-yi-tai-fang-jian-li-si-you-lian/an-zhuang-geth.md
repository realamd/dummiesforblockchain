以太坊\(Ethereum\)客户端实现以太坊协议，用于接入以太坊网络进行各种操作，例如挖矿、交易等。网上有各种语言实现的以太坊客户端，其中官方提供了基于Go和C++的CLI客户端，即go-ethereum和cpp-ethereum。由于go-ethereum实现相对较完备且在学习Go语言，因此将使用go-ethereum作为客户端。

go-ethereum中包括许多命令，其中最重要的命令时Geth，该命令具备初始化、启动、挖矿、创建账户等许多重要功能，因此一般以Geth来代表以太坊客户端。

在Mac上安装Geth十分简单，有3种方法：

> * 下载预编译好的安装包https://geth.ethereum.org/downloads
> * 基于源码编译https://github.com/ethereum/go-ethereum
> * 通过brew安装

本文将使用brew的安装方法。安装Geth执行如下命令：

```
# brew tap ethereum/ethereum
# brew install ethereum
  需要等待一段时间…
```

说明：由于brew官方源中不包含ethereum，因此需要通过**brew tap**添加以太坊源（类似yum添加第三方源）。

安装完成后执行geth version命令查看版本信息：

```
Geth
Version: 1.7.3-stable
Architecture: amd64
Protocol Versions: [63 62]
Network Id: 1
Go Version: go1.9.2
Operating System: darwin
…
```



