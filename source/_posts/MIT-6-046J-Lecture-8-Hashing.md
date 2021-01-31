---
title: MIT-6.046J-Lecture-8-Hashing
date: 2021-01-31 13:06:11
tags:
---

# Lecture 8: Hashing

> 冷知识：英文单词 'hash' 的本身含义为 "把...切成小块"，该词来源于法语中的 'hacher'，意思相似。而 'hatcher' 来源于古法语 'hache'，意为斧头。

本节课主要分为 3 个部分：

1. 复习 6.006 课程中的概念：dictionaries, chaining, simple uniform
2. Universal hashing
3. Perfect hashing

## 1. Review

Dictionary 是一种抽象数据结构 (ADT)，维护着一个集合，我们称集合中的元素为 item，集合中的元素由键值对 (key, value) 构成，它支持以下操作：

* `insert(item)`：将 item 添加到集合中
* `delete(item)`：从集合中删除 item
* `search(key)`：在集合中寻找 key 对应的 item，如果存在就返回

Dictionary 不维护数据的顺序关系。

### Hashing from 6.006

在 6.006 中介绍的 Dictionary 实现的时空复杂度如下：

* 时间复杂度：`insert`、`delete` 和 `search` 都是 $O(1)$
* 空间复杂度：$O(n)$

具体来说，假设：

* $u$ 表示 key 的可能取值总量
* $n$ 表示在 hash table 中的 keys/items 总量
* $m$ 表示在 hash table 中的坑位 (slots) 总量

我们可以通过 hashing with chaining + table doubling 来实现 $\theta{(1 + \alpha)}$ 的各操作事件复杂度，其中 $\alpha = \frac{n}{m}$ 为 load factor。在推导过程中，我们做了一个很强的假设：我们使用的 hash function 满足 simple uniform hashing assumption (SUHA)，即：
$$
\underset{k_1 \ne k_2}{\mathbb{P}}{\{h(k_1) = h(k_2)\}} = \frac{1}{m}
$$
即 hash function 的散列过程是随机的，但我们知道 hash function 必须是稳定的，即输入相同时输出相同。随机性和稳定性是对立的，本节课的目的就是去除 simple uniform hashing 的假设，来看我们如何在理论上达到相同的时空复杂度。

## 2. Universal Hashing

