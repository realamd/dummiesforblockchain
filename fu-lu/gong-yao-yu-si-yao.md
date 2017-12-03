公钥和私钥俗称非对称加密，公钥和私钥称为密钥对：用公钥加密的内容只能用私钥解密；用私钥加密的内容只能用公钥解密。

公钥和私钥一般有两种用途：加密（公钥加密，私钥解密）和认证（私钥加密，公钥解密）。

以下示例（引用自[中文翻译](http://www.cnblogs.com/shijingjing07/p/5965792.html)或[英文原文](http://www.youdzone.com/signature.html)）生动形象地说明两种应用场景，其中1-4是加密过程，5-9是认证过程，而10-12解释了认证中心的必要性：

\(1\) 鲍勃有两把钥匙，一把是公钥，另一把是私钥

![](/assets/f1.png)

\(2\) 鲍勃把公钥送给他的朋友们----帕蒂、道格、苏珊----每人一把。

![](/assets/f2.png)

\(3\) 苏珊要给鲍勃写一封保密的信。她写完后用鲍勃的公钥加密，就可以达到保密的效果。

![](/assets/f3.png)

\(4\) 鲍勃收信后，用私钥解密，就看到了信件内容。这里要强调的是，只要鲍勃的私钥不泄露，这封信就是安全的，即使落在别人手里，也无法解密。

![](/assets/f4.png)

\(5\) 鲍勃给苏珊回信，决定采用"数字签名"。他写完后先用Hash函数，生成信件的摘要（digest）。

![](/assets/f5.png)

\(6\) 然后，鲍勃使用私钥，对这个摘要加密，生成"数字签名"（signature）。

![](/assets/f6.png)

\(7\) 鲍勃将这个签名，附在信件下面，一起发给苏珊。

![](/assets/f7.png)

\(8\) 苏珊收信后，取下数字签名，用鲍勃的公钥解密，得到信件的摘要。由此证明，这封信确实是鲍勃发出的。

![](/assets/f8.png)

\(9\) 苏珊再对信件本身使用Hash函数，将得到的结果，与上一步得到的摘要进行对比。如果两者一致，就证明这封信未被修改过。

![](/assets/f9.png)

\(10\) 复杂的情况出现了。道格想欺骗苏珊，他偷偷使用了苏珊的电脑，用自己的公钥换走了鲍勃的公钥。此时，苏珊实际拥有的是道格的公钥，但是还以为这是鲍勃的公钥。因此，道格就可以冒充鲍勃，用自己的私钥做成"数字签名"，写信给苏珊，让苏珊用假的鲍勃公钥进行解密。

![](/assets/f10.png)

\(11\) 后来，苏珊感觉不对劲，发现自己无法确定公钥是否真的属于鲍勃。她想到了一个办法，要求鲍勃去找"证书中心"（certificate authority，简称CA），为公钥做认证。证书中心用自己的私钥，对鲍勃的公钥和一些相关信息一起加密，生成"数字证书"（Digital Certificate）。

![](/assets/f11.png)

\(12\) 鲍勃拿到数字证书以后，就可以放心了。以后再给苏珊写信，只要在签名的同时，再附上数字证书就行了。

![](/assets/f12.png)

\(13\) 苏珊收信后，用CA的公钥解开数字证书，就可以拿到鲍勃真实的公钥了，然后就能证明"数字签名"是否真的是鲍勃签的。

![](/assets/f13.png)
