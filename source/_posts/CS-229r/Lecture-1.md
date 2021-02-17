---
title: Lecture-1
date: 2021-02-16 10:22:01
tags:
mathjax:
- true
---

# Lecture 1

{% youtube s9xSfIw83tk %}

## 0. Overview

* course topics
* basic tail bounds
* approximate counting problem

## 1. Topic Overview

### 1.1 Sketching/Streaming

想象一个场景：A 和 B 各有一批数据，他们想要通过计算某个指标来衡量这两批数据的相似度。一种方式是 A (B) 把手里的数据全量发送给 B (A)，然后 B (A) 在本地计算相似度。但如果这两批数据体量都特别大，通过网络传输所有数据到对方的做法就变得不现实。另一种做法就是 A (B) 先计算手上数据的 sketch，即压缩后的数据，把 sketch 发送给 B (A)，然后 B (A) 在本地利用相同的方式计算手上数据的 sketch，最终通过比较两批数据的 sketch 近似估计相似度。

举一个具体例子：A 和 B 各有一组数字 $X$ 和 $Y$，他们想要计算两组数字的总和 (sum)，这时 A (B) 可以先计算 $X$ 的总和，然后发给 B (A)，然后 B (A) 也同样计算手上数字的总和，并将其和 A (B) 发送的总和加总，得到结果。这里的 总和就是我们说的 sketch。

现在我们标准化 sketch 的定义：

> 当我们想对一组数据 $X$ 执行某种计算，记为 $f(X)$，"*Sketch*" $C(X)$ 是 $X$ 的某种压缩形式，他允许我们在无法获取原始数据 $X$ 的情况下近似估计 $f(X)$ 的值。有时候 $f$ 也可以是二元函数：即给定 $C(X)$、$C(Y)$，近似估计 $f(X, Y)$ 的值。

在实践中，$X$ 可能由不断产生的样本点 $x$ 构成，我们无法一次性获取，甚至永远无法获取样本总体 $X$，这时就希望能维持当前已获得样本 $X'$ 的 sketch $C(X')$。以刚才的求和为例，我们可以维护 $X'$ 和 $Y'$ 的 running sum，这个过程就是 streaming。streaming 很常见，比如你的本地路由器就会通过计算流量的 sketch 来寻找 pattern。

### 1.2 Dimensionality Reduction

dimensionality reduction 就是将 high-dimensional 数据转化成 lower-dimensional 数据，通过对 lower-dimensional 数据计算来估计 high-dimensional 数据的对应值。而维度的降低能节省大量的计算资源，你的算法也就可以跑得更快。dimensionality reduction 常常被用于加速 clustering (聚类) 和 nearest neighbor (近邻) 等机器学习算法。

### 1.3 Large-scale Machine Learning

比如机器学习中最常见的 regression problems (回归问题)：先搜集一批样本点 $\{(z_i, b_i|i = 1,...,n)\}$，假设存在 $b_i = f(z_i) + noise$，我们期望能够找到尽可能接近 $f$ 的 $\tilde{f}$。而最简单的回归方法就是 linear regression (线性回归)，假设存在 $x$ 使得 $f(z) = \langle x, z \rangle$，且 noise 符合正态分布，最优的估计就是 Least Squares (最小二乘法)：
$$
x^{LS} = arg\space min \|Zx - b\|_2^2 = (Z^TZ)^{-1}Zb
$$
其中 $b = (b1,...,b_n)^T)$，$Z = (z_1,...,z_n)^T$。如果 $Z$ 很大，即样本数量巨大，直接执行矩阵乘法将消耗大量计算资源。在之后的课程中，我们会学习更快的近似计算 $(Z^TZ)^{-1}Z$ 的方法。

matrix completion 是另外一种 regression problem。以 Netflix 为例：将用户和偏好当作两个维度形成的矩阵通常是十分稀疏的矩阵，因为每个用户只可能为很小的一部分影视作品打分，那么想要基于有限的信息来估计用户的完整偏好，就依赖于 matrix completion。

### 1.4 Compressed Sensing

如常见的图片压缩格式 JPEG 和 MRI (核磁共振) 的数据处理。

### 1.5 External Memory Model

在一些无法将所需数据一次性读入内存的场景中，磁盘 I/O 通常是性能瓶颈所在。此时需要以 I/O 来衡量算法的效率，这时计算模型就变成了 external memory model。在该模型下，我们假设有一个无限大的磁盘，磁盘被划分成固定大小 (如 $b$ bits) 的 blocks (数据块)，同时拥有一个大小为 $M$ 的内存，同样也被划分成大小为 $b$ bits 的 blocks。

数据库中常见的 B-tree/LSM-tree 就是为 external memory model 所设计的数据结构。

### 1.6 Other Models

如 map reduce