理解 universal hashing 的直觉就是：**将随机性加在 hash functions 的选择上**。即从一个 hash functions 集合 $\mathcal{H}$ 中随机选择一个 (记为 $h$)，其中 $\mathcal{H}$ 是一个 *universal hashing family*，后者满足：
$$
\underset{h \in \mathcal{H}}{\mathbb{P}}{\{h(k) = h(k^{'})\}} \le \frac{1}{m} for \space all \space k \ne k^{'}
$$
即从 $\mathcal{H}$ 中随机选一个 hash function，在该情况下两个不同元素被散列到同一个位置上的概率小于 $\frac{1}{m}$。到此你可能像我一样有疑问，这样随机选取 hash function 不就又丧失了稳定性吗？**如果每次散列都要从 $\mathcal{H}$ 中随机取一个 hash function，那如何保证第二次遇到相同的元素还会被散列到相同的位置？**实际上，对于同一个 hash table，随机选取 hash function 的过程只会执行一次，即对于这张 table 所有 keys 都使用同一个的 $h$，直到 hash table handler 因某种条件触发 rehash 时，如 table doubling，才会重新选择 hash function。而 rehash 需要将当前所有 keys/items 取出，重新散列，详情见这里的[讨论](https://math.stackexchange.com/questions/103107/universal-hashing)。

接下来，我们要先证明 universal hashing 能取得与 SUHA 相同的时空复杂度，然后再找到这样的 hashing family。

> 定理：从 $u$ 中随机取任意不同的 $n$ 个 key，且为每个 key 随机从某 universal hashing family $\mathcal{H}$ 中取一个 hash function，记为 $h$，被散列到某一个 slot 中的 keys/items 数量预期小于 $1+\alpha$，即：
> $$
> \mathbb{E}[\#keys\space colliding\space in\space a\space slot] \le 1 + \alpha,\space where\space \alpha = \frac{n}{m}
> $$

当我们要证明某事件发生次数与某阈值的关系时，一种常用技巧就是 indicator variable。假设所有在 hash table 中的 keys 分别为 $k_1, k_2,..., k_n$，我们设 indicator variable：
$$
I_{i,j} = 
\begin{cases}
	1 & if\space h(k_i) = h(k_j) \\
	0 & otherwise
\end{cases}
$$
那么：
$$
\begin{align*}
\mathbb{E}[I_{i,j}] &= \mathbb{P}{\{I_{i,j} = 1\}} \\
                    &= \mathbb{P}{\{h(k_i) = h(k_j)\}} \\
                    &\le \frac{1}{m} \space for \space any \space j \ne i
\end{align*}
$$
因此：
$$
\begin{align*}
\mathbb{E}[\#keys\space hashing\space to\space the\space same \space slot \space as \space k_i] &= \mathbb{E}[\sum_{j=i}^{n}{I_{i,j}}] \\
 &= \sum_{j\ne i}^{n}\mathbb{E}[{I_{i,j}}] + \mathbb{E}[{I_{i,i}}] \\
 &= \frac{n}{m} + 1
\end{align*}
$$
以上证明了 universal hashing 能够取得与 SUHA 相同的时间复杂度。接下来举几个 universal hashing family 的例子：

### Example #1: all hash functions

将所有可能的 hash functions 组成一个集合：
$$
\mathcal{H} = {\{all\space hash\space functions\space h: \{0,1,...,u-1\} \rightarrow \{0,1,...,m-1\}\}}
$$
显然，$\mathcal{H}$ 本身是 universal hashing family，因为对于任意一个 key 来说，从所有可能的 hash functions 中随机取一个，该 key 被散列到某个 slot 上的概率为 $\frac{1}{m}$。但 $\mathcal{H}$ 只存在理论价值，不具备实践意义：

1. 存储某个 $h \in \mathcal{H}$ 需要花费 $log(m^u) = ulog(m) \gg n$ bits。而且为了存储这个 hash function，你需要使用一个 hash table 😂，这就来到了鸡生蛋还是蛋生鸡的问题
2. 生成某个 $h \in \mathcal{H}$ 的时间复杂度为 $\Omega(u)$ ，因为你需要为 $u$ 个值生成对应的散列位置

### Example #2: dot-product hash family

假设：

* $m$ 是一个素数
* $u = m^r$，其中 $r$ 是一个整数

在实践中，你总能够将 $m$ 和 $u$ 向上取到满足上述条件的数。现在我们将所有的 keys 转化成 $m$ 进制的数，即 $k = \langle k_0,k_1,...,k_{r-1} \rangle $，取某个常数 $a = \langle a_0,a_1,...,a_{r-1} \rangle $，定义 hash function：
$$
\begin{align*}
h_a{(k)} &= a \cdot k \space mod \space m \\
         &= \sum_{i=0}^{r-1} a_i k_i \space mod \space m
\end{align*}
$$
于是得到 dot-product hash family $\mathcal{H} = \{h_a | a \in \{ {0, 1,..., u-1} \} \}$。

存储任意一个 $h_a \in \mathcal{H}$ 只需要存储一个 key，即 $a$ 本身。在 word RAM model 计算模型下，简单的数据 (如 keys) 都可以放入一个 machine word 中，这个计算模型的硬件实现能保证针对 machine words 的操作能在 $O(1)$ 的复杂度下完成，因此计算 $h_a{(k)}$ 的时间复杂度为 $O(1)$。

> 定理：dot-product hash family $\mathcal{H}$ 是 universal hashing family

证明：对于任意两个不同的 keys，$k \ne k^{'}$。它们在 $m$ 进制下至少有某个位 ($d$) 不同，假设 $k_d \ne k_{d^{'}}$，$not \space d = \{0,1,...r-1\} \backslash \{d\}$，我们可以推导：
$$
\begin{align*}
\underset{a}{\mathbb{P}}\{h_a(k) = h_a(k^{'})\} &= 
  \underset{a}{\mathbb{P}} \{\sum_{i=0}^{r-1}{a_i k_i} = \sum_{i=0}^{r-1}{a_i k_i^{'}}\space (mod\space m) \} \tag{1} \\
                                                &=
  \underset{a}{\mathbb{P}} \{\sum_{i \ne d}a_i k_i + a_d k_d = \sum_{i \ne d}a_i k_i + a_d k_d \space (mod\space m) \} \tag{2} \\
                                                &=
  \underset{a}{\mathbb{P}} \{\sum_{i \ne d} a_i(k_i - k_i^{'}) + a_d(k_d - k_d^{'}) = 0 \space (mod\space m) \} \tag{3} \\
                                                &=
  \underset{a}{\mathbb{P}} \{a_d = -(k_d - k_d^{'})^{-1} \sum_{i \ne d} a_i (k_i - k_i^{'}) (mod \space m) \} \tag{4} \\
                                                &=
  \sum_{x}{\mathbb{P}\{a_{not \space d} = x\} \underset{a_d}{\mathbb{P}}\{a_d = f(k,k^{'},x)\}} \tag{5} \\
                                                &=
  \underset{a_{not\space d}}{\mathbb{E}}[\underset{a_d}{\mathbb{P}} = f(k, k', a_{not\space d})] \tag{6} \\
                                                &=
  \underset{a_{not\space d}}{\mathbb{E}}[\frac{1}{m}] \tag{7} \\
                                                &= \frac{1}{m}
\end{align*}
$$
其中：

* $(3) \rightarrow (4)$ ：由于 $m$ 是素数，$\mathbb{Z}_m$ 中存在模乘逆元 (modular multiplicative inverse)
* $(4) \rightarrow (5)$：由于 $a$ 是 $m$ 进制下 $[0, u-1]$ 区间内的随机数，因此可以认为 $a$ 的任意位都是 $[0,m-1]$ 区间内的随机数，且任意位之间的取值相互独立，于是我们可以将 $not \space d$ 位的概率和 $d$ 位的概率分开考虑。同时 $-(k_d - k_d^{'})^{-1} \sum_{i \ne d} a_i (k_i - k_i^{'})$ 只与 $k$，$k^{'}$ 以及所有 $not \space d$ 取值相关，与 $a_d$ 无关，其取值为 $[0, m-1]$ 区间内的某个值。
* $(5) \rightarrow (6)$：由于 $a$ 是 $m$ 进制下 $[0, u-1]$ 区间内的随机数，$a_d$ 取$[0, m-1]$ 区间内的某个值的概率一定为 $\frac{1}{m}$
* $(6) \rightarrow (7)$：一个常数的期望就是它本身

至此，我们证明了 dot-product hash family 是 universal hashing family。

### Example #3

CLRS 中还介绍了另一个 universal hash family：首先选择一个素数 $p \ge u$，定义 $h_{ab}{k} = [(ak + b) mod \space p] \space mod \space m$，则 $\mathcal{H} = \{h_{ab} | a,b \in {0,1,...,u-1}\}$ 是一个 universal hashing famliy。

## 3. Perfect Hashing

如果已知要插入 hash table 的所有 keys，数量不变，且该 hash table 只需要支持 `search(key)`，我们能做得更好吗？这便是 *static dictionary problem*。

利用 universal hashing family，我们能做到平均情况下 (average case) ，时间复杂度 $O(1)$，空间复杂度 $O(n)$，但本节要介绍的 perfect hashing 能做到最坏情况下 (worst case)，时间复杂度 $O(1)$，空间复杂度 $O(n)$，但整个 hash table 构建的时间复杂度为 polynomial (w.h.p)。

(TODO)

## 参考资料

* Course [home page](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-046j-design-and-analysis-of-algorithms-spring-2015/index.htm)
* Lecture notes, [pdf](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-046j-design-and-analysis-of-algorithms-spring-2015/lecture-notes/MIT6_046JS15_lec08.pdf), [hand written](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-046j-design-and-analysis-of-algorithms-spring-2015/lecture-notes/MIT6_046JS15_writtenlec8.pdf)
* Lecture [video](https://www.youtube.com/watch?v=z0lJ2k0sl1g&t=2s)

* Birthday Paradox
  * [Better Explained: Understanding the Birthday Paradox](https://betterexplained.com/articles/understanding-the-birthday-paradox/)
  * [wikipedia: birthday problem](https://en.wikipedia.org/wiki/Birthday_problem)
* [SUHA](https://en.wikipedia.org/wiki/SUHA_(computer_science))
* [StackExchange-Mathematics: Universal Hashing](https://math.stackexchange.com/questions/103107/universal-hashing)
* word RAM model
  * [wikipedia: word in computer science](https://en.wikipedia.org/wiki/Word_(computer_architecture))
  * [wikipedia: Word RAM](https://en.wikipedia.org/wiki/Word_RAM)
* [CLRS](https://en.wikipedia.org/wiki/Introduction_to_Algorithms)
* [perfect hashing paper: Storing a Sparse Table with O(1) Worst Case Access Time](https://cs.dartmouth.edu/~ac/Teach/CS105-Winter05/Handouts/fks-perfecthash.pdf)

