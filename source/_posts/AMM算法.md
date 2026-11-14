---
title: AMM算法
date: 2026-09-26 21:42:37
tags: [crypto, 学习笔记, 非对称加密]
categories: [学习笔记]
description: "AMM算法"
mathjax: true
---

解决 $e$ 与 $p$ 不互质的问题, 可以使用 AMM 算法.  

## AMM 开二次根

对于奇质数 $p$, $p - 1 = 2^{t} \cdot s$, 其中 $s$ 是奇数.  
对于 $x^2 \equiv 1 \pmod{p}$, 则 $x^{\frac{p - 1}{2}} \equiv 1 \pmod{p}$,  

$$
x^{\frac{p - 1}{2}} \equiv x^{2^{t - 1} \cdot s} \equiv 1 \pmod{p}\\
$$

### 当 t = 1 时

此时 $x^{s} \equiv 1 \pmod{p}$, 两边同乘 $x$, 在同时开方可以得到 $x^{\frac{s + 1}{2}} \equiv x^{\frac{1}{2}} \pmod{p}$, 此时代入 $c$, 就可以得到 $c^{\frac{1}{2}} \equiv c^{\frac{s + 1}{2}} \pmod{p}$, 又有 $m^2 \equiv c \pmod{p}$, 因此 $m \equiv c^{\frac{1}{2}} \pmod{p}$, 得到 $m$.  

### 当 t > 1 时

此时 $p - 1$ 有 $2$ 个即以上的 $2$, 此时想要消掉 $t$ 就有些麻烦了.  

$$
\begin{gather}
x^{2^{t - 1} \cdot s} \equiv 1 \pmod{p}\\
x^{2^{t - 2} \cdot s} \equiv 1 \pmod{p} 或 x^{2^{t - 2} \cdot s} \equiv -1 \pmod{p}\\
\end{gather}
$$

当我们开方后得到 $-1$, 可以找一个非二次剩余 $y$, $y^{2^{t - 1} \cdot s} \equiv -1 \pmod{p}$, $x^{2^{t - 2} \cdot s} \cdot y^{2^{t - 1} \cdot s \cdot k} \equiv 1 \pmod{p}$, 在开方得到 $1$ 时 $k = 0$, 在开方得到 $-1$ 时 $k = 1$, 一直开方下去, 最终可以得到 $x^s \cdot y^{s \cdot (2k_1 + 2^2k_2 + \cdots + 2^{t-1}k_{t-1})} \equiv 1 \pmod{p}$, 两边同乘 $x$ 再同时开方得到 $x^{\frac{s + 1}{2}} \cdot y^{s \cdot (k_1 + 2k_2 + \cdots + 2^{t - 2}k_{t - 1})} \equiv x^{\frac{1}{2}} \pmod{p}$.  
代入 $c$ 即可得到 $m$, $m \equiv c^{\frac{s + 1}{2}} \cdot y^{s \cdot (k_1 + 2k_2 + \cdots + 2^{t - 2}k_{t - 1})} \pmod{p}$.  

伪代码:  

分解 $p - 1$, 得到 $p - 1 = 2^{t} \cdot s$  
$val = c^{2^{t - 1} \cdot s}$  
$cnt = t - 1$  
$while\quad cnt:$  
$\qquad if \quad \sqrt{val} \equiv 1 \pmod{p}:$  
$\qquad \qquad k = 0$  
$\qquad \qquad val = \sqrt{val} \cdot y^{2^{t - 1} \cdot s \cdot k} \bmod{p}$
$\qquad else:$  
$\qquad \qquad k = 1$  
$\qquad \qquad val = \sqrt{val} \cdot y^{2^{t - 1} \cdot s \cdot k} \bmod{p}$  
$\qquad cnt --$  
$val = val \cdot c \bmod{p}$  
$m = \sqrt{val} \bmod{p}$

## AMM 开 e 次方根

这里我们要分两种情况考虑

### gcd(e, p - 1) = 1

这种是简单的 RSA, 存在逆元可以直接求逆元.  

### e | p - 1

我们先令 $p - 1 = e^t \cdot s$ (相当于将 $2$ 换成了 $e$)  

