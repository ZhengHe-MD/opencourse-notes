---
title: Lecture 10 Hashing III
date: 2021-02-14 21:59:16
tags:
mathjax:
- true
---

# Lecture 10: Hashing  III: Open Addressing

{% youtube rvdJDijO2Ro %}

## 0. Overview

* Open Addressing, Probing Strategies
* Uniform Hashing, Analysis
* Cryptographic Hashing

## 1. Open Adressing

在 [Lecture 8](../Lecture-8-Hashing-I) 中我们提到过，解决 collision 的方法有两种：

* Chaining
* Open Addressing

本节介绍的就是第二种。发生 collision 时，open addressing 会按照某种固定的方式寻找 table 中的下一个 slot，每个 slot 只存放一个 key/item，因此 $ m \ge n$。在 open addressing 中，hash function 负责确定 probe (探测) 的 slots 顺序，而不是只指定单个 slot：
$$
h: \mathcal{U} \times \{0,1,...,m-1\} \rightarrow \{0,1,...,m-1\}
$$
由于最多 probe $(m-1)$ 次，因此其定义域是 $\mathcal{U} \times \{0,1,...,m-1\}$。换一种方式来理解，hash function 实际上为每个 key/item 确定了一个 $\{0,1,...,m-1\}$ 的 permutation (排列)。

> NOTE: 这里把 open addressing 和原始 hash function 合起来当作一个 hash function 实际上增加了学生的理解负担。open addressing 与 chaining 一样，是解决 collision 的策略，这种策略适用于任意 hash function。将 collision resolution strategy 和 hash function 看作是可以组合的两个部件更容易理解。不过本文将遵守原课程教授的方式。

### 1.1 Operations

#### 1.1.1 Insert

Insert key/item 时，不断 probe 直到找到空 slot 为止：

```python
for i in range(m):
    if T[h(k, i)] is None:
        T[h(k, i)] = (k, v)
        return
raise 'full'
```

举例如下：

![](./insert-example.png)

根据给定的某个 probe strategy，$k=496$ 的 probe 顺序是 `4, 6, 1, 5...`，在 slot 5 之前都发生 collision，因此 496 最终被插入到 slot 5 中。

#### 1.1.2 Search

Search key/item 时，按照相同的顺序 probe，直到找到空 slot 或者目标 key 为止：

```python
for i in range(m):
    if T[h(k, i)] is None:
        return None
    elif T[h(k, i)][0] == k:
        return T[h(k, i)]
return None
```

#### 1.1.3 Delete

