---
title: RSA的数学原理
tags:
  - 算法
categories:
  - 软件开发
date: 2019-06-23 14:16:48
---

## 发展历史

**密码学**的历史大致可以追溯到两千年前，相传古罗马名将凯撒大帝为了防止敌方截获情报，用密码传送情报。凯撒的做法很简单，就是对二十几个罗马字母建立一张对应表。这样，如果不知道密码本，即使截获一段信息也看不懂。

<!-- more -->
从凯撒大帝时代到上世纪70年代这段很长的时间里，密码学的发展非常的缓慢，因为设计者基本上靠经验，加密解密使用同一套算法，没有运用**数学原理**。

## 近代发展
- 在1976年以前，所有的加密方法都是同一种模式：**加密**、**解密**使用**同一种**算法。在交互数据的时候，彼此通信的双方就必须将规则告诉对方，否则没法解密。那么加密和解密的规则（简称**密钥**），它保护就显得尤其重要。传递密钥就成为了最大的隐患。这种加密方式被成为**对称加密算法（**symmetric encryption algorithm**）**。
- 1976年，两位美国计算机学家 **迪菲**（W.Diffie）、**赫尔曼**（ M.Hellman ） 提出了一种崭新构思，可以在不直接传递密钥的情况下，完成密钥交换。这被称为 **“迪菲赫尔曼密钥交换”** 算法。开创了密码学研究的新方向。
- 1977年三位麻省理工学院的数学家 [罗纳德](https://baike.baidu.com/item/罗纳德·李维斯特/700199)[·](https://baike.baidu.com/item/罗纳德·李维斯特/700199)[李维斯特](https://baike.baidu.com/item/罗纳德·李维斯特/700199)（Ron Rivest）、[阿迪](https://baike.baidu.com/item/阿迪·萨莫尔)[·](https://baike.baidu.com/item/阿迪·萨莫尔)[萨莫尔](https://baike.baidu.com/item/阿迪·萨莫尔)（Adi Shamir）和[伦纳德](https://baike.baidu.com/item/伦纳德·阿德曼/12575612)[·](https://baike.baidu.com/item/伦纳德·阿德曼/12575612)[阿德曼](https://baike.baidu.com/item/伦纳德·阿德曼/12575612)（Leonard Adleman）一起设计了一种算法，可以实现非对称加密。这个算法用他们三个人的名字命名，叫做[RSA算法](http://zh.wikipedia.org/zh-cn/RSA加密算法)。

## 迪菲-赫尔曼密钥交换
[迪菲-赫尔曼密钥交换](https://zh.wikipedia.org/wiki/%E8%BF%AA%E8%8F%B2-%E8%B5%AB%E7%88%BE%E6%9B%BC%E5%AF%86%E9%91%B0%E4%BA%A4%E6%8F%9B)（D-H）是一种**安全协议**。它可以让双方在完全没有对方任何预先信息的条件下**通过不安全信道创建起一个密钥**。

D-H 的基本原理其实就是下式：

> <font size=4 >$(m^d \bmod n)^e \bmod n = (m^e \bmod n)^d \bmod n$</font>



通过一个例子理解一下：
<font color=green>绿色 </font>表示非秘密信息, <font color=red>红色</font> 表示秘密信息：

1. 客户端 C 与服务器 S 协定 <font color=green>m</font> = <font color=green>3</font>，<font color=green>n</font> = <font color=green>17</font>；
2. S 产生一个随机数 <font color=red>d</font> = <font color=red>15</font>， 并通过公式 $ A\equiv m^d\bmod n$ 生成 <font color=green>A</font> = <font color=green>6</font> 发送给 C；
3. C 产生一个随机数 <font color=red>e</font> = <font color=red>13</font>,  并通过公式 $ B\equiv m^e\bmod n$ 生成 <font color=green>B</font> = <font color=green>12</font> 发送给 S；
4. S 拿到 <font color=green>B</font> 后, 计算:
   <font color=red>S_KEY</font> =  $12^{15} \bmod 17 = 10$
5. C 拿到 A 后计算：
   <font color=red>C_KEY</font> =  $6^{13} \bmod 17 = 10$
**<font color=red>C_KEY</font> = <font color=red>S_KEY</font> = <font color=red>10</font>  并没有在公共信道上传输，但是客户端与服务端拿到了同样的值，即完成了秘钥的交换。**

过程如下：
![](/images/WX20201224-165942@2x.png)

## RSA数学原理与概念
> 上世纪70年代产生的一种加密算法。其加密方式比较特殊，需要两个密钥：公开密钥简称**公钥（publickey）**和私有密钥简称**私钥（privatekey）**。**公钥加密，私钥解密；私钥加密，公钥解密**。
>
> 这种算法非常可靠，密钥越长，破解难度越高。根据公开资料，目前被破解的最长 RSA密钥是 768 个二进制位。因此可以认为，1024 位的 RSA 密钥基本安全，2048 位的密钥极其安全。

### 离散对数问题
RSA 算法的基础建立在[离散对数](https://baike.baidu.com/item/%E7%A6%BB%E6%95%A3%E5%AF%B9%E6%95%B0/4538780)问题求解的复杂度上，即加密容易而破解困难。如：
对于`g^N mod P = A` 的等式来说，正向求值是很容易的，但是想反向求出 `N` 则非常困难，而且困难程度随着 P 的增大指数上升。

### 原根
[原根](https://baike.baidu.com/item/%E5%8E%9F%E6%A0%B9)是数论中一个非常重要的概念，它在密码学中有着很广泛的应，原根从直观上非常好理解。
定义：**<font color=green>对于素数 P 存在整数 $g(1<g<P)$ 使 $g^i \bmod P(0<i<P) $ 的结果两两不同，即称 g 为 P 的原根。</font>**
记为：

> $g^i \bmod P \neq g^j \bmod P (i\neq j；0<i,j<P；1<g<P)$

如 ：$2^1 \bmod 5 \equiv 2$；$2^2 \bmod 5 \equiv 4$； $2^3 \bmod 5 \equiv 3$； $2^3 \bmod 5 \equiv 1$；
可见指数取值为{ 1、2、3、4} 时，结果取值也正好为 {1、2、3、4}
则称 **2 为 5 的原根**。

### 欧拉函数
思考：**<font color=green>任意给定正整数n，请问在小于等于n的正整数之中，有多少个与n构成互质关系？</font>**

上述问题的求解函数及为**欧拉函数**，记为**φ(n)**。
欧拉函数有如下特点：

- 当n是质数的时候，φ(n)=n-1。
- 如果n可以分解成两个互质的整数之积，如`n=A*B`则：`φ(A*B)=φ(A)φ(B)`

### 欧拉定理
定理：**<font color=green>如果两个正整数m和n互质，那么m的φ(n)次方减去1，可以被n整除。</font>**
记为：

> $m^{\phi (n)} \bmod n \equiv 1$

### 模反元素
定理：**<font color=green>如果两个正整数e和x互质，那么一定可以找到整数d，使得 ed-1 被 x 整除。</font>**
那么d就是e对于x的 **“模反元素”**

## RSA公式推导
<font color=#be7148 size=4>根据欧拉定理</font>
<font color=#be7148 size=4>对于 $m^{\phi (n)} \bmod n \equiv 1$ 且 gcd(m, n) = 1</font>
<font color=#be7148 size=4>有$m^{\phi (n) \times k} \bmod n \equiv 1^k$</font>
<font color=#be7148 size=4>两边同乘以 m</font>
<font color=#be7148 size=4>则 $m^{k*\phi (n)+1} \bmod n \equiv m$		①</font>
<font color=#be7148 size=4>根据模反元素</font>
<font color=#be7148 size=4>$e \times d \bmod x \equiv 1$</font>
<font color=#be7148 size=4>则存在 **k** 使  $ed \equiv kx + 1$		②</font>
<font color=#be7148 size=4>结合 ①、 ②则有</font>
<font color=#1aa size=4>**$m^{e*d}\bmod n \equiv m$**</font> (n为素数，且 m < n)

上式即为**非对称加密的基本公式。**

**`n`和`e`、`n`和`d `分别组成对应的公钥和私钥， `m` 即为需要加密的数据。**

### 加密解密过程
根据**迪菲-赫尔曼密钥交换** $m^{e*d}\bmod n \equiv m$ 可拆解为两部分，即：

> $m^e \bmod n \equiv C$ ---加密
>  $C^d\bmod n \equiv m$ ---解密

C 即为密文，可以安全的在网络上传输。

## RSA秘钥生成过程
由于 RSA 加密消耗较大，通常在通信的初期使用 RSA 加密需要下发的非对称加密的秘钥，后续通信使用该秘钥进行通信。
仍然使用 S 与 C 进行举例：

1. **S 随机生成两个质数**
   p=61, q=53
2. **通过 p、q 得到 n**
   $n=p\times q$
   ​	$=61\times 53=3233$
3. **通过欧拉函数计算  $\phi (n)$**
   $\phi (n)=(p-1)\times (q-1)$ 
   ​		 $=60\times 52=3120$
4. **随机选取整数 e 满足$(1<e<\phi (n))$且 e 与 $\phi(n)$ 互质**
   S 随机选择了17。（实际应用中，常常选择65537。）
5. **计算 e 对于 $\phi(n)$ 的模反元素 d**
   d=2753
6. **第六步，将n和e封装成公钥，n和d封装成私钥**
   上例中，n=3233，e=17，d=2753，所以公钥就是 **(3233,17)**，私钥就是 **(3233, 2753)**。
   实际应用中，公钥和私钥的数据都采用[ASN.1](http://zh.wikipedia.org/zh-cn/ASN.1)格式表达。
7. 下发公钥 **(3233,17)** 给 C， C 拿到后使用公钥进行加密通信
由上可知，一共用到了 6 个数字：`p、q、n、φ(n)、e、d`。
> 这里除了` n、e` 其他数字都是不公开的，由于**离散对数问题**与**大数的质因数分解问题**的复杂度是极高的，通过` n、e` 求出 `d、q `被认为是不可行的，所以 RSA 算法被认为是安全的加密算法。

当然非对称加密的算法并不仅有 RSA，如 [ECC](https://zh.wikipedia.org/wiki/%E6%A4%AD%E5%9C%86%E6%9B%B2%E7%BA%BF%E5%AF%86%E7%A0%81%E5%AD%A6)  被广泛认为是在给定密钥长度的情况下，最强大的非对称算法，可使用更短的秘钥达到与 RSA 相同的加密强度（192-bit 的强度即相当于 RSA 的 1024-bit）。我们**国密算法**中的 `SM2` 即采用的该种算法，有时间可以学习一下。



参考： 

[RSA算法原理（二）](https://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html)



