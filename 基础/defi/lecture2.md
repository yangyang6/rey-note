密码学



什么是区块链？

抽象的回答：

区块链提供了一种让多个参与方在没有一个唯一可信方的情况下达成的合作

若有可信第三方 => 不需要区块链

【金融系统中常常没有可信的参与方】





共识层

一个公共的 仅附加的数据结构：

* 不可篡改性：一旦添加，数据永远无法被删除
* 一致性：所有的诚实参与方有相同的数据
* 活性：诚实的参与方总能够添加新的交易
* 开放性（？） ： 任何人都能够添加数据 【联盟链等不公开】





应用层：去中心化应用（DAPPS）

一个应用：承诺数据



### 加密学背景

##### 加密哈希函数

一个有效计算函数  H：M -> T 

百万字节 ->  32字节 哈希值  = {0,1}^256

抗碰撞性 （斯坦佛大学课程），论文 去详细了解



默克尔树（Merkle树）

```
默克尔树逐层记录哈希值的特点，让它具有了一些独特的性质。例如，底层数据的任何变动，都会传递到其父节点，一层层沿着路径一直到树根。这意味着根的值实际上代表了对底层所有数据的“数字摘要”。
```

应用场景：

* 证明某个集合中存在或不存在某个元素
* 快速比较大量数据
* 快速定位修改
* 零知识证明



默克尔证明：用来证明一个交易是否在链上





数字化签名

解决方案：让签名由文件来决定

定义：一个签名的方案由三个算法构成：

* Gen()：产出一对秘钥（pk，sk）
* Sign(sk,msg)：产生sigs/δ 
* Verify(pk,msg,δ )  产出’接受‘或’拒绝‘

签名方案的种类

* RSA签名（不会用在区块链上）

​		sigs和公钥太大（>=256字节），可以很快速地验证

* 离散型对数签名：Schnorr和ECDSA（比特币，以太坊）

​		sigs短（48或64字节）且公钥也短（32字节）

* BLS签名：48字节，可聚合，低门槛（以太坊2.0，Chia）
* 后量子签名：长（>=768字节）





以太坊Rollup（卷叠）扩容