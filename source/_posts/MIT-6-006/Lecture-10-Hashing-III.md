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



## References

* MIT 6.006 lecture notes, [notes typed](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-006-introduction-to-algorithms-fall-2011/lecture-videos/MIT6_006F11_lec10.pdf), [video](https://www.youtube.com/watch?v=rvdJDijO2Ro&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb&index=10&t=2605s)
* CLRS: 11.4, 11.3.3, 11.5

## TODO

* clusters 大小证明