## 2. Probability Review

在本课中，我们主要讨论的是离散变量，假设存在一个离散变量 $X$，其所有可能取值组成集合 $S$，那么它的期望 就是：$\mathbb{E}(X) = \sum_{j\in S} j \cdot \mathbb{P}(X = j)$。

### 2.1 Linearity of Expectation

不同随机变量的期望线性可加：
$$
\mathbb{E}(X + Y) = \mathbb{E}X + \mathbb{E}Y
$$

### 2.2 Markov's Inequality

如果 $X$ 的取值非负，那么 Markov's Inequality 成立：
$$
\forall \lambda > 0, \mathbb{P}(X > \lambda) < \frac{\mathbb{E}X}{\lambda}
$$
证明：假设 $X$ 的可能取值从小到大依次是：$x_1 < x_2 < ... < x_j = \lambda <...<x_n$，那么：
$$
\begin{align*}
	\mathbb{E}X &= \sum_{i=1}^{n}x_i\mathbb{P}(X = x_i) \\
	              &> \sum_{i=j}^{n}x_i\mathbb{P}(X = x_i) \\
	              &> \sum_{i=j}^{n}\lambda\mathbb{P}(X = x_i)
\end{align*}
$$
两边同时除以 $\lambda$：
$$
\sum_{i=j}^{n}\mathbb{P}(X = x_i) = \mathbb{P}(X > \lambda) < \frac{\mathbb{E}X}{\lambda}
$$

> Markov's Inequality 为一个 non-negative random variable 取值超过某个常数 $\lambda$ 的概率给出上限

有兴趣可以看一下 MIT RES.6-012 概率论入门课的相关视频：

{% youtube vjYanZ1nsZg %}

### 2.3 Chebyshev's Inequality

对任意随机变量 $X$，有 Chebyshev's Inequality 成立：
$$
\forall \lambda > 0, \mathbb{P}(|X - \mathbb{E}X| > \lambda) < \frac{\mathbb{E}(X - \mathbb{E}X)^2}{\lambda^2}
$$
证明：$\mathbb{P}(|X - \mathbb{E}(X)| > \lambda) = \mathbb{P}((X - \mathbb{E}(X))^2 > \lambda^2)$，利用 Markov's Inequality 得证。

更广义的 Chebyshev's Inequality 版本是：
$$
\forall p > 0,\forall \lambda > 0, \mathbb{P}(|X - \mathbb{E}X| > \lambda) < \frac{\mathbb{E}(X - \mathbb{E}X)^p}{\lambda^p}
$$
证明同上。

> Chebyshev's Inequality 为任意随机变量取值与期望之间的距离超过某个常数 $\lambda$ 的概率给出上限。

### 2.4 Chernoff Bound

假设 $X_1,...,X_n$ 是相互独立的随机变量，且 $X_i \in [0, 1]$，定义 $X = \sum_iX_i, \lambda > 0,$
$$
\mathbb{P}(|X - \mathbb{E}X| >
\lambda \cdot \mathbb{E}X) \le 2 \cdot e^{-\lambda^2 \cdot \mathbb{E}X/3}
$$
证明：要证明上述不等式成立，只需要分别证明 $(1)$ 和 $(2)$ 即可。
$$
\mathbb{P}(X - \mathbb{E}X >
\lambda \cdot \mathbb{E}X) \le  e^{-\lambda^2 \cdot \mathbb{E}X/3} \tag{1}
$$

$$
\mathbb{P}(X - \mathbb{E}X <
-\lambda \cdot \mathbb{E}X) \le  e^{-\lambda^2 \cdot \mathbb{E}X/3} \tag{2}
$$
证明 $(1)$，对 $\forall t > 0$：
$$
\mathbb{P}(X - \mathbb{E}X > \lambda \cdot \mathbb{E}X) = 
	\mathbb{P}(e^{t(X-\mathbb{E}X)} > e^{\lambda t\mathbb{E}X})
$$
要证明 $(1)$ 成立，等价于证明：
$$
\mathbb{P}(e^{t(X-\mathbb{E}X)} > e^{\lambda t\mathbb{E}X}) \le e^{-\lambda^2 \cdot \mathbb{E}X/3} \tag{3}
$$
利用 Markov's Inequality，$(3)$ 等价于：
$$
\begin{align*}
\frac{\mathbb{E}e^{t(X - \mathbb{E}X)}}{e^{\lambda t\mathbb{E}X}} &\le e^{-\lambda^2 \cdot \mathbb{E}X/3} \\
\frac{1}{e^{\lambda t\mathbb{E}X}} &\le e^{-\lambda^2 \cdot \mathbb{E}X/3} \\
t &\ge \frac{\lambda}{3}
\end{align*}
$$
即对 $\forall \lambda$，取 $t \ge \frac{\lambda}{3}$ 即可。同理可证相同条件下，有 $(2)$ 成立。

