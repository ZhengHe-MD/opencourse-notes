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

利用 Chebyshev's Inequality：
$$
\mathbb{P}(|X - \frac{1}{t+1}| > \frac{\varepsilon}{t+1}) <
	\frac{(t+1)^2}{\varepsilon^2} \cdot \frac{2}{(t+1)(t+2)} < \frac{2}{\varepsilon^2}
$$

### 3.2 FM+

Morris+ 故技重施，用多个 FM 副本取均值：

1. 初始化 $q = \frac{\eta}{\varepsilon^2}$ 个相互独立的 $FM$ 副本
2. 记 $FM_i$ 上的计数器为 $X_i$
3. 输出 $\frac{1}{Z} - 1$，其中 $Z = \frac{1}{q}\sum_iX_i$

利用 3.1 中的结论，可以直接得到 $\mathbb{E}(Z) = \frac{1}{t+1}$，$Var(Z) = \frac{1}{q}\frac{t}{(t+1)^2(t+2)} < \frac{1}{q(t+1)^2}$。

**Claim 3：**$\mathbb{P}(|Z - \frac{1}{t+1}| > \frac{\varepsilon}{t+1}) < \eta$

利用 Chebyshev's Inequality：
$$
\mathbb{P}(|Z - \frac{1}{t+1}| > \frac{\varepsilon}{t+1}) <
	\frac{(t+1)^2}{\varepsilon^2}\frac{1}{q(t+1)^2} < \eta
$$
**Claim 4：**$\mathbb{P}(|(\frac{1}{Z} - 1) - t|) > O(\varepsilon)t) < \eta$

根据 Claim 3，在概率 $1 - \eta$ 下：
$$
Z = (1 \pm \varepsilon)\frac{1}{t+1}
$$
因此：
$$
\begin{align*}
	\frac{1}{Z} - 1 &= \frac{1}{(1 \pm \varepsilon)\frac{1}{t+1}} - 1 \tag{5} \\ 
	                &= (1\pm O(\varepsilon))(t+1) - 1 \tag{6} \\
	                &= (1\pm O(\varepsilon))t \pm O(\varepsilon)
\end{align*}
$$
其中 $(5) \rightarrow (6)$ 可以通过 Taylor's Series 展开得到：
$$
\frac{1}{1-x} = 1 + x + x^2 + x^3 + ... \space \space \space if \space |x| < 1
$$

### 3.3 FM++

Morris++ 故技重施，取多个 FM+ 副本的中位数：

1. 初始化 $s = \lceil 36ln(\frac{2}{\delta}) \rceil$ 个相互独立的、失败率为 $\eta = \frac{1}{3}$ 的 FM+ 副本
2. 记 $FM+_i$ 上的计数器为 $Z_i$
3. 输出集合 $\{\frac{1}{Z_j} - 1\}_{j=1}^{s}$ 的中位数 $\hat{t}$

**Claim 5**：$\mathbb{P}(|\hat{t} - t| > \varepsilon t) < \delta$

证明：定义指示变量
$$
Y_j =
\begin{cases}
	1 & if\space |(\frac{1}{Z_j}-1) - t| > \varepsilon t \\
	0 & otherwise
\end{cases}
$$
如果 $FM++$ 失效，意味着超过半数的 $FM+$ 实例失效，即：
$$
\sum_{j=1}^s Y_j > \frac{s}{2} \tag{7}
$$
由于每个 $FM+$ 副本的失败率为 $\frac{1}{3}$，那么 $Y_j$ 的期望
$$
\mathbb{E}Y_j = \mathbb{P}(Y_j = 1) < \frac{1}{3}
$$
于是调整 $(7)$ 式，得到：
$$
\mathbb{P}(\sum_{j=1}^s Y_j > 
	\frac{s}{2}) =
	\mathbb{P}(\sum Y_j - \frac{s}{3} > \frac{s}{6}) \tag{8}
$$
假设 $\mathbb{E}Y_j = \frac{1}{3}$，$(8)$ 式可转化为：
$$
\mathbb{P}(\sum Y_j - \mathbb{E}\sum Y_j > \frac{1}{2}\mathbb{E}\sum Y_j)
$$
利用 Chernoff Inequality 得到：
$$
\mathbb{P}(\sum Y_j - \mathbb{E}\sum Y_j > \frac{1}{2}\mathbb{E}\sum Y_j) \le
	e^{-\frac{(\frac{1}{2})^2\frac{s}{3}}{3}} < \delta
$$
一个 FM++ 需要 $O(lg(\frac{1}{\delta}))$ 个 FM+ 副本，每个副本需要消耗 $O(\frac{1}{\varepsilon^2})$ 空间，因此空间复杂度总计为 $O(\frac{lg(1/\delta)}{\varepsilon^2})$。

