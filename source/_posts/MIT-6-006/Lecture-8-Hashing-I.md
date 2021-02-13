---
title: MIT-6.006-Lecture-8-Hashing-I
date: 2021-02-13 14:26:28
tags:
mathjax:
- true
---

# Lecture 8: Hashing-I

## 0. Overview

* Dictionary problem
* Motivation
* Prehashing
* Chaining
* Simple uniform hashing
* "Good" hash functions

## 1. Dictionary Problem

dictionary、map、associated array 以及 symbol table 都指同一个 ADT (Abstract Data Type)，它维护者一个 items 集合，每个 item 对应唯一的 key，dictionary 需要满足：

* `insert(item)`：将 item 添加到集合中
* `delete(item)`：从集合中删除 item
* `search(key)`：在集合中寻找 key 对应的 item，如果存在就返回

其中，不同 items 对应的 key 不同，如果插入多个 key 相同的 items，则只有最后一个会被保留。dictionary 对 items 的顺序关系没有规定，利用 Balanced BSTs 可以解决 dictionary problem，其 insert、delete、search 的时间复杂度都是 $ O(lgn) $，而我们的目标是将时间复杂度降到 $O(1)$。

## 2. Motivation

Dictionaries 是计算机科学中应用范围最广的数据的结构之一。

## 3. How do we solve the dictionary problem?

### 3.1 Associative list/linked list

最简单的方法就是将所有 items 存储在 linked list 中，但它的时间复杂度不满足要求：

| Method       | Average-case complexity | Worst-case complexity |
| ------------ | ----------------------- | --------------------- |
| insert(item) | $O(1)$                  | $O(1)$                |
| delete(item) | $O(n)$                  | $O(n)$                |
| search(key)  | $O(n)$                  | $O(n)$                |

在集合很小的情况下，这种实现方案性能不错，但随着数据量增加，它的 delete、search 时间复杂度也将线性增加。

### 3.2 Direct access table/array

linked list 的问题在于不支持 random access，那么为什么不用 direct access table 呢？如果 key 恰好是非负数，我们就能将 delete、search 的时间复杂度降到 $O(1)$。这时我们面临两个问题：

1. keys 取值范围是非负整数 (或者另外用一个 array 保存 keys 为负整数的 items)
2. keys 的取值范围不宜过大，过大意味着存储单个 key 所需空间变大，同时 array 变大、变稀疏 (利用率变低)

#### 3.2.1 Solution to 1: "prehash" keys to integers

将每个 items 的 key 先转化成非负整数，这样既可以解决负整数 key 问题，也能解决非整数类型的 key 问题。理论上，只要 items 集合是可数的 (countable)，这一步肯定能实现。

#### 3.2.2 Solution to 2: hashing

如果能将 keys 的取值 $\mathcal{U}$ 映射到更小的、可以接受的区间 $[0, m)$ 内。假设 keys/items 的数量 (不是取值范围) 为 $n$，完美情况下 $m \approx n$。我们称实现映射关系的函数为 hash function，即：
$$
h:\space \mathcal{U} \rightarrow \{0, 1, ..., m-1\}
$$
示意图如下图所示：

![](./mapping-keys-to-a-table.png)

由于 $|\mathcal{U}| \gg m$，在不知道 $n$ 的大小或无法找到通用的完美 hash function 时，必然会出现 collisions：即不同 key 被散列到同一个位置上。解决 collision 的方法有两种：

* Chaining
* Open addressing：在 [Lecture 10](TODO) 中介绍

### 3.3 Chaining

Chaining 就是将散列到同一个位置的 items 用 linked list 串联：

![](./chaining.png)

由于 search、delete 需要遍历 key/item 对应的 linked list，因此最差情况就是所有 keys/items 都被散列到同一个 slot，导致 search、delete 的时间复杂度为 $\theta(n)$。

#### 3.3.1 Simple Uniform Hashing

什么样的 hash function 才是个优秀的 hash function 呢？一个常用的定义/假设就是：SUHA (Simple Uniform Hashing Assumption)：**同一个 key 被散列到每个 slot 的概率相等，且与其它 keys 的散列结果相互独立**。

#### 3.3.2 Performance

在 SUHA 下，我们定义：

