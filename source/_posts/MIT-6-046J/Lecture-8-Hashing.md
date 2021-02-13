---
title: MIT-6.046J-Lecture-8-Hashing
date: 2021-01-31 13:06:11
mathjax: true
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
\underset{k_1 \ne k_2}{ \mathbb{P} } {\{ h(k_1) = h(k_2) \}} = \frac{1}{m}
$$
即 hash function 的散列过程是随机的，但我们知道 hash function 必须是稳定的，即输入相同时输出相同。随机性和稳定性是对立的，本节课的目的就是去除 SUHA，来看我们如何在理论上达到相同的时空复杂度。

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
\mathbb{E}[\#keys\space hashing\space to\space the\space same \space slot \space as \space k_i] &= \mathbb{E}[\sum_{j=1}^{n}{I_{i,j}}] \\
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

CLRS 中还介绍了另一个 universal hash family：multiplicative hashing。首先选择一个素数 $p \ge u$，定义 $h_{ab}(k) = [(ak + b) mod \space p] \space mod \space m$，则 $\mathcal{H} = \{h_{ab} | a \in [1,u-1],b \in [0,u-1]\}$ 是一个 universal hashing famliy。

multiplicative hashing 的直觉与轮盘赌 (Roulette Wheel) 的过程类似：

<img src="./roulette-wheel.jpg" style="width: 480px"/>

如果轮盘转动很多圈，那么最终小球落入的地方将变得不可预测，即随机。通常 $a$、$b$ 是比较大的整数，$a \cdot k + b$ 则是一个更大的数。$(a \cdot k + b)\space mod \space p$ 表示转动的总距离，$\lfloor [(a \cdot k + b)\space mod \space p]\space / \space m \rfloor$ 表示转动的总圈数；$[(a \cdot k + b)\space mod \space p]\space mod \space m$ 表示最后一圈转动的角度。

证明：对于任意两个不同的 keys，$k \ne k^{'}$，如果发生 collision，即 $h(x) = h(y)$，那么存在某个整数 $i \in \lfloor 0, (p-1)/m \rfloor$，使得：
$$
a \cdot k + b \equiv a \cdot k^{'} + b + im \space (mod\space p)
$$
由于 $k \ne k^{'}$，$k - k^{'} \ne 0$，那么 $(k - k^{'})$ 存在模乘逆元，推导得到：
$$
a \equiv im(x-y)^{-1} \space (mod \space p)
$$
由于 $a \in [1, p-1]$，等式左边有 $p-1$ 种取值；对于固定的 $p$，等式右边最多存在 $\lfloor (p-1)/m \rfloor$ 种非 0 取值，于是有：
$$
\begin{align*}
	\mathbb{P}[a \equiv im(x-y)^{-1}(mod\space p)] &= \frac{\lfloor (p-1)/m \rfloor}{p-1} \\
	                                               &\le \frac{(p-1)/m}{p-1} \\
	                                               &= \frac{1}{m}
\end{align*}
$$
至此，我们证明了 multiplicative hashing family 是 universal hashing family。

## 3. Perfect Hashing

如果已知要插入 hash table 的所有 keys，数量不变，且该 hash table 只需要支持 `search(key)`，我们能做得更好吗？这便是 *static dictionary problem*。

利用 universal hashing family，我们能做到平均情况下 (average case) ，时间复杂度 $O(1)$，空间复杂度 $O(n)$，但本节要介绍的 perfect hashing 能做到最坏情况下 (worst case)，时间复杂度 $O(1)$，空间复杂度 $O(n)$，但整个 hash table 构建的时间复杂度为 polynomial (w.h.p)，即 $O(n^c)$。

理解 perfect hashing 的直觉就是：使用 2-level hashing，即将 hashing with chaining 中的 chain (链表) 替换成另一个 hash table，且保证后者中不出现 hash collision，同时保证空间复杂度不变。

算法如下：

**Step 1：**从一个 universal hash family 中随机选取一个 hash function $h_1: \{0,1,...,u-1\} \rightarrow  \{0,1,...,m-1\}$，取 $m = \theta{(n)}$，即 $m$ 为最接近 $n$ 的素数，利用 $h1$ 将所有 keys/items 散列到不同的 slots 中，用链表串联。

**Step 2：**针对任意 slot $j \in \{0,1,...,m-1\}$，记散列到 slot $j$ 中的 keys/items 总数为 $l_j$ ，即 $l_j = |\{i | h_1(k_i) = j\}|$。继续从 universal hash family 中随机选取一个 hash function $h_{2,j}: \{0,1,...,u-1\} \rightarrow \{0,1,...m_j\}$ ，其中 $l_j^2 \le m_j \le O(l_j^2)$，即取 $m_j$ 为大于或等于 $l_j^2$ 最近的素数，将 slot $j$ 中的链表替换成 $h_{2,j}$ 的 hash table。

如下图所示：

![](./2-level-hashing.png)

此时算法的空间复杂度为 $O(n + \sum_{j=0}^{m-1}l_j^2)$，为了使其减小到 $O(n)$，我们还需要两个步骤：

**Step 1.5：**如果 $\sum_{j=0}^{m-1}l_j^2 > cn$，$c$ 为某设定好的常数，则重新执行 *Step 1*。

**Step 2.5：**当某 slot $j$ 中出现 $h_{2,j}(k_i) = h_{2,j}(k_i^{'})$，其中 $i \ne i^{'}$，则重新随机选取 $h_{2,j}$。

上述两个步骤保证在第二层 hash table 中不存在 hash collision，且空间复杂度为 $O(n)$，因此 `search` 在最坏情况下的时间复杂度为 $O(1)$。现在我们分析一下两层 hash table 的构建时间复杂度，**Step 1** 和 **Step 2** 的空间复杂度都是 $O(n)$，对于 **Step 2.5** 来说：
$$
\begin{align*}
	\underset{h_{2,j}}{\mathbb{P}} \{h_{2,j}(k_i) = h_{2,j}(k_i^{'})\} &\le \underset{i \ne i'}{\sum} \underset{h_{2,j}}{\mathbb{P}} \{h_{2,j}(k_i) = h_{2,j}(k_i^{'})\} \tag{1} \\
	& \le \binom{l_j}{2} \cdot \frac{1}{l_j^2} \tag{2} \\
	& \lt \frac{1}{2}
\end{align*}
$$
其中：

* (1)：$h_{2,j}$ 在 slot $j$ 中出现 hash collision 的概率小于或等于任意 $i$ 和 $i^{'}$ 组合出现 hash collision 的概率总和。若不同组合出现 hash collision 的事件相互独立，则取等号。
* (1) $\rightarrow$ (2)：所有可能组合可能总数为 $\binom{l_j}{2}$，而每个组合下，$h_{2,j}$ 出现 hash collision 的概率为 $\frac{1}{m}$，即 $\frac{1}{l_j^{2}}$。

以上计算过程与 birthday paradox 十分类似，有兴趣可阅读参考文献了解详情。其基本结论就是：假设一年有 $m$ 天，那么如果想让 $n$ 个人中存在 2 个人生日相同的概率为 $\frac{1}{2}$，则 $n \approx \sqrt{m}$。

综上所述，**Step 2.5** 执行一次出现 hash collision 的概率小于 $\frac{1}{2}$，那么这个过程就与问题 **”抛硬币直到抛到一次正面为止”** 同构，在 Lecutre 7 中，我们计算过 $\mathbb{E}(\#trials) \le 2$，且在极大概率下 (w.h.p)，$\#trials = O(logn)$。另外在 CLRS 的课后题 Problem 11-2 中证明了 $\mathbb{E}[max(l_j)] = \theta{(\frac{logn}{log(logn)})} = O(logn)$，于是整个构建时间复杂度在极大概率下 (w.h.p) 为 $O(logn) \cdot O(logn) \cdot O(n) = O(nlog^{2}n)$，即上文提到的 polynomial (w.h.p)。

针对 **Step 1.5**，定义 indicator variable：
$$
I_{i,i^{'}} = 
\begin{cases}
	1 & if\space h(k_i) = h(k_i^{'}) \\
	0 & otherwise
\end{cases}
$$
那么就可以计算第二层 hash table 总共占用的空间：
$$
\begin{align*}
  \mathbb{E}[\sum_{j=0}^{m-1}l_j^{2}] &= \mathbb{E}[\sum_{i=1}^{n}\sum_{i'=1}^{n} I_{i,i'}] \\
  &= \sum_{i=1}^{n}\sum_{i'=1}^{n} E(I_{i,i'}) \\
  &= n + 2\binom{n}{2} \cdot \frac{1}{m} \\
  &= O(n) \space becasue \space m = \theta(n)
\end{align*}
$$
利用 Markov inequality，要使得：
$$
\underset{h_1}{\mathbb{p}}\{\sum_{j=0}^{m-1}l_j^2 \le cn \} \le \frac{1}{2}
$$
则需要：
$$
\frac{\sum_{j=0}^{m-1}l_j^2}{cn} \le \frac{1}{2}
$$
那么只要取得一个足够大的常数 $c$ 就能满足。现在又回到了问题 **”抛硬币直到抛到一次正面为止”** ，同样根据 Lecture 7，$\mathbb{E}(\#trials) \le 2$ ，且在极大概率下 (w.h.p)，$\#trials = O(logn)$。因此 **Step 1** 和 **Step 1.5** 的时间复杂度在极大概率下 (w.h.p) 为 $O(nlogn)$。

## TODO

* 阅读 perfect hashing paper。

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
  * Problem 11-2: slot-size bound for chaining
* [perfect hashing paper: Storing a Sparse Table with O(1) Worst Case Access Time](https://cs.dartmouth.edu/~ac/Teach/CS105-Winter05/Handouts/fks-perfecthash.pdf)

