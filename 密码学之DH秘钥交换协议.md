## 密码学之DH秘钥交换协议

### DH秘钥交换协议简介

- 普遍大家都认为[公钥](https://baike.baidu.com/item/公钥)密码体制是迪菲(W.Diffie)和赫尔曼(E.Hellman)发明的！

Diffie-Hellman:一种确保共享KEY安全穿越不安全网络的方法，它是OAKLEY的一个组成部分。Whitefield与Martin Hellman在1976年提出了一个奇妙的密钥交换协议，称为Diffie-Hellman密钥交换**协议/算法**(Diffie-Hellman Key Exchange/Agreement Algorithm).这个机制的巧妙在于需要安全通信的双方可以用这个方法确定**对称密钥**。然后可以用这个密钥进行**加密和解密**。但是注意，这个密钥交换协议/算法只能用于密钥的交换，而不能进行消息的加密和解密。双方确定要用的密钥后，要使用其他对称密钥操作加密算法实现加密和解密消息。

**DHKE**是最早的**公钥协议**之一，它允许两方安全地交换数据，因此有人嗅探双方之间的通信，交换的信息就可以泄露。

**Oakley**协议是对Diffie-Hellman[密钥](https://baike.baidu.com/item/密钥)交换算法的优化，它保留了后者的优点，同时克服了其弱点。

Diffie–Hellman (DH) 方法是**匿名密钥协商方案**：它允许彼此没有先验知识的两方共同建立**通过不安全通道**的共享密钥**。

注意DHKE方法对**[**嗅探攻击**](https://en.wikipedia.org/wiki/Sniffing_attack)（数据拦截）有**抵抗力，但容易受到[**man- 中间人攻击**](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)（攻击者秘密中继并可能**改变两方之间的通信**）。

DH秘钥交换协议可以使用**离散对数**（经典的 [**DHKE**](https://en.wikipedia.org/wiki/Diffie–Hellman_key_exchange) 算法）实现或使用**椭圆曲线加密**（[**ECDH**]（https://en.wikipedia.org/wiki/Elliptic-curve_Diffie–Hellman）算法）。

Diffie–Hellman Key Exchange 以下简称为 **（DHKE）DH秘钥交换协议**

## 通过混合颜色进行密钥交换引出DH秘钥交换思想

**DH秘钥交换协议**与“**通过混合颜色交换密钥**”的概念非常相似，具有很好的视觉表现，简化了其理解。 这就是为什么我们首先要解释如何通过**混色**来交换秘密颜色。

- 混色密钥交换方案的设计假设 前提：如果我们有两种不同颜色的液体，我们可以**轻松地混合颜色**并获得新的颜色，但反向操作几乎是不可能的：**没有办法将颜色分开 混合颜色** 恢复到它们原来的颜色成分。

这是换色**场景**，步骤如下：

- **Alice** 和 **Bob**，就不需要保密的任意 **起始（共享）颜色**达成一致（例如 *黄色*）。
- **Alice** 和 **Bob** 分别选择他们自己保留的**秘密颜色**（例如 **red** 和 **sea green**）。
- 最后 **Alice** 和 **Bob** 将他们的秘密颜色与他们共同共享的颜色混合在一起。 获得的混合颜色区域准备公开交换（在我们的例子中*橙色*和*浅天蓝色*）。

![img](http://p0.qhimg.com/t01c9f374d5336ae4d5.png)

得到MIxed colors 之后的后续步骤如下：

- **Alice** 和 **Bob** 公开**交换混色**。
  - 我们假设没有有效的方法从混合颜色中提取（分离）秘密颜色，因此知道混合颜色的第三方无法透露秘密颜色。
- 最后，**Alice** 和**Bob** 将他们从合作伙伴那里收到的颜色与他们自己的秘密颜色混合在一起。
  - 结果是**最终颜色混合**（*黄棕色*），与合作伙伴的颜色混合相同。
  - 这个最终混色就是 安全交换的**共享密钥**  shared key。

![img](http://p0.qhimg.com/t01b501b741ac438ff4.png)

如果第三方拦截了颜色交换过程，那么他们在计算上很难确定秘密颜色。

总结：

- **Alice** 和 **Bob** 拥有协同一致的起始颜色，黄色
- **Alice** 和 **Bob**交换混合颜色，也就是 公钥
- **Alice** 和 **Bob**拥有自己的私钥 红色和绿色
- **Alice** 和 **Bob**通过起始颜色混合私钥，得到公钥Mixed
- **Alice** 和 **Bob**交换公钥（Mixed）
- **Alice** 和 **Bob**    公钥+私钥=最终一样的**Shared key共享秘钥**
- **Alice** 和 **Bob**用**Shared key共享秘钥 ** todo something

**DH秘钥交换协议**基于类似的概念，但使用 [**离散对数（discrete logarithms）**](https://en.wikipedia.org/wiki/Discrete_logarithm) 和 [**模幂运算（modular exponentiations）** ](https://en.wikipedia.org/wiki/Modular_exponentiation) 而不是颜色混合。

### 



# Diffie-Hellman 密钥交换 (DHKE) 协议

现在，让我们解释一下 **DHKE** 协议是如何工作的。

## DHKE 背后的数学

**DHKE** 基于 [**（模幂运算）modular exponentiations**](https://en.wikipedia.org/wiki/Modular_exponentiation) 的一个简单属性：

- (**ga**)**b** mod **p** = (**gb**)**a** mod **p**

- 其中 **g**、**a**、**b** 和 **p** 是正整数。

- 如果我们有 **A** = **ga** mod **p** 和 **B** = **gb** mod **p**，我们可以计算 **gab** mod  p**，不显示 **a 或 **b**（称为 **秘密指数**）。

- 在计算理论中，这些都不是可以找到秘密指数的有效算法。 如果我们有以下等式中的 **m**、**g** 和 **p**：

- **m** = **gs** mod **p**

- 没有找到秘密指数 **s** 的有效（快速）算法。 这被称为 [离散对数问题 (DLP)](https://en.wikipedia.org/wiki/Discrete_Logarithm_Problem_(DLP))。

## 离散对数问题 Discrete Logarithm Problem (DLP)

计算机科学中的**离散对数问题 (DLP)** 定义如下：

- 在密码学中，许多算法依赖于 **DLP 问题的计算难度**在精心挑选的组上，**不存在有效的算法**。
- 离散对数问题是指从已知的A, g, p，很难求得a，这里的计算很难的关键是p是个很大的素数，比如1024-bit, 2048-bit, 3076-bit。

## DHKE 协议

现在，在我们熟悉了模幂的上述数学性质后，我们准备解释**DHKE协议**。 这是它的工作原理：

![img](http://p0.qhimg.com/t01884310706da57b25.png)

让我们解释一下这个密钥交换过程的实例：

- Alice 和 Bob 同意使用两个公共整数：**modulus p** 和 **base g**（其中 **p** 是 [(质数)prime](https://en.wikipedia.org/wiki/Prime_number)， **g** 是 [原始根模](https://en.wikipedia.org/wiki/Primitive_root_modulo_n) **p**)。
  - 例如，让 **p** = 23 和 **g** = 5。
  - 整数 **g** 和 **p** 是公共的，通常是源代码中的硬编码常量。
- Alice 选择一个 **秘密整数 a**（例如 **a** = 4），然后计算并向 Bob 发送数字 **A = ga mod p**。
  - 数字**A** 是公开的。 它是通过公共信道发送的，它的拦截不能泄露秘密指数**a**。
  - 在我们的例子中，我们有：**A** = 54 mod 23 = 4。
- Bob 选择一个 **秘密整数 b**（例如 **b** = 3），然后计算并发送给 Alice 数字 **B = gb mod p**。
  - 在我们的例子中，我们有：**B** = 53 mod 23 = 10
- Alice 计算 **s** = Ba mod p
  - 在我们的例子中：**s** = 104 mod 23 = **18**
- Bob 计算 s = Ab mod p
  - 在我们的例子中：**s** = 43 mod 23 = **18**
- Alice和Bob现在共享一个**秘密号码 s**
  - **s** = Ab mod p = Ba mod p = (ga)b mod p = (gb)a mod p = gab mod p = **18**
  - 共享密钥 **s** 无法从公开可用的数字 **A** 和 **B** 计算，因为无法有效计算秘密指数 **a** 和 **b**。

在最常见的 DHKE 实现中（遵循 [RFC 3526](https://tools.ietf.org/html/rfc3526)），基数是 **g** = **2** 和模数 **p ** 是一个很大的**质数**（1536 ... 8192 位）。

## DHKE 协议的安全性 Security of the DHKE Protocol

DHKE 协议基于 [Diffie-Hellman 问题](https://en.wikipedia.org/wiki/Diffie-Hellman_problem) 的实际难度，它是计算机科学中众所周知的 [DLP (离散对数问题)](https://en.wikipedia.org/wiki/Discrete_Logarithm_Problem_(DLP))，仍然没有有效的算法存在。

DHKE 通过不安全的公共（可嗅探）通道（例如通过电缆或通过空气波传播的信号）交换**非秘密整数序列**，但不会透露秘密交换的共享私钥.

再次提醒，DHKE 协议以其经典形式**易受到** [**中间人攻击**](https://en.wikipedia.org/wiki/Man-in-the -middle_attack)，黑客可以在其中拦截和修改双方之间交换的消息。

最后，请注意整数 **g**、**p**、**a** 和 **p** 通常是非常大的数字（1024、2048 或 4096 位甚至更大），这使得 [ **蛮力攻击**](https://en.wikipedia.org/wiki/Brute-force_attack) 毫无意义。

## DHKE - Live Example

As live example, you can play with this online DHKE tool: http://www.irongeek.com/diffie-hellman.php

![img](http://p0.qhimg.com/t0138058f56b5ccde7d.png)

## ECDH - 基于椭圆曲线的 Diffie-Hellman 密钥交换协议

The [**Elliptic-Curve Diffie–Hellman (ECDH)**](https://en.wikipedia.org/wiki/Elliptic-curve_Diffie–Hellman) 是一种匿名密钥协商协议，允许两方（每方都有一个**椭圆曲线公私密钥对**）在不安全的通道上建立共享秘密。

**ECDH** 是经典 **DHKE** 协议的变体，其中**模幂**计算被**椭圆曲线**计算取代，以提高安全性。 稍后我们将详细解释**椭圆曲线密码（ECC）**部分。

## 引用

- [Hellman_key_exchange](https://en.wikipedia.org/wiki/Diffie–Hellman_key_exchange)

- https://cryptobook.nakov.com

- https://en.wikipedia.org/wiki/Man-in-the-middle_attack

- [ECDH](https://en.wikipedia.org/wiki/Elliptic-curve_Diffie–Hellman)

- DLP (离散对数问题 : https://en.wikipedia.org/wiki/Discrete_Logarithm_Problem_(DLP