### 3.4 Non-idealized FM

接下来，我们看如何构造非理想化的 FM 算法。

#### 3.4.1 k-wise independent functions

定义：一个 hash family $\mathcal{H}$ ，其中任意 $h \in \mathcal{H}$ 会将集合 $[a]$ 中的元素映射到集合 $[b]$ 中。如果 $\mathcal{H}$ 满足对 $\forall j_1,...,j_k \in [b]$ 以及 $\forall\space distinct \space i_1,...,i_k \in [a]$：
$$
\mathbb{P}_{h \in \mathcal{H}}(h(i_1) = j_1 \cap ...\cap h(i_k) = j_k) = \frac{1}{b_k}
$$
即 $k$ 个元素和哈希值间相互独立。

**例 1：**$[a]\rightarrow[b]$ 上的所有 hash function 共同组成的 hash family，对任意 k 满足 k-wise independent。整个 hash family 需要存储 $|\mathcal{H}| = b^a$ 种映射关系，因此它的空间占用为 $O(a\lg{b})$。

**例 2：**设 $a = b = q$，其中 $q$ 是一个素数幂 (prime power)，即 $q = p^r$，定义 $\mathcal{H}_{poly}$ 为所有系数属于 $\mathbb{F}_q$ 的 $k-1$ 次多项式集合。整个 hash family 需要存储 $|\mathcal{H}_{poly}| = q^k$，因此它的空间占用为 $O(k\lg{q})$。可以通过 **the fundamental theorem of algebra** 证明  $\mathcal{H}_{poly}$ 满足 k-wise independent。

#### 3.4.2 Non-idealized FM

我们先利用 $O(\lg{n})$ bits 空间获得 $t$ 的 $O(1)-approximation$ ，即估计值 $\tilde{t}$ 对于某个常数 $C$，满足 $\frac{t}{C} \le \tilde{t} \le Ct$。

1. 从某 $[n]\rightarrow [n]$， 2-wise independent hash family 中随机选取一个 $h$，其中 $n$ 是 2 次幂 (power of 2)，如果不是，则向上取最近的 2 次幂。
2. 维护计数器 $X = max_{i\in stream}lsb(h(i))$，其中 $lsb$ 是 least significant bit，但这里的意思实际上是：如果 $h(i) = 0b0^k1...$，那么 $lsb(h(i)) = k+1$
3. 输出 $2^X$

这里的直觉与 Idealized FM 类似，不再赘述。对于某个固定的值 $j$，定义

* $Z_j$ 为满足 $lsb(h(i)) = j$ 的 $i$ 的个数
* $Z_{>j}$ 为满足 $lsb(h(i)) > j$ 的 $i$ 的个数

以及指示变量：
$$
Y_i = 
\begin{cases}
  1, & lsb(h(i)) = j \\
  0, & otherwise
\end{cases}
$$
那么 $Z_j = \sum_{i \in stream} Y_i$，由于 $lsb(h(i)) = j$ 意味着 $h(i) = 0b0^{j}1...$，即前 $j$ 位必须为 0，第 $j+1$ 位必须为 1，那么 $P(Y_i = 1) = \frac{1}{2^{j+1}}$，因此 $\mathbb{E}Z_j = \frac{t}{2^{j+1}}$，类似地：
$$
\mathbb{E}Z_{>j} = t(\frac{1}{2^{j+1}} + \frac{1}{2^{j+3}}+...) < \frac{t}{2^{j+1}}
$$
以及：
$$
\begin{align*}
Var[Z_j] = Var[\sum Y_i] &= \mathbb{E}(\sum Y_i^2) - (\mathbb{E}\sum Y_i)^2 \\
	                       &= \sum_{i_1,i_2}\mathbb{E}(Y_{i_1} Y_{i_2}) \\
	                       &= \frac{t(t-1)}{2^{2(j+1)}} \\
	                       &< (\frac{t}{2^{j+1}})^2 \\
	                       &= (\mathbb{E}Z_{>j})^2 
\end{align*}
$$
其中 $h$ 属于 2-wise hash family，因此 $\mathbb{E}(Y_{i_1} Y_{i_2}) = \mathbb{E}(Y_{i_1})\mathbb{E}(Y_{i_2})$。我们从 $j$ 附近的两个值 $j^-$ 和 $j^+$，来分析估计的准确度。

对于 $j^- = \lceil\lg{t} - 5\rceil$，有 $16 \le \mathbb{E}Z_{j^-} \le 32$，那么：
$$
\begin{align*}
\mathbb{P}(Z_{j^-} = 0) &\le \mathbb{P}(|Z_{j^-} - \mathbb{E}Z_{j^-}| \ge 16) \\
			&< \frac{Var[Z_{j^-}]}{16^2} \tag{9} \\
			&< (\frac{t}{2^{\lceil\lg{t} - 5\rceil+1}})^2 \frac{1}{16^2} \\
			&< (\frac{t}{2^{\lceil\lg{t} - 5\rceil}})^2 \frac{1}{4 \cdot 16^2} \\
			&< (\frac{t}{2^{\lceil\lg{t}\rceil}})^2 \frac{1}{4 \cdot 16^2 \cdot 16^2} \\
			&< \frac{1}{2^{18}}