> Chernoff Bound 为多个随机变量的和与其期望的距离超过 $\lambda$ **倍**的概率给出呈指数下降的上限。

### 2.5 Bernstein Inequality

假设 $X_1,...,X_n$ 是相互独立的随机变量，且 $|X_i| \le K$，定义 $X = \sum_iX_i, \sigma^2 = \sum_i\mathbb{E}(X_i - \mathbb{E}X_i)^2$，对于 $\forall t > 0$，有：
$$
\mathbb{P}(|X - \mathbb{E}X|>t) \lesssim e^{-ct^2/\sigma^2} + e^{-ct/K}
$$
其中 $\lesssim$ 表示存在一个足够大的常数 $c$ 使得不等式成立。

证明：暂略。

> Bernstein Inequality 为多个随机变量的和与其期望的距离超过 $\lambda$ 的概率给出呈指数下降的概率上限。

## 3. Approximate Counting Problem

接下来要讨论的问题首次出现在论文 [Counting Large Numbers of Events in Small Registers](https://www.inf.ed.ac.uk/teaching/courses/exc/reading/morris.pdf) 中。

### 3.1 Problem Definition

问题：有一类事件不断发生，我们想要计算事件发生次数 (**但不要求精确计算**)，同时将数据占用的空间最小化。

一种最简单直接的方式就是维护一个精确的计数器，假设事件发生总数为 $n$，那么计数器占用的空间大小为 $\log{n}$ bits。根据 Pigeonhole Principle，如果要精确计数，我们不可能使用比 $log\space n$ bits 更小的空间。

如果使用近似计数，即如题所述 (approximate counting)，那么问题就转化成输出 $\tilde{n}$ 使得：
$$
\mathbb{P}(|\tilde{n} - n| > \varepsilon n) < \delta
$$
其中 $\varepsilon$ 和 $\delta$ 是我们可以任意取的参数，如 $\varepsilon = \frac{1}{3}$、$\delta = 1\%$。

### 3.2 Analysis

首先，我们可以用直觉来推测，如果要设计一个确定性的 (deterministic) 算法来解决上述问题，我们的空间复杂度下限是 $O(\log\log{n})$ bits。因为我们最多可以记录 $$\log{n} 个从小到大的区间，且区间大小呈指数增长，那么原本记录事件发生次数的计数器就变成记录当前处于第几个区间的计数器，因此就能推测出空间复杂度满足 $O(\log\log{n})$。实际上，下面的 Morris 算法就能满足我们的空间复杂度下限。

#### 3.2.1 Morris

Morris 算法步骤如下：

1. 初始化 $X \leftarrow 0$
2. 每当遇到新的事件，按概率 $\frac{1}{2^X}$ 执行 $X \leftarrow X + 1$
3. 输出 $\tilde{n} = 2^X - 1$

从直觉触发，我们可以推测 $X \approx \log{n}$。用 $X_n$ 表示接收到 $n$ 次事件后的 $X$ 值，那么我们需要证明：
$$
\mathbb{E}2^{X_n} = n + 1
$$
利用数学归纳法证明：

1. 当 $m=0$ 时，等式成立
2. 假设 $m = n$ 时等式成立，$m = n+1$ 时，有：

$$
\begin{align*}
	\mathbb{E}2^{X_{n+1}}
	 &= \sum_{j=0}^{\infty}\mathbb{P}(X_n = j) \cdot \mathbb{E}(2^{X_{n+1}}|X_n = j) \\
	 &= \sum_{j=0}^{\infty}\mathbb{P}(X_n = j) \cdot (2^j(1-\frac{1}{2^j} ) + \frac{1}{2^j} \cdot 2^{j+1}) \\
	 &= \sum_{j=0}^{\infty}\mathbb{P}(X_n = j)2^j + \sum_{j}^{\infty}\mathbb{P}(X_n = j) \\
	 &= \mathbb{E}2^{X_n} + 1 \\
	 &= (n+1) + 1
\end{align*}
$$

Q.E.D。解决了 expectation 问题，接下来分析一下 $\tilde{n}$ 不同偏离程度下的概率上限，由 Chebshev's Inequality：
$$
\mathbb{P}(|\tilde{n} - n| > \varepsilon n) <
  \frac{1}{\varepsilon^2n^2} \mathbb{E}(\tilde{n} - n)^2 = 
  \frac{1}{\varepsilon^2n^2} \mathbb{E}(2^X - 1 - n)^2 \tag{4}
$$
同样通过数学归纳法可以证明：
$$
\mathbb{E}(2^{2X_n}) = \frac{3}{2}n^2 + \frac{3}{2}n + 1 \tag{5}
$$
将 $(5)$ 代入 $(4)$ 中可以得到：
$$
\mathbb{P}(|\tilde{n} - n| > \varepsilon n)
  = \frac{1}{\varepsilon^2n^2} \cdot \frac{n(n-1)}{2}
	< \frac{1}{\varepsilon^2n^2} \cdot \frac{n^2}{2}
	= \frac{1}{2\varepsilon^2} \tag{6}
$$
如果希望 $\varepsilon = \frac{1}{3}$、$\delta = 1\%$，将 $\varepsilon$ 代入得到：
$$
\mathbb{P}(|\tilde{n} - n| > \varepsilon n) < \frac{9}{2}
$$
概率小于 $450\%$，这意味着 $\tilde{n}$ 与 $n$ 之间的差肯定大于 $\frac{1}{3}n$；如果希望 $\delta = 1\%$，将它代入：
$$
\frac{1}{2 \varepsilon^2} = \frac{1}{100} \Rightarrow \varepsilon \approx 7.07
$$
即 $\tilde{n}$ 与 $n$ 之间的差大概率在 7 倍以内。如果这样的精度不满意，我们就需要想办法缩进上限。

#### 3.2.2 Morris+

俗话说，三个臭皮匠赛过诸葛亮。利用 $s$ 个相互独立的 Morris 副本计数，然后取它们的输出均值，那么 $(6)$ 式就变成：
$$
\mathbb{P}(|\tilde{n} - n| > \varepsilon n) < \frac{1}{2s\varepsilon^2} \tag{7}
$$
通过控制副本的数量就能达到缩进上限的目的，也是很典型的空间换精度。需要注意的是，如果 $s\log\log{n} \ge log{n}$ 时，精确计数就更划算。因此，副本的数量也不是越多越好。

#### 3.2.3 Morris++

Morris++ 的思路是：**利用 $t$ 个 failure probability 为 $\frac{1}{3}$ 的 Morris+ 副本，取结果的中位数** 。

根据 $(7)$ 式： 
$$
\frac{1}{2s\varepsilon^2} = \frac{1}{3} \Rightarrow
s = \frac{3}{2\varepsilon^2}
$$
定义指示变量：
$$
Y_i =
\begin{cases}
	1, & \text{if i}th\text{ Morris+ fails.} \\
	0, & \text{otherwise.}
\end{cases}
$$
其中 $\mathbb{P}(|Y_i - n| > \varepsilon n) < \frac{1}{3}$。如果中位数估计错误，那么至少有 $\frac{t}{2}$ 个 Morris+ 副本估计错误，因此：
$$
\begin{align*}
\mathbb{P}(\sum_{i}Y_i > \frac{t}{2}) &=
	\mathbb{P}(\sum_iY_i - \frac{t}{3} > \frac{t}{6}) \\ &=
	\mathbb{P}(\sum_iY_i - \mathbb{E}\sum_iY_i > \frac{t}{6}) \\ &\le
	\mathbb{P}(|\sum_iY_i - \mathbb{E}\sum_iY_i| > \frac{1}{2}\mathbb{E}\sum_iY_i) \tag{8} \\ &\le
	2 \cdot e^{-(\frac{1}{2})^2\frac{t}{9}} \tag{9} \\ &=
	2 \cdot e^{-\frac{t}{36}}
\end{align*}
$$
其中，$(8)\rightarrow (9)$ 利用了 Chernoff Bound。令 $2 \cdot e^{-\frac{t}{36}} = \delta$ 得到 $t = 36ln(\frac{\delta}{2})$。

综上所述，给定 $\varepsilon$ 和 $\delta$，取 $s = \frac{3}{2\varepsilon^2}$，$t = 36ln(\frac{\delta}{2})$，通过 Morris++ 就能取得：
$$
\mathbb{P}(|\tilde{n} - n| > \varepsilon n) < \delta
$$

## References

* Lecture 1: [notes](http://people.seas.harvard.edu/~minilek/cs229r/fall15/lec/lec1.pdf), [video](https://www.youtube.com/playlist?list=PL2SOU6wwxB0v1kQTpqpuu5kEJo2i-iUyf)
* Wikipedia: [Markov's inequality](https://en.wikipedia.org/wiki/Markov%27s_inequality), [Chebyshev's inequality](https://en.wikipedia.org/wiki/Chebyshev%27s_inequality), [Chernoff bound](https://en.wikipedia.org/wiki/Chernoff_bound)
* MIT RES.6-012 Introduction to Probability, Spring 2018, L18.2, [video](https://www.youtube.com/watch?v=vjYanZ1nsZg)

* [Counting Large Numbers of Events in Small Registers](https://www.inf.ed.ac.uk/teaching/courses/exc/reading/morris.pdf)

## TODO

* Bernstein 证明