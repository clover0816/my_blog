---
title: RSA
date: 2026-09-11 23:35:07
tags: [crypto, 学习笔记, 非对称加密]
categories: [学习笔记]
description: "RSA"
mathjax: true
---

## 核心原理

公钥 $(n, e)$, 私钥 $(n, d)$  
消息加密: $m^e \equiv c \pmod{n}$  
密文解密: $ed \equiv 1 \pmod{\varphi(n)}$, $m^{ed} \equiv c^{d} \equiv m \pmod{n}$ (证明: 欧拉定理)  
要求: $\gcd(e, \varphi(n)) = 1, \gcd(n, m) = 1$

相关基础文章:  

- [RSA算法基础详解](https://www.cnblogs.com/hykun/p/RSA.html)
- [阮一峰《RSA算法原理（一）》](https://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html)
- [阮一峰《RSA算法原理（二）》](https://www.ruanyifeng.com/blog/2013/07/rsa_algorithm_part_two.html)

## 攻击

### p, q, dp, dq 泄露, e 隐藏

已知 $p, q, c, dp, dq$, 其中 $d \equiv dp \pmod{\varphi(p)}, d \equiv dq \pmod{\varphi(q)}$  

$$
\begin{gathered}
d \equiv d_p \pmod{\varphi(p)}\\
d \equiv d_q \pmod{\varphi(q)}\\
d = d_p + k \varphi(p)\\
m^e \equiv c \pmod{n} \Rightarrow m^e \equiv c \pmod{p}\\
c^{d_p} = m^{ed_p} = m^{ed - ek\varphi(p)} \equiv m \pmod{p}\\
m \equiv c^{d_p} \pmod{p}\\
同理 m \equiv c^{d_q} \pmod{q}\\
使用 crt 合并求出 m\\
\end{gathered}
$$

解题脚本:

```sage
import gmpy2
from Crypto.Util.number import *

p = ...
q = ...
dp = ...
dq = ...
c = ...

re = [int(pow(c, dp, p)), int(pow(c, dq, q))]
mo = [p, q]
m = crt(re, mo)
f = long_to_bytes(m)
print(f)
```

### dp, e, n, c泄露

已知 $d \equiv dp \pmod{\varphi(p)}$

$$
\begin{gathered}
d \equiv d_p \pmod{\varphi(p)}\\
ed = e \cdot d_p + k \cdot \varphi(p)\\
ed = t \cdot \varphi(n) + 1\\
e \cdot d_p - 1 = t \cdot \varphi(n) - k \cdot \varphi(p) = (p - 1)[t(q - 1) - k]\\
\because d_p < (p - 1)\\
\therefore ed_p - 1 < e(p - 1) - 1\\
t(q - 1) - k < e - \dfrac{1}{p - 1}\\
\end{gathered}
$$

枚举 $t(q - 1) - k$ 的值即可, 范围 $(1, e)$  

解题脚本:

```sage
import gmpy2

e = ...
n = ...
dp = ...
c = ...
p = None
q = None

for i in range(2, e):
    if (dp * e - 1) % i == 0:
        p = (dp * e - 1) // i + 1
        if n % p == 0:
            q = n // p
            break;
d = gmpy2.invert(e, (p - 1) * (q - 1))
m = pow(c, d, n)
flag = long_to_bytes(m)
print(flag)
```

### 共模攻击

已知 $n, e_1, c_1, e_2, c_2$, 要求同时满足 $\gcd(e_1, e_2) = 1$.  
构造 $e_1 \cdot s_1 + e_2 \cdot s_2 = 1$  

$$
\begin{gathered}
m^{e_1 \cdot s_1} \equiv c_1^{s_1} \pmod{n}\\
m^{e_2 \cdot s_2} \equiv c_2^{s_2} \pmod{n}\\
c_1^{s_1} \cdot c_2^{s_2} \equiv m \pmod{n}\\
\end{gathered}
$$

可以使用扩展欧几里得算法来求解 $s_1, s_2$, 先找到特解:

$$
\begin{gathered}
e_1 \cdot (s_1 + s_2) + (e_2 - e_1) \cdot s_2 = 1\\
s_2 \cdot (e_2 - e_1) \equiv 1 \pmod{e_1}\\
s_2 \equiv (e_2 - e_1)^{-1} \pmod{e_1}\\
\end{gathered}
$$

之后求解 $s_1$ 即可.  
通解:

$$
\begin{gathered}
t_1 = s_1 + t \cdot \dfrac{e_2}{\gcd(e_1, e_2)}\\
t_2 = s_2 - t \cdot \dfrac{e_1}{\gcd(e_1, e_2)}\\
\end{gathered}
$$

```sage
import gmpy2
import libnum

n = ...
c1 = ...
c2 = ...
e1 = ...
e2 = ...

if e1 < e2:
    e1, e2 = e2, e1
    c1, c2 = c2, c1

s2 = gmpy2.invert(e2 - e1)
s1 = (1 - e2 * s2) // e1
m = pow(c1, s1, n) * pow(c2, s2, n) % n
flag = long_to_bytes(m)
print(flag)
```

### 低加密指数广播攻击

$e$ 比较小, 会有多组 $(c_i, n_i)$.  

$$
m^e \equiv c_i \pmod{n_i}
$$

使用 crt 进行合并就行了得到 $m^e$, 由于 $e$ 比较小, 可以直接开方得到 $m$.  

### 维纳攻击

要求: $d < \dfrac{1}{3}N^{\frac{1}{4}}$.  
在满足要求的条件下, 则可以分解 $N$.  
对于 $ed \equiv 1 \pmod{\varphi(n)}$, $ed = k \cdot \varphi(n) + 1$, 当 $e$ 很大时, $d$ 就会相应的比较小.  

$$
\begin{gathered}
ed = k \cdot \varphi(n) + 1\\
e = \dfrac{k \cdot \varphi(n) + 1}{d}\\
ed - kn = (ed - k \cdot \varphi(n)) + (k \cdot \varphi(n) - kn) = 1 + k[(p - 1)(q - 1) - pq]\\
ed - kn = 1 + k(1 - p - q)\\
\dfrac{e}{n} - \dfrac{k}{d} = \dfrac{1 + k(1 - p - q)}{n \cdot d}\\
|\dfrac{e}{n} - \dfrac{k}{d}| = \dfrac{k(p + q - 1) - 1}{n \cdot d} < \dfrac{k(p + q)}{n \cdot d}\\
p + q \approx 2\sqrt{n} < 3\sqrt{n}\\
k = \dfrac{ed - 1}{\varphi(n)} < \dfrac{ed}{\varphi(n)} < d\\
|\dfrac{e}{n} - \dfrac{k}{d}| < \dfrac{p + q}{n} < \dfrac{3}{\sqrt{n}}\\
d < \dfrac{1}{3}n^{\frac{1}{4}} \longrightarrow \dfrac{1}{\sqrt{n}} < \dfrac{1}{9d^2}\\
|\dfrac{e}{n} - \dfrac{k}{d}| < \dfrac{1}{3d^2} < \dfrac{1}{2d^2}\\
\end{gathered}
$$

Legendre 定理: 如果 $\left|x - \dfrac{a}{b} \right| < \dfrac{1}{2d^2}$, 则 $\dfrac{a}{b}$ 一定是 $x$ 的一个渐进分数.  
则由 Legendre 定理得: $\dfrac{k}{d}$ 一定是 $\dfrac{e}{n}$ 的一个渐进分数.  
我们只需要枚举 $\dfrac{e}{n}$ 的渐进分数，测试 $\dfrac{ed - 1}{k}$ 是否为整数.
求连分数代码:  

```sage
def get_lfs(fz, fm):
    a = []
    while fm :
        g = fz // fm
        a.append(g)
        fz, fm = fm, fz - g * fm
    return a
```

具体原理见[百度百科](https://baike.baidu.com/item/%E8%BF%9E%E5%88%86%E6%95%B0/2715871).  
求渐进连分数代码:  

```sage
def get_close_lfs(a):
    b = []
    h0, h1 = 0, 1
    k0, k1 = 1, 0
    for i in a:
        h2 = i * h1 + h0
        k2 = i * k1 + k0
        b.append((h2, k2))
        h0, k0 = h1, k1
        h1, k1 = h2, k2
    return b
```

### e 与 ɸ(n) 不互质

要求能够分解 $n$.  
$n = p \cdot q$.  

#### AMM 算法

AMM 算法对于这类问题是比较通用的.  
后面会写一篇文章专门讲这个算法.  

### e 与 p - 1 或 q - 1 互质

当 $e$ 与 $\varphi{(p)}$ 或 $\varphi{(q)}$ 互质, 而 $p$、$q$ 比 m 大, 可将 $\varphi{(n)}$ 转化为 $\varphi{(p)}$ 或 $\varphi{(q)}$ 进行计算.  

### e = 2

二次剩余 + CRT

前置知识: [二次剩余](https://www.cnblogs.com/yifan0305/p/22508764)

已知 $m^2 \equiv c \pmod{n}$, 则可以得知 $m^2 \equiv c \pmod{p}, m^2 \equiv c \pmod{q}$, 对模 $p$ 与模 $q$ 各求二次根, 得到 $m1_p, m2_p, m1_q, m2_q$, 进行$\{m1_p, m1_q\}, \{m1_p, m2_q\}, \{m2_p, m1_q\}, \{m2_p, m2_q\}$ 四种情况的 CRT, 最后得到 $4$ 个预选 $m$, 进行验证得到正确的 $m$.
