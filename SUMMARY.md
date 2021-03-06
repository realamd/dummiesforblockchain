# Summary

* [前言](README.md)
* [第1章 去中心化世界](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue.md)
  * [去中心化](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue/bi-te-bi/yun-gong-shi.md)
    * [引言](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue/bi-te-bi/yun-gong-shi/yin-yan.md)
    * [分布式系统与去中心化](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue/bi-te-bi/yun-gong-shi/fen-bu-shi-xi-tong-yu-qu-zhong-xin-hua.md)
    * [Paxos算法](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue/bi-te-bi/yun-gong-shi/rong-cuoxing-yu-paxos.md)
    * [云共识与发展](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue/bi-te-bi/yun-gong-shi/gong-shi-yu-fa-zhan.md)
  * [区块链](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue/qu-kuai-lian.md)
    * [引言](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue/qu-kuai-lian/yin-yan.md)
    * [什么是区块链](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue/qu-kuai-lian/shi-yao-shi-qu-kuai-lian.md)
    * [结构](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue/qu-kuai-lian/jie-gou.md)
    * [分类](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue/qu-kuai-lian/fen-lei.md)
  * [共识算法](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue/bi-te-bi/gong-shi-suan-fa.md)
    * [引言](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue/bi-te-bi/gong-shi-suan-fa/yin-yan.md)
    * [共识](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue/bi-te-bi/gong-shi-suan-fa/gong-shi.md)
  * [智能合约](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue/zhi-neng-he-yue.md)
    * [引言](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue/zhi-neng-he-yue/yin-yan.md)
  * [比特币](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue/bi-te-bi.md)
    * [引言](di-1-zhang-bi-te-bi-3001-qu-kuai-lian-3001-zhi-neng-he-yue/bi-te-bi/yin-yan.md)
* [第2章 基于Go实现区块链](chapter1.md)
  * [基本原型](chapter1/ji-ben-yuan-xing.md)
    * [引言](chapter1/ji-ben-yuan-xing/yin-yan.md)
    * [区块\(Block\)](chapter1/ji-ben-yuan-xing/qu-kuai-ff08-block.md)
    * [区块链\(Blockchain\)](chapter1/ji-ben-yuan-xing/qu-kuai-lian-ff08-blockchain.md)
    * [总结](chapter1/ji-ben-yuan-xing/zong-jie.md)
  * [工作量证明](chapter1/gong-zuo-liang-zheng-ming.md)
    * [引言](chapter1/gong-zuo-liang-zheng-ming/yin-yan.md)
    * [PoW](chapter1/gong-zuo-liang-zheng-ming/pow.md)
    * [哈希](chapter1/gong-zuo-liang-zheng-ming/ha-xi.md)
    * [Hashcash算法](chapter1/gong-zuo-liang-zheng-ming/hashcashsuan-fa.md)
    * [实现](chapter1/gong-zuo-liang-zheng-ming/shi-xian.md)
    * [总结](chapter1/gong-zuo-liang-zheng-ming/zong-jie.md)
  * [持久化和CLI](chapter1/chi-jiu-hua-he-cli.md)
    * [引言](chapter1/chi-jiu-hua-he-cli/yin-yan.md)
    * [数据库选型](chapter1/chi-jiu-hua-he-cli/shu-ju-ku-xuan-xing.md)
    * [BoltDB](chapter1/chi-jiu-hua-he-cli/boltdb.md)
    * [数据库结构](chapter1/chi-jiu-hua-he-cli/shu-ju-ku-jie-gou.md)
    * [序列化](chapter1/chi-jiu-hua-he-cli/xu-lie-hua.md)
    * [持久化](chapter1/chi-jiu-hua-he-cli/chi-jiu-hua.md)
    * [查看blockchain内容](chapter1/chi-jiu-hua-he-cli/cha-kan-blockchain-nei-rong.md)
    * [命令行接口\(CLI\)](chapter1/chi-jiu-hua-he-cli/ming-ling-xing-jie-kou-ff08-cli.md)
    * [总结](chapter1/chi-jiu-hua-he-cli/zong-jie.md)
  * [交易](chapter1/jiao-yi.md)
    * [引言](chapter1/jiao-yi/yin-yan.md)
    * [There is no spoon](chapter1/jiao-yi/there-is-no-spoon.md)
    * [比特币交易](chapter1/jiao-yi/bi-te-bi-jiao-yi.md)
    * [交易输出\(TXO\)](chapter1/jiao-yi/jiao-yi-shu-51fa28-txo.md)
    * [交易输入\(TXI\)](chapter1/jiao-yi/jiao-yi-shu-516528-txi.md)
    * [先有蛋后有鸡](chapter1/jiao-yi/xian-you-dan-hou-you-ji.md)
    * [交易\(Transaction\)](chapter1/jiao-yi/jiao-yi-guo-cheng.md)
    * [工作量证明](chapter1/jiao-yi/gong-zuo-liang-zheng-ming.md)
    * [未消费TXO\(UTXO\)](chapter1/jiao-yi/wei-xiao-fei-txo-utxo.md)
    * [货币流通](chapter1/jiao-yi/huo-bi-liu-tong.md)
    * [总结](chapter1/jiao-yi/zong-jie.md)
  * [地址](chapter1/di-zhi.md)
    * [引言](chapter1/di-zhi/yin-yan.md)
    * [比特币地址](chapter1/di-zhi/bi-te-bi-di-zhi.md)
    * [公钥加密](chapter1/di-zhi/gong-yao-jia-mi.md)
    * [数字签名](chapter1/di-zhi/shu-zi-qian-ming.md)
    * [椭圆曲线加密算法](chapter1/di-zhi/tuo-yuan-qu-xian-jia-mi-suan-fa.md)
    * [Base58编码算法](chapter1/di-zhi/base58bian-ma-suan-fa.md)
    * [地址](chapter1/di-zhi/di-zhi.md)
    * [签名](chapter1/di-zhi/qian-ming.md)
    * [总结](chapter1/di-zhi/zong-jie.md)
  * [再论交易](chapter1/zai-tan-jiao-yi.md)
    * [引言](chapter1/zai-tan-jiao-yi/yin-yan.md)
    * [奖励](chapter1/zai-tan-jiao-yi/jiang-li.md)
    * [UTXO集合](chapter1/zai-tan-jiao-yi/utxoji-he.md)
    * [默克尔树\(Merkle Tree\)](chapter1/zai-tan-jiao-yi/mo-ke-er681128-merkle-tree.md)
    * [P2PKH](chapter1/zai-tan-jiao-yi/p2pkh.md)
    * [总结](chapter1/zai-tan-jiao-yi/zong-jie.md)
  * [网络](chapter1/wang-luo.md)
    * 引言
    * [Blockchain网络](chapter1/wang-luo/blockchainwang-luo.md)
    * [节点角色](chapter1/wang-luo/jie-dian-jiao-se.md)
    * [精简](chapter1/wang-luo/jing-jian.md)
    * [实现](chapter1/wang-luo/shi-xian.md)
    * [演练](chapter1/wang-luo/yan-lian.md)
    * [总结](chapter1/wang-luo/zong-jie.md)
