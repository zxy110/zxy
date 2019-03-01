---
title: VRF
date: 2019-03-01 15:51:15
categories:
  - 密码学
tags: [区块链, 零知识证明]
---

##### 1. Why VRF？

- 场景

  在区块链场景中，有的框架会用算法随机产生出块节点与验证节点（如Algorand），甚至解决分叉。按传统的随机算法，按一定的哈希规则随机轮询，选出一个节点来记账/验证。如果这个随机轮询的规则是谁都可以复现的，那么可以推测出将来的某个记账/验证节点，集中攻击它。

  为了解决这个问题，就引入了VRF，只有自己能够完成这个哈希过程，而别人只能在他声明之后验证这个过程，防止有人可以提前推测出将来的记账节点。						

- POS中的权益研磨（Grinding）

  以下来源于以太坊Github上的《[Proof of Stake FAQ](https://github.com/ethereum/wiki/wiki/Proof-of-Stake-FAQs)》：

  在任何基于区块链的权益证明算法中，都需要某种机制，来随机从当前活跃验证者集合中选择能够产生下一个区块的验证者。举个例子，如果当前活跃的验证者集合由持有40以太币的 Alice，持有 30 以太币的 Bob，持有 20 以太币的 Charlie 与持有 10 以太币的 David 组成，那么你想让 Alice 成为下一个区块的创建者的概率为 40%，而 Bob 的概率为 30% 等（在实践中，不仅要随机选择一个验证者，而是要（随机产生）一个无限验证者序列，只有这样如果 Alice 不在线的时候，就可以有其他人在过段时间替代她，但是这并没有改变问题的本质）。在非基于区块链的算法中，出于不同的原因也经常需要考虑随机性。

  ​ 以下来源Ouroboros白皮书《[Ouroboros: A Provably Secure Proof-of-Stake Blockchain Protocol](https://link.jianshu.com/?t=https%3A%2F%2Feprint.iacr.org%2F2016%2F889.pdf%3Fnsukey%3DQRILh1BmjE5k%252BvQjynm%252F8CQnpycVkRtlhQSCk3m9mGPIMbtcRp5Akse%252FVt9b6v24XVK27vaSczZjH%252BtBcuUsAihW4l0RO%252Bjea4aSj%252BhS4ktWhidEePrI2uG3GEECQ2PoBe8vMZMhR93MVWyaHdT9P29f4vpunEIPWUNbnfXx4zvXTi%252B1FWAMlqqRAnyYnQhu8jUgX%252FEqrqKbyl%252B6HsE2Fw%253D%253D)》：

  基于PoS的区块链协议最基本的一个问题就是模拟领导者选举过程。为了在股东们之间的选举达到一个真正的随机性，系统中就必须要引入熵(entropy)，但引入熵的机制可能会容易被敌手操作。例如，一个控制一群股东的敌手可能会试图模拟协议的执行，尝试不同的股东参与者的顺序以此来找到对敌对股东有力的继续者。这会导致一个叫做"grinding"的致命弱点，敌对参与者可能会使用计算资源来倾斜领导者选举。

- VRF的目的

  VRF 的目的就是要生成随机值，既要可验证，可重放又要无法被预测。

##### 2. VRF是什么?

VRF是可验证随机函数(verifiable random function)，一方面具有伪随机性，另一方面它还具有可验证性(输出包括一个非交互零知识证明)

- 伪随机性
- 可验证性

 VRF 的方式是，实现本地抽签，各个节点自己抽签，如果抽中了之后，大家可以很容易地验证这个结果确实是你生成的。

eg. 假设现在是 round 10（第 10 轮），节点们可能会轮流抽签，以节点自己的私钥 + 一个全网都知道的随机数（比如是这轮的轮次 10）作为输入，生成了一个随机数（0-100）；设置一个条件：100 个节点轮流抽签，谁先抽出来的随机数大于 10，就是这一轮的打包者。假设 5 号节点抽到了 11，可是只有 5 号知道其他人不知道，因此他在广播这个随机的同时还需要广播一个零知识证明。通过零知识证明，全网只需要通过 5 号的公钥就可以验证，接受 5 号为这轮打包者。图解如下：

![VRF举例](F:\BC&NDN&IOT\Privicy\VRF\Image\VRF举例.png)

##### 3. VRF具体的操作流程？

![四个函数](F:\BC&NDN&IOT\Privicy\VRF\Image\四个函数.png)

- 证明者生成一对密钥，PK、SK；
- 证明者计算result = VRF_Hash（SK，info），proof = VRF_Proof（SK，info）；
- 证明者把result，proof，PK递交给验证者；
- 验证者计算result = VRF_P2H（proof)，True/False = VRF_Verify(PK, info, proof) 

True表示验证通过，False表示验证未通过。所谓的验证通过，就是指proof是否是通过info生成的，通过proof是否可以计算出result，从而推导出info和result是否对应匹配、证明者给出的材料是否有问题。

##### 4. 抽签有没有必要用VRF？

- 相比随机预言机

1. 普通哈希Hash(a)=b，所有人都可以重现，检验正确性；
2. VRF是Hash(SIG(sk, a))=b，别人无法复现这个过程。但是可以拿b，pk，和中间信息验证b是跟a对应的。

- 相比非对称加密

1. 在密码学签名算法中，大都会引入随机性，也就是对相同信息的多次签名会得到不同的签名值，因此矿工可以不断对相同的输入SK和block，计算签名，以满足结果小于D。那么理论上任何人都会成为出块者，只要计算足够多次的签名。
2. 有些非对称加密方式得到的随机数不是均匀分布的，如RSA
3. 缺乏零知识，不管使用确定性签名还是随机性签名，都存在个安全隐患。就是一旦将自己的出块凭证公布，任何人都可以公开验证，包括攻击者。那么攻击者可以对出块节点进行攻击，使其不能出块。使用VRFs的方式，矿工只需要公布自己的R表明自己的出块权，当出完块的时候再公布P，那么攻击者就无法在出块之前知道谁具有出块权，因此也就无法实施针对性的攻击。

##### 5. 应用

1. **Consensus：共识算法中安全性**

   VRF Sortition，Smart Contracts， 例如本体，Cardano，Dfinity，Algorand等，不同点在于如何产生输入以及输出怎样用。VRF的返回结果可以用来公开或私密地完成节点或节点群体的选择。eg. Dfinity利用mod操作来唯一，公开的确定一个group。Algorand，Ouroboros Praos是私密选择，即计算出哈希值后，如果哈希值小于某个阈值，节点可以私密地知道自己被选中。

![应用](F:\BC&NDN&IOT\Privicy\VRF\Image\应用.png)

- **本体-VBFT共识算法**：

  1. 根据 VRF 从共识网络中选择备选提案节点，各个备选节点将独立提出备选区块；
  2. 根据 VRF 从共识网络中选择多个验证节点，每个验证节点将从网络中收集备选的区块，进行验证，然后对最高优先级的备选区块进行投票；
  3. 根据 VRF 从共识网络中选择多个确认节点，对上述验证节点的投票结果进行统计验证，并确定出最终的共识结果。
  4. 所有节点都将接收确认节点的共识结果，并在一轮共识确认后开启新的共识。

- **Algorand中**：

  1. 先选打包者，选完打包者选委员会，委员会用BA*进行选区块。
  2. 输入值由前一个随机数(最初的随机数是协议给定的)和某种代表高度，轮次的变量进行组合，然后用私钥对之进行签名(或者先签名再组合)，最后哈希一下得出最新的随机数。
  3. 条件：①签名算法应当具有唯一性；②避免在生成新随机数时将当前块的数据作为随机性来源之一。

- **Dfinity中**：

  交保证金提高门槛，并降低参与节点的数量，然后选打包者，选完打包者选公证人，对区块权重进行排序，选出区块。

- **Cardano的共识机制-Ouroboros Praos**：

  在根据Random seed选举slot leader时，通过VRF确保slot leader不被事先计算出来被攻击。

  ![Ouroboros](F:\BC&NDN&IOT\Privicy\VRF\Image\Ouroboros.png)![Ouroboros Paros](F:\BC&NDN&IOT\Privicy\VRF\Image\Ouroboros Paros.png)

1. **IOST的高效分布式分配片**

   使用了VRF来进行领头节点的选举，通过VRF得到随机数之后，会将结果进行广播，然后其他节点会进行统计，得到随机数值最小的作为分片领头节点。是一种交互式的选举方式。--用VRF保证公正性和不可攻击的特性。

2. **Key Transparency**

   密钥管理系统，使我的消息在不相信服务端的情况下做到点对点的安全上的提升。

3. **DNSSEC**

   DNS服务的安全性。