\end{align*}
$$
$(9)$ 式通过 Chebyshev's Inequality 得到。通过以上推导可以得出结论：$\tilde{t}$ 过大的概率很小，更精确地说 $\mathbb{P}(\tilde{t} > 32t) < \frac{1}{2^{18}}$，即 $\tilde{t} < Ct$。

对于 $j^+ = \lceil\lg{t} + 5\rceil$，有 $\mathbb{E}Z_{j^+} \le \frac{1}{16}$，根据 Markov's Inequality 可以得到：
$$
\begin{align*}
	\mathbb{P}(Z_{>j} \ge 1) < \frac{\mathbb{E}Z_{j^+}}{1} = \frac{1}{16}
\end{align*}
$$
通过以上推导可以得出结论：$\tilde{t}$ 过小的概率很小，更精确地说，$\mathbb{P}(\tilde{t} < \frac{t}{32}) < \frac{1}{16}$，即 $\tilde{t} > \frac{t}{C}$。至此，我们证明了估计值 **$\tilde{t}$ 对于某个常数 $C$，满足 $\frac{t}{C} \le \tilde{t} \le Ct$**，即 $\tilde{t}$ 是 $t$ 的 constant approximation。

#### 3.4.3 Refine to $1+\varepsilon$

定义 TS (trivial solution)：算法 TS 完整地存储前 $\frac{C}{\varepsilon}^2$ 个不同元素，它能保证当 $t \le \frac{C}{\varepsilon^2}$ 时，计数正确。于是我们可以用 TS 构建下面的算法：

1. 初始化 $\lg{n}$ 个 TS 副本：$TS_0, ...,TS_{\lg{n}}$
2. 从某 $[n] \rightarrow [n]$， 2-wise independent hash family 中随机选取一个 hash function $g$
3. 将数字 $i \in stream$ 丢进 $TS_{lsb(g(i))}$ 中
4. 输出 $\tilde{t} = 2^{j+1} \cdot |TS_j|$，其中 $\frac{t}{2^{j+1}} \approx (\frac{1}{\varepsilon})^2$

定义 $B_j$ 为进入 $TS_j$ 的不同元素个数，则 $\mathbb{E}B_j = \frac{t}{2^{j+1}} = Q_j$。Chebyshev's Inequality 描述的是数据分布在 $O(1)$ 个标准差 ($\sigma$) 返回内的概率较高，因此在大概率下，$B_j = Q_j \pm O(\sqrt{Q_j})$。当 $Q_j \approx \frac{1}{\varepsilon^2}$ 时，$B_j = (1\pm O(\varepsilon))Q_j$。

理解本算法的直觉在于：从任意一个 $TS$ 副本中，我们都可以去估算 $t' = 2^{j+1} \cdot |TS_j|$，但选择哪个才是最准确的？如果 $Q_j = \frac{t}{2^{j+1}} \ll \frac{1}{\varepsilon^2}$，它的估计值可能偏差较大，比如 $Q_j = 1$；如果 $Q_j = \frac{t}{2^{j+1}} \gg \frac{1}{\varepsilon^2}$，那么 TS 就无法准确存储不同的元素，因为它支持完整存储的元素个数存在上限 $\frac{C^2}{\varepsilon}$。而取中间某个 $j$ 满足 $\frac{t}{2^{j+1}} \approx (\frac{1}{\varepsilon})^2$ 能够真正满足我们的预期。

整个算法使用 $\lg{n}$ 个 TS 副本，每个 TS 副本存储 $\frac{C^2}{\varepsilon}$ 个元素，每个元素占用 $\lg{n}$ bits 空间，因此总体空间复杂度为 $\frac{C}{\varepsilon^2}(\lg{n})^2$，即 $O(\frac{1}{\varepsilon^2}\lg^2{n})$。目前已知的最优算法空间复杂度为 $O(\frac{1}{\varepsilon^2} + \log{n})$。

## References

* Lecture 2: [notes](http://people.seas.harvard.edu/~minilek/cs229r/fall15/lec/lec2.pdf), [video](https://www.youtube.com/watch?v=fvZ_6pvP4jc&list=PL2SOU6wwxB0v1kQTpqpuu5kEJo2i-iUyf&index=2&pbjreload=101)

* Flajolet and Martin, [paper](http://algo.inria.fr/flajolet/Publications/src/FlMa85.pdf)