$$
\begin{gather}
x^{p - 1} \equiv 1 \pmod{p}\\
(x^e)^{\frac{p - 1}{e}} \equiv c ^{\frac{p - 1}{e}} \equiv 1 \pmod{p}\\
\end{gather}
$$

现在找一个最小的非负整数 $d$, 使得 $s \mid ed - 1$, 可以认为 $d$ 是模 $s$ 下 $e$ 的逆元.  
$c^{\frac{p - 1}{e}} \equiv (c^s)^{e^{t - 1}} \equiv 1 \pmod{p}$  
两边同时 $k$ 次方可以得到 $(c^{ed - 1})^{e^{t - 1}} \equiv 1^k \equiv 1 \pmod{p}$.  

#### 当 t = 1 时

此时上式就等于 $c^{ed - 1} \equiv 1 \pmod{p}$, 两边同时乘 $c$, 可以得到 $c^{ed} \equiv c \pmod{p}$, 此时 $c^d$ 就是我们要找的 $e$ 次根.  

#### 当 t > 1 时

当 $t > 1$ 时, 我们无法像 $t=1$ 那样直接获得 $c^d$ 作为根, 因为 $c^{ed-1} \not\equiv 1 \pmod{p}$, 而只是 $c^{e^{t-1}(ed-1)} \equiv 1 \pmod{p}$. 这说明 $c^{ed-1}$ 的阶是 $e^{t-1}$ 的因子, 即：

$$
c^{ed-1} \equiv a^{k} \pmod{p}
$$

其中 $a$ 是某个阶为 $e^t$ 的元素, $0 \le k < e^{t-1}$, 这个 $a$ 可以由一个非 $e$ 次剩余 $z$ 构造:  

$$
a \equiv z^s \pmod{p}
$$

则 $a^{e^t} \equiv 1 \pmod{p}$, 且 $a^{e^{t-1}} \not\equiv 1 \pmod{p}$.  
于是可以把 $c^{ed-1}$ 写成 $a^{k}$ 的形式, 要得到 $e$ 次根, 我们需要从 $c^{ed-1}$ 中逐层提取 $e$ 的幂次. 具体地, 令 $b_0 = c^{ed-1}$, 由于 $b_0^{e^{t-1}} \equiv 1 \pmod{p}$, 所以 $b_0$ 的阶整除为 $e^{t-1}$.  
然后对于 $i = 0, 1, \dots, t-2$, 计算: $b_i^{e^{t-2-i}}$.  
这个值要么是 $1$, 要么是某个非 $1$ 的 $e$ 次单位根, 如果它是 $1$, 说明当前层已经消干净; 如果不是 $1$, 它就必然等于 $a^{k_i e^{t-2-i}}$ 的形式(因为所有阶为 $e$ 的幂次的元素都在 $\langle a \rangle$ 中).  
此时令 $k_i$ 为该值对应的指数, 然后更新:  

$$
b_{i+1} = b_i \cdot a^{-k_i e^{t-1-i}}
$$

这样做的效果是消去 $b_i$ 中阶为 $e^{t-1-i}$ 的因子, 使 $b_{i+1}$ 的阶下降到 $e^{t-2-i}$ 或更低, 最终经过 $t-1$ 步, 得到 $b_{t-1} \equiv 1 \pmod{p}$.  
在这个过程中, 我们同时修正初始猜测 $c^d$:  
令 $x_0 = c^d$, 每次当 $b_i^{e^{t-2-i}} = a^{k_i e^{t-2-i}}$ 时, 将 $x_i$ 更新为:  

$$
x_{i+1} = x_i \cdot a^{-k_i e^{t-2-i}}
$$

这样在每一步都保持:  

$$
x_i^e \equiv b_i \cdot c \pmod{p}
$$

当 $b_{t-1} \equiv 1 \pmod{p} $时, 就得到:  

$$
x_{t-1}^e \equiv c \pmod{p}
$$

因此最终的 $e$ 次根为:  

$$
m \equiv x_{t-1} \equiv c^d \cdot \prod_{i=0}^{t-2} a^{-k_i e^{t-2-i}} \pmod{p}
$$

其中每个 $k_i$ 由 $b_i^{e^{t-2-i}} = a^{k_i e^{t-2-i}}$ 唯一确定 (若 $b_i^{e^{t-2-i}} \equiv 1$, 则 $k_i = 0$).  