Delete key 时，如果直接从 slot 中移除元素会导致后续的 search 无法找到正确的结果。回到 [1.1.1 insert](./#insert) 一节中的例子，如果我们将 $k = 586$ 从 slot 1 中移除，`search(496)` 将失效。合理的做法是利用某个特殊标记，如 "DeleteMe"，insert 认为它是空 slot，而 search 不认为。

### 1.2 Probing Strategies

#### 1.2.1 Linear Probing

顾名思义，linear probing 就是从初始位置开始往后顺序遍历，假设 $h'(k)$ 为普通 hash function：
$$
h(k, i) = (h'(k) + i)\space mod\space m
$$
一个很好的比喻是  linear probing 就像在停车场上找车位：当车比较多时，车主会按顺序遍历车位，直到发现空的停车位位置。linear probing 的问题在于 clustering (聚集效应)，当 hash table 中逐渐形成连续被占用的 slots 时，会影响新的 key/item 插入性能，同时后者将进一步产生负向反馈，进一步扩大 clustering 的影响，如下图所示：

![](./primary-clustering.png)

可以证明，当 $0.01 \lt \alpha \lt 0.99$ 时，clusters 的大小为 $\theta(logn)$。

#### 1.2.2 Double Hashing

利用 2 个 hash function，$h_1(k)$ 和 $h_2(k)$：
$$
h(k, i) = (h_1(k) + i \cdot h_2(k)) \space mod \space m
$$
在什么情况下  $h_1(k) + i \cdot h_2(k), \space i = \{1,2,...,m-1\}$ 会遍历所有 slots？假设存在 $i \ne j$ 且 $i, j \in [1,m-1]$，满足：
$$
h_1(k) + i \cdot h_2(k) \space mod \space m = h_1(k) + j \cdot h_2(k) \space mod \space m
$$
消元后得到：
$$
(i-j)\cdot h_2(k) \space mod \space m = 0
$$


如果 $h_2(k)$ 与 $m$ 互质，则只能 $i = j$。举一个具体的例子，假设 $m = 2^r$，让 $h_2(k)$ 返回的永远是奇数即可。

##### Complexity Analysis

假设使用 open addressing 策略往大小为 $m$ 的 table 中插入 $n$ 个 keys/items 后，在 SUHA 下，当 $\alpha = \frac{n}{m} \lt 1$ 时，下一次 search/insert/delete 的时间复杂度为 $O(\frac{1}{1-\alpha})$。

证明如下：假设我们想要继续往 table 中插入 $k$，且 $k$ 尚不存在于 table 中。

* 首次 probe 成功的概率为：$\frac{m-n}{m}$ (在 SUHA 下)，记为 $p$
* 首次 probe 失败，第二次 probe 成功的概率为：$\frac{m-n}{m-1} \gt \frac{m-n}{m} = p$
* 前两次 probe 都失败，第三次 probe 成功的概率为：$\frac{m-n}{m-2} \gt \frac{m-n}{m} = p$
* ...

每次 probe 的成功率都至少为 $p$，尝试次数的期望为：
$$
\frac{1}{p} = \frac{1}{1-\alpha}
$$
以此类推，search 和 delete 的时间复杂度同样为 $O(\frac{1}{1-\alpha})$，$k$ 在插入前已经存在于 table 中也同理可证。

### 1.4 Open Addressing vs. Chaining

由于 clustering 现象的存在且实现中没有指针寻址，open addressing 对缓存更友好，但同样由于 clustering 现象的存在，open addresing 对 hash functions 的选择比较敏感，且其 $\alpha$ 不能过大 (通常要小于 70%)；chaining 与 open addressing 正好相反。

## 2. Cryptographic Hashing

cryptographic hash function 可以将任意长度的数据稳定地 (deterministic) 转化成固定大小的 hash value，即 bit string，意外或恶意修改数据会造成 hash value 的大幅度变化。通常称待加密的数据为 message，加密后的 hash value 为 (message) digest。

### 2.1 Desirable Properties

假设 *d* 是 hash value 的 bit 数量，通常安全性较高的 $d \ge 160$，那么理想中的 cryptographic hash function 应该具备以下特性：

* **One-Way (OW)** (pre-image resistance)： 通过 digest 找到一条 message 计算上不可行。即给定 $ y \in_{R} \{0, 1\}^d$，找到 $x$ 满足 $h(x) = y$

* **Collision-resistance (CR)**：找到存在 collision 的两条 message 计算上不可行。即找到 $x \ne x'$ 且 $h(x) = h(x')$

* **Target collision-resistance (TCR)** (2nd pre-image resistance)：给定条 message，找到另一条与它冲突的 message' 计算上不可行。即给定 $x$，找到 $x'$ 满足 $h(x) = h(x')$

CR 是 TCR 的充分非必要条件，而 OW 与 CR/TCR 之间没有直接关系。

### 2.2 Applications

* Password storage：在数据库存储 $h(PW)$，而不是 $PW$，当用户输入 $PW'$ 验证时，对比 $h(PW')$ 与 $h(PW)$ 即可。在该场景下使用的 cryptographic hash function 需满足 OW。攻击者通常无法获取 $PW$ 以及 $PW'$，因此 TCR 和 CR 并不是必须要求。
* File modification detector：对于每个文件 $F$，保存 $h(F)$。检查 $F$ 在传输过程中是否被篡改，只需要重新计算 $h(F)$ 对比即可。常常用在文件下载的校验过程。在该场景下使用的 cryptographic hash function 需满足 TCR，防止攻击者篡改 $F$ 的同时保持 $h(F)$ 不变。
* Digital signatures：用于非对称加密中。假设 Alice 拥有一个公钥 $PK_A$ 和一个私钥 $SK_A$，Alice 可以对 message $M$ 使用私钥加密，得到 $\sigma = sign(SK_A, M)$，Bob 知道 Alice 的公钥，可以用它来验证 $\sigma$ 中的消息 $M$ 是否来自于 Alice，即 $verify(M, \sigma, PK_A)$。攻击者想要伪装成 Alice，伪造 $M'$。通常对于数据量比较大的 $M$，我们会选择先做一次散列，即 $\sigma = sign(SK_A, h(M))$，此时 $h$ 需满足 TCR，因为我们不希望攻击者能够先让 Alice sign $x$，然后对外声称 Alice 发送的是消息 $x'$，即 $h(x) = h(x')$。

### 2.3 Implementations

cryptographic hash function 的核心是一个特殊的函数，其输入是 2 个固定大小的数据块，输出是对应的 hash value，如下图所示：

![](./hash_function_structure.jpg)

数据块的大小取决于算法本身，通常在 128 bits 到 512 bits 之间。上述函数是整个 cryptographic hash function 的最小计算单元，完整的加密过程是将原始待加密数据切分成固定大小，然后执行多轮次的上述计算，将前一次的结果与下一个数据块共同作为下一轮计算的输入，如下图所示：

![](./hashing_algorithm.jpg)

我们称这个过程为 avalanche effect of hashing (哈希雪崩效应)，即前面的数据块会不断影响后续的所有轮次加密结果。avalanche effect 使得一旦两条消息存在一点不同 (single bit)，digest 就会显著不同。

综上所述，cryptographic hash function 需要决定如何切分 message，如何将前一次处理的结果输入到后一次计算中，在这两个设计点上的不同决定影响着函数的功能。

## References

* MIT 6.006 lecture notes, [notes typed](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-006-introduction-to-algorithms-fall-2011/lecture-videos/MIT6_006F11_lec10.pdf), [video](https://www.youtube.com/watch?v=rvdJDijO2Ro&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb&index=10&t=2605s)
* CLRS: 11.4, 11.3.3, 11.5
* [Tutorialspoint: cryptography hash functions](https://www.tutorialspoint.com/cryptography/cryptography_hash_functions.htm)
* [Cryptographic Hash-Function Basics: Deginitions, Implications, and Separations for Preimage Resistance, Second-Preimage Resistance and Collision Resistance](https://people.csail.mit.edu/alinush/6.857-spring-2015/papers/rogaway-hashes.pdf)

## TODO

* clusters 大小证明