* [第3章 "集市"以太坊](di-4-zhang-dapps.md)
  * [以太坊概述](di-4-zhang-dapps/yi-tai-fang-gai-shu.md)
  * [基于以太坊建立私有链](di-3-zhang-ji-yu-yi-tai-fang-jian-li-si-you-lian.md)
    * [软/硬件环境](di-3-zhang-ji-yu-yi-tai-fang-jian-li-si-you-lian/8f6f-ying-jian-huan-jing.md)
    * [安装Geth](di-3-zhang-ji-yu-yi-tai-fang-jian-li-si-you-lian/an-zhuang-geth.md)
    * [创建私有链](di-3-zhang-ji-yu-yi-tai-fang-jian-li-si-you-lian/chuang-jian-si-you-lian.md)
    * [钱包应用：Mist](di-3-zhang-ji-yu-yi-tai-fang-jian-li-si-you-lian/qian-bao-ying-yong-ff1a-mist.md)
    * [多节点互联](di-3-zhang-ji-yu-yi-tai-fang-jian-li-si-you-lian/duo-jie-dian-hu-lian.md)
  * [智能合约语言Solidity](di-4-zhang-dapps/zhi-neng-he-yue-yu-yan-solidity.md)
  * [构建自己的DApp](di-4-zhang-dapps/gou-jian-zi-ji-de-dapp.md)
* [第4章 "大教堂"Hyperledger](di-5-zhang-zhi-neng-he-yue.md)
  * [Hyperledger概述](di-5-zhang-zhi-neng-he-yue/hyperledgergai-shu.md)
  * [Hyperledger架构](di-5-zhang-zhi-neng-he-yue/hyperledgerjia-gou.md)
  * [构建联盟链应用](di-5-zhang-zhi-neng-he-yue/gou-jian-lian-meng-lian-ying-yong.md)
* [第5章 ZBChain平台](di-5-zhang-zbchain-ping-tai.md)
  * [ZBChain白皮书](di-5-zhang-zbchain-ping-tai/zbchainbai-pi-shu.md)
  * [ZBChain架构](di-5-zhang-zbchain-ping-tai/zbchainjia-gou.md)
  * [ZBChain应用](di-5-zhang-zbchain-ping-tai/zbchainying-yong.md)
* [附录](fu-lu.md)
  * [公钥与私钥](fu-lu/gong-yao-yu-si-yao.md)

