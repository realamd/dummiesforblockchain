Mist是以太坊官方提供的钱包应用，为geth的GUI版本。官方下载地址：[https://github.com/ethereum/mist/releases，本文中使用的是0.9.3版本。](https://github.com/ethereum/mist/releases，本文中使用的是0.9.3版本。)

# 连接私有链

启动Mist，过一段时间后会出现如下错误：

![](/assets/3.1.1.png)

为什么会这样呢？原因如下：

Mist启动后查找本地默认ipc是否存在（Mac默认文件为$HOME/Library/Ethereum/geth.ipc；Windows默认文件为.\pipe\geth.ipc；Linux默认文件为/.ethereum/geth.ipc），如果存在则发送rpc请求获取相关数据，否则将从公有链网络下载Genesis block等信息初始化本地链并启动节点。

而在本例中，节点初始化和启动均使用了自定义目录$HOME/share/q-btc/data，因此Mist没有连接到私有链网络，而是连接到公有链获取信息初始化并启动节点。

此时，本地节点和Mist启动的新节点均使用的是默认端口30303，发生了端口冲突，因此出现上面的错误。

为了使Mist连接到本地节点，可以在启动Mist时设--rpc参数指定ipc文件：

```
# cd /Applications/Mist.app/Contents/
# ./Mist --rpc $HOME/share/q-btc/data/geth.ipc
[xxxx-xx-xx 01:04:34.881] [INFO] main - Running in production mode: true
[xxxx-xx-xx 01:04:34.912] [INFO] EthereumNode - undefined null 'fast'
[xxxx-xx-xx 01:04:34.913] [INFO] EthereumNode - Defaults loaded: geth main fast
[xxxx-xx-xx 01:04:35.258] [INFO] main - Starting in Mist mode
[xxxx-xx-xx 01:04:35.349] [INFO] Db - Loading db: $HOME/Library/Application Support/Mist/mist.lokidb
[xxxx-xx-xx 01:04:35.358] [INFO] Windows - Creating commonly-used windows
[xxxx-xx-xx 01:04:35.358] [INFO] Windows - Create secondary window: loading, owner: notset
[xxxx-xx-xx 01:04:35.456] [INFO] updateChecker - Check for update...
[xxxx-xx-xx 01:04:38.396] [INFO] Windows - Create primary window: main, owner: notset
[xxxx-xx-xx 01:04:38.404] [INFO] Windows - Create primary window: splash, owner: notset
[xxxx-xx-xx 01:04:38.786] [INFO] ipcCommunicator - Backend language set to:  zh
[xxxx-xx-xx 01:04:39.208] [INFO] (ui: splashscreen) - Web3 already initialized, re-using provider.
[xxxx-xx-xx 01:04:39.275] [INFO] (ui: splashscreen) - Meteor starting up...
[xxxx-xx-xx 01:04:39.453] [INFO] ClientBinaryManager - Initializing...
[xxxx-xx-xx 01:04:39.454] [INFO] ClientBinaryManager - Checking for new client binaries config from: https://raw.githubusercontent.com/ethereum/mist/master/clientBinaries.json
[xxxx-xx-xx 01:04:40.536] [INFO] ClientBinaryManager - No "skippedNodeVersion.json" found.
[xxxx-xx-xx 01:04:40.537] [INFO] ClientBinaryManager - Initializing...
[xxxx-xx-xx 01:04:40.538] [INFO] ClientBinaryManager - Resolving platform...
[xxxx-xx-xx 01:04:40.538] [INFO] ClientBinaryManager - Calculating possible clients...
[xxxx-xx-xx 01:04:40.539] [INFO] ClientBinaryManager - 1 possible clients.
[xxxx-xx-xx 01:04:40.540] [INFO] ClientBinaryManager - Verifying status of all 1 possible clients...
[xxxx-xx-xx 01:04:40.540] [INFO] ClientBinaryManager - Verify Geth status ...
[xxxx-xx-xx 01:04:40.574] [INFO] ClientBinaryManager - Checking for Geth sanity check ...
[xxxx-xx-xx 01:04:40.574] [INFO] ClientBinaryManager - Checking for Geth sanity check ...
[xxxx-xx-xx 01:04:40.574] [INFO] ClientBinaryManager - Checking sanity for Geth ...
[xxxx-xx-xx 01:04:40.579] [INFO] ClientBinaryManager - Checking sanity for Geth ...
[xxxx-xx-xx 01:04:40.662] [ERROR] ClientBinaryManager - Sanity check failed for Geth Error: Unable to find "1.7.2" in Geth output
    at Promise.resolve.then.then.then (/Applications/Mist.app/Contents/Resources/app.asar/node_modules/ethereum-client-binaries/src/index.js:635:17)
    at <anonymous>
    at process._tickCallback (internal/process/next_tick.js:109:7)
[xxxx-xx-xx 01:04:40.795] [INFO] Sockets/node-ipc - Connect to {"path":"$HOME/share/q-btc/data/geth.ipc"}
[xxxx-xx-xx 01:04:40.797] [INFO] Sockets/node-ipc - Connected!
[xxxx-xx-xx 01:04:40.798] [INFO] NodeSync - Ethereum node connected, re-start sync
[xxxx-xx-xx 01:04:40.798] [INFO] NodeSync - Starting sync loop
[xxxx-xx-xx 01:04:40.799] [INFO] Sockets/3 - Connect to {"path":"$HOME/share/q-btc/data/geth.ipc"}
[xxxx-xx-xx 01:04:40.810] [INFO] Sockets/3 - Connected!
[xxxx-xx-xx 01:04:40.835] [INFO] (ui: splashscreen) - Network is privatenet
[xxxx-xx-xx 01:04:40.836] [INFO] (ui: splashscreen) - Network is privatenet
[xxxx-xx-xx 01:04:43.383] [INFO] main - Connected via IPC to node.
[xxxx-xx-xx 01:04:43.515] [INFO] updateChecker - App is up-to-date.
```

**注意：**“Sanity check failed for Geth Error**”**错误信息标识Mist所需的geth版本信息与从私有链获取到的geth版本信息不匹配，暂没发现该错误对后续应用造成影响，可以先忽略。![](/assets/3.1.2.png)Mist界面启动后，右上角显示PRIVATE-NET，已经成功连接到私有链网络中了，点击LAUNCH APPLICATION，进入Mist主界面：![](/assets/3.1.3.png)可以看到界面显示已经有一个账户，拥有1590个以太币，与之前geth命令行返回的信息一致。

# 创建账户

账户是一对公钥/私钥对，用于进行挖矿、交易、存储ether等活动。之前已经通过Geth创建了一个账户，该账户为Main Account，挖矿获得的ether默认存在该账户。账户可以有多个，下面通过Mist创建另一个账户![](/assets/3.1.4.png)![](/assets/3.1.5.png)![](/assets/3.1.6.png)![](/assets/3.1.7.png)![](/assets/3.1.8.png)至此，新创建了一个账户ACCOUNT3，该账户地址为**0xD12B5885d62BB06c76A1699E560C37AC06Ee81C4。**

账户文件保存在--datadir目录中的keystore目录中：

```
Chris:keystore zhangli$ ls $HOME/share/q-btc/private_chain/data/keystore
UTC--2017-11-25T14-50-17.692945588Z--205c6e56f2b809d686b4afc42b241004c985c900
UTC--2017-11-26T13-28-26.329917826Z--66e840ec7caf3538c8aaa34d74d882a33fc20ce6
UTC--2017-11-26T14-17-13.507560584Z--d12b5885d62bb06c76a1699e560c37ac06ee81c4
```

# 创建交易

从MAIN ACCOUNT向ACCOUNT3转入一些ether，会创建一个交易：

![](/assets/3.1.9.png)![](/assets/3.1.10.png)![](/assets/3.1.11.png)![](/assets/3.1.12.png)![](/assets/3.1.13.png)输入MAIN ACCOUNT账户密码，SEND TRANSACTION后创建一个交易，可以在MAIN ACCOUNT账户的交易列表中看到：![](/assets/3.1.14.png)点击该交易，可以看到交易信息：![](/assets/3.1.15.png)交易ID为0x8c50622c1648b46c7f94cc9aca8eded3af3231eb611d61bfeaae2e9ffefa702f，此时console会显示日志：

```
> INFO [xx-xx|22:26:19] Submitted transaction                    fullhash=0x8c50622c1648b46c7f94cc9aca8eded3af3231eb611d61bfeaae2e9ffefa702f recipient=0xD12B5885d62BB06c76A1699E560C37AC06Ee81C4
```

该交易一直处于灰色，显示“0的12确认区块”，不会有任何变化，而ACCOUNT3的余额一直未0。这是因为目前没有任何节点进行挖矿操作，因此不会将该交易放入私有链中。

通过console启动挖矿：

```
> miner.start(1)
```

开始挖矿后，可以看到交易开始产生进度，信息也开始有所变化：![](/assets/3.1.16.png)当交易完成时，交易状态如下，同时ACCOUNT3的余额变为200：![](/assets/3.1.17.png)![](/assets/3.1.18.png)

