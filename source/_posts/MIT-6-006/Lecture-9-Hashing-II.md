---
title: Lecture-9-Hashing-II
date: 2021-02-14 08:28:33
tags:
mathjax:
- true
---

# Lecture 9: Hashing II

## 0. Overview

* Table Resizing
* Amortization
* String Matching and Karp-Rabin
* Rolling Hash

## 1. How large should table be? Table Resizing

到目前为止，我们的讨论都建立在假设 "hash table 大小不变" 的前提下。在实践中多数情况下我们无法确定 hash table 的最终大小，且不同的 hash table 大小不同，理想情况下，我们希望无论输入怎样，都能做到 $m = \theta(n)$。如果使用静态大小的 hash table，则可能出现以下两种情况：若 $m$ 过小，则 $\alpha$ 过大，导致 hash table 性能下降；若 $m$ 过大，则填充率低，导致空间浪费。一个自然而然的想法就是使用动态大小的 hash table：**从一个小的固定 (常数) 开始，按需放缩**。

### 1.1 Rehashing

在扩容或缩容之后，我们需要重新计算每个元素的位置，这个过程称为 rehashing：

```python
for item in oldTable:
  newTable.insert(item)
```

rehashing 的时间复杂度为 $\theta(n+m)$，如果 $m = \theta(n)$，则总时间复杂度为 $\theta(n)$。

### 1.2 How fast to grow?

什么时候触发 table resizing？

* $m += 1$：即每次插入新的 key/item 时扩容。$n$ 次 `insert` 的时间复杂度为：$\theta(1+2+...+n) = \theta(n^2)$
* $m\space *= 2$：在第 $2^i$ 次插入 (假设没有重复 keys) 时扩容。$n$ 次 `insert` 的时间复杂度为：$\theta(1+2+4+8+...+n) = \theta(n)$。尽管每次扩容的时间复杂度为 $\theta(n)$，但平均到每次插入时间复杂度为 $\theta(1)$，即所谓 amotized cost (均摊复杂度)。

### 1.3 Amortized Analysis

amorized analysis 就像按揭贷款，尽管贷款总金额高，但月供可以接受。

> 当一个方法执行 $k$ 次的复杂度 $\le k \cdot T(n)$ 时，我们称该方法的 amortized cost 为 $T(n)$

所以，根据上一节的分析，我们可以认为：插入一个新 key/item 到 hash table 中的 amortized time 为 $\theta(1)$。

### 1.4 Final Time Complexity

在 SUHA 下，通过 table resizing，我们能够维持 $m = \theta(n)$，即保证 $\alpha = \theta(1)$，这样 search 的时间复杂度为 $\theta(1)$。如果删除数据后不缩容，可能造成空间浪费。CLRS 建议当 $n$ 下降到 $\frac{m}{4}$ 时将容量缩小一半，推理过程类似 insert，不难想到 delete 的 amortized time 为 $\theta(1)$。

综上所述：hash table 的时间复杂度分析如下表所示：

| Method       | Time Complexity       |
| ------------ | --------------------- |
| Insert(item) | $\theta(1)$ amortized |
| delete(item) | $\theta(1)$ amortized |
| search(key)  | $\theta(1)$           |

## 2. String Matching

**输入两个字符串 $s$ 和 $t$，确定 $s$ 是否是 $t$ 的子串 (如果是，请输出出现的次数和位置)**。string matching 最常见的应用场景就是 unix/linux 中的 *grep* 命令。

### 2.1 Simple Algorithm

暴力解法：

```python
ret, pos = false, -1
for i in range(len(t) - len(s)):
  if s == t[i:i+len(s)]:
    ret, pos = true, i
    break
return ret, pos
```

时间复杂度：每次子串对比消耗 $\theta(|s|)$，一共对比 $|t| - |s|$ 次，因此时间复杂度为：$\theta(|s| \cdot (|t| - |s|))$。

### 2.2 Karp-Rabin Algorithm

用对比 hash value 代替直接对比字符串：

```python
ret, pos = false, -1
hs = hash(s)
for i in range(len(t)-len(s)):
  if hs == hash(t[i:i+len(s)]):
    if s == t[i:i+len(s)]:
    	ret, pos = true, i
    break
return ret, pos
```

时间复杂度：每次计算 hash value 消耗 $\theta(|s|)$，如果 hash value 相等，需要再对比一次字符串，消耗 $\theta(|s|)$。总体时间复杂度与 simple algorithm 相同，在常数项上有所差异。在计算 `hash(t[i:i+len(s)])` 时，我们已经见过了 `t[i:i+len(s)]` 中的所有字母，如果在计算 `hash(t[i+1:i+1+len(s)])` 时，我们可以只考虑 `t[i]` 和 `t[i+1+len(s)]`，那么时间复杂度降进一步降低。接下来看我们如何进一步将 $|t| - |s|$ 次 hash value 计算复杂度从 $\theta(|s|)$ 降低到 $\theta(1)$。 

### 2.2.1 Rolling Hash ADT

rolling hash ADT 维持一个字符串 $s$ 的信息，支持 3 种方法：

* `r()`：获取当前字符串 $s$ 的 hash value，即 $h(s)$
* `r.append(c)`：往 $s$ 的末尾增加一个字符 $c$
* `r.skip(c)`：从 $s$ 的头部删除一个字符 $c$

利用 rolling hash ADT，上面的 string matching 算法可以改写成：

```python
rs, rt = RollingHash(), RollingHash()
for c in s:
  rs.append(c)
for c in t[:len(s)]:
  rt.append(c)
if rs() == rt():
  #...
for i in range(len(s), len(t)):
  rt.skip(t[i-len(s)])
  rt.append(t[i])
  if rs() == rt():
    #...
```

### 2.2.2 Rolling Hash Implementation

将 $s$ 看作是基数 (base) 为 $a$ 的数字 $u$，$a$ 为字符集大小。hash function 使用 division method：
$$
h(s) = u\space mod\space p
$$
其中 $p$ 是与 $|s|$ 或 $|t|$ 接近的素数，那么：

* $r() = u\space mod\space p$
* $r.append(c): (u \cdot a + ord(c))\space mod\space p = [(u\space mod\space p)\cdot a + ord(c)]\space mod\space p$
* $r.skip(c): [u-ord(c)\cdot (a^{|s| - 1}\space mod \space p)]\space mod\space p \\= [(u\space mod\space p) - ord(c) \cdot (a^{|s|-1} mod \space p)] \space mod\space p$

rolling hash implementation 需要保存 $u\space mod\space p$ 和 $|s|$ 信息。

### 2.2.3 Complexity

利用 Karp-Rabin Algorithm，每次计算 hash value 的时间从 $\theta(|s|)$ 降低到 $\theta(1)$，平均情况下时间复杂度为 $\theta(|t|) + (1 + \#matches)\theta(|s|)$，即 $O(|t| + |s|)$。最坏情况下时间复杂度仍为 $O(|t|\cdot |s|)$，即每次 $rs() == rt()$ 都为 `true`，触发字符串比较。

## References

* MIT 6.006 lecture notes, [handwritten](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-006-introduction-to-algorithms-fall-2011/lecture-videos/MIT6_006F11_lec09_orig.pdf), [notes typed](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-006-introduction-to-algorithms-fall-2011/lecture-videos/MIT6_006F11_lec09.pdf), [video](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-006-introduction-to-algorithms-fall-2011/lecture-videos/lecture-9-table-doubling-karp-rabin/)
* Wikipedia: [Rabin-Karp algorithm](https://en.wikipedia.org/wiki/Rabin%E2%80%93Karp_algorithm)
* CLRS: 17.4

