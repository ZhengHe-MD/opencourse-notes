---
title: Lecture 2
date: 2021-02-17 22:50:32
tags:
mathjax:
- true
---

# Lecture 2

{% youtube fvZ_6pvP4jc %}

## 0. Overview

* Distinct elements

## 1. Distinct elements

问题描述：给定一个整数流 (stream of integers)，$i_1, ..., i_m \in [n]$，其中 $[n] := \{1,2,...,n\}$，输出目前所见的数字集合大小，即不同的数字个数。

## 2. Straightforward algorithms

1. 利用一个长度为 $n$ 的 bit array，若遇到某整数 $k$，就将第 $k$ bit 置为 1
2. 将整个整数流出现过的数字存下。

方法 1 占用 $O(n)$ bits 空间；方法 2 占用 $O(m\lg{n})$ bits 空间。因此，结合二者我们能得到 $O(\min{(n, m\lg{n})}$) bits 空间。

## 3. Randomized approximation

类似 Morris 算法，如果想进一步减少空间占用，可以用精度换空间。即找到 $\tilde{t}$ 使得 $P(|t-\tilde{t}| > \varepsilon t) < \delta$。该算法最早由 Flajolet and Martin 提出，因此称之为 FM。

### 3.1 Idealized algorithm (FM)

FM 算法执行步骤如下：

* 选取一个随机函数 $h: [n] \rightarrow [0,1]$ (实际上这步无法实现：首先，至少需要 $O(n)$ 空间来存储 $n$ 个随机数；其次，$[0,1]$ 包含这之间的所有实数，计算机无法实现无限精度)
* 维护一个计数器 $X$，使得 $X = min_{i \in stream}h(i)$
* 输出 $\frac{1}{X} - 1$

这个算法的直觉是：如果出现 $t$ 个不同的数字，那么这些数字应该均匀地分布在 $[0,1]$ 上：

<img src="./fm-intuition.png" style="width: 480px" />

当 $X = 1$ 时，在 $\frac{1}{2}$ 处；当 $X = 2$ 时，分别在 $\frac{1}{3}$ 和 $\frac{2}{3}$ 处；当 $X = 5$，分别在 $\frac{k_i}{6}, k_i = {1, 2, ..., 5}$ 处。那么就可以直接猜测 $t + 1 = \frac{1}{X}$，即 $t = \frac{1}{X} - 1$。

**Claim 1**：$\mathbb{E}X = \frac{1}{t+1}$

证明：
$$
\begin{align*}
	\mathbb{E}X &=\int_{0}^{\infty}\mathbb{P}(X>\lambda)d\lambda \tag{1}\\
						  &=\int_{0}^{\infty}\mathbb{P}(\forall i \in stream, h(i) > \lambda)d\lambda \\
						  &=\int_{0}^{\infty}\prod_{r=1}^{t}\mathbb{P}(h(i_r) > \lambda)d\lambda \\
						  &=\int_{0}^{1}(1-\lambda)^td\lambda \\
						  &=\frac{1}{t + 1}
\end{align*}
$$
在上课时，最难理解的是 $(1)$ 式，它对于所有取值非负的随机变量都成立。$(1)$ 式的离散版本证明在[这里](https://math.stackexchange.com/questions/843845/find-the-mean-for-non-negative-integer-valued-random-variable)，连续版本证明在[这里](https://math.stackexchange.com/questions/172841/explain-why-ex-int-0-infty-1-f-x-t-dt-for-every-nonnegative-rando)。

**Claim 2**：$\mathbb{E}X^2 = \frac{2}{(t+1)(t+2)}$

证明：
$$
\begin{align*}
	\mathbb{E}X^2 &= \int_0^1 \mathbb{P}(X^2 > \lambda)d\lambda \\
	              &= \int_0^1 \mathbb{P}(X > \sqrt{\lambda})d\lambda \tag{2}\\
	              &= \int_0^1 (1 - \sqrt{\lambda})^td\lambda \tag{3}\\
	              &= 2\int_0^1 u^t(1-u)du \tag{4}\\
	              &= \frac{2}{(t+1)(t+2)}
\end{align*}
$$

* $(2) \rightarrow (3)$：$h(i)$ 服从在区间 $[0,1]$ 上的均匀分布
* $(3) \rightarrow (4)$：换元法，$u = 1 - \sqrt{\lambda}$

有了 **Claim 1** 和 **Claim 2**，就能得到方差 $Var(X) = \mathbb{E}X^2 - (\mathbb{E}X)^2 = \frac{t}{(t+1)^2(t+2)} < \frac{1}{(t+1)^2} = (\mathbb{E}X)^2$。

### 3.2 FM+

### 3.3 FM++

### 3.4 Non-idealized FM

#### 3.4.1 k-wise independent functions

#### 3.4.2 Non-idealized FM

#### 3.4.3 Refine to $1+\varepsilon$

## References

* Lecture 2: [notes](http://people.seas.harvard.edu/~minilek/cs229r/fall15/lec/lec2.pdf), [video](https://www.youtube.com/watch?v=fvZ_6pvP4jc&list=PL2SOU6wwxB0v1kQTpqpuu5kEJo2i-iUyf&index=2&pbjreload=101)

* Flajolet and Martin, [paper](http://algo.inria.fr/flajolet/Publications/src/FlMa85.pdf)