* $n$: table 中存储的 keys/items 总数
* $m$: table 大小，即 slot 数量
* $\alpha$：负载因子，即 $\frac{n}{m}$，即每个 slot 上的 chain 长度的期望

于是 search、delete 的时间复杂度为 $\theta(1 + \alpha)$，如果 $\alpha = O(1)$，即 $m = \Omega(n)$，那么该 dictionary 实现的各方法时间复杂度都为 $O(1)$。

### 3.5 Hash Functions

本节介绍 3 种在实践中能达到上述性能的 hash functions。实践中达到指的是在大多数 keys/items 集合上表现符合要求，理论上满足 SUHA 的只有 Universal Hashing。

#### 3.5.1 Division Method

Division method 就是选择一个足够大的数 $M$：
$$
h(k) = k\space mod\space m
$$
实践中，当 $M$ 是素数且与 2 或 10 的 $n$ 次幂不接近时效果最好。这里选素数的原因是可以将其整除，即 $u\space mod\space M = 0$ 的数较少；不与 2 的 $n$ 次幂接近的原因在于：数字在计算机中用二进制存储，如果 $M$ 接近 2 的 $n$ 次幂，$mod$ 操作实际使用的 bits 过少；不与 10 的 $n$ 次幂接近的原因在于：数字在人类社会中常常用十进制记录，如果 $M$ 接近 10 的 $n$ 次幂，$mod$ 操作实际使用的 digits 过少。bits/digits 过少意味着散列过程中使用的信息量少，散列效果就很难达到均匀。

Division method 在许多 keys/items 集合上能取得不错的散列效果。但 division 在现代计算机架构上的速度是 multiply 的 $\frac{1}{10}$；division method 的另一缺点在于连续的 keys 会被散列到连续的 slots 上，在一些场景下，如 open addressing，并非我们所愿。

#### 3.5.2 Multiplication Method

Multiplication method 的核心计算过程如下：
$$
h(k) = [(a \cdot k)\space mod \space 2^{\omega}] \gg (\omega - r)
$$
其中 $\omega$ 是 machine word 的 bits 数，如 32、64； $a$ 是区间 $[0, 2^{\omega})$ 内的随机数，$k$ 是待散列的 key，其取值范围与 $a$ 相同；$r$ 满足 $m = 2^r$。 一种理解 multiplication method 的直觉如下图所示：

![](./multiplication-method.png)

1. 用类似乘法笔算的方式理解：$a \cdot k$ 表示将 $k$ 按照 $a$ 的 set bits 向左分别分别平移若干次

2. 将多个平移版本的 $k$ 加总，取前 $\omega$ 位 bits
3. 取第 2 步中得到的数字的高 $r$ 位 (higher order $r$ bits)，即为哈希值

从图中可以看出：高 $r$ 位 bits 利用了输入值 $k$ 最多最丰富的信息，因此它的随机程度也相对较高。在实践中，当 $a$ 是奇数 (odd) 、位于区间 $(2^{\omega - 1}, 2^{\omega})$、且离 $2^{\omega-1}$ 和 $2^{\omega}$ 距离较远时，散列效果最好。相较于 division method，multiplication method 中只需要乘法和位运算即可搞定，性能好于前者。

#### 3.5.3 Universal Hashing

详情看[这里](../../MIT-6-046J/Lecture-8-Hashing)

## 参考文献

* wikipedia: [SUHA](https://en.wikipedia.org/wiki/SUHA_(computer_science))、[Hash function](https://en.wikipedia.org/wiki/Hash_function)、[Universal hashing](https://en.wikipedia.org/wiki/Universal_hashing)
* MIT 6.006 lecture notes, [handwritten](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-006-introduction-to-algorithms-fall-2011/lecture-videos/MIT6_006F11_lec08_orig.pdf), [typed notes](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-006-introduction-to-algorithms-fall-2011/lecture-videos/MIT6_006F11_lec08.pdf), [video](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-006-introduction-to-algorithms-fall-2011/lecture-videos/lecture-8-hashing-with-chaining/)
* CSPC 131, Data Structures, [Multiplicative Hashing](https://www.youtube.com/watch?v=BmKMzAt2Gjc&list=PLqODvuetmZCT1sN3Vd0vwvHeJPa3vSrC9&index=68)

## TODO

* Lecture 10 的链接





