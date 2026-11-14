---
title: Elgamal
date: 2026-11-13 00:22:49
tags: [crypto, 学习笔记, 非对称加密, 离散对数]
categories: [学习笔记]
description: "Elgamal"
mathjax: true
---

## 群、域

详细部分可以看 [初探群论 | clover's blog](https://clover0816.github.io/my_blog/2026/10/25/初探群论/). 这里再简单叙述一下.
一个集合 $G$ 和一个运算 $*$, 如果满足:

1. 封闭性: $a, b \in G$, $a * b \in G$.
2. 结合律: $(a * b) * c = a * (b * c)$.
3. 单位元: 存在 $e$, 使得 $e * a = a * e = a$.
4. 逆元: 每个 $a$ 都有 $a^{-1}$, 使 $a * a^{-1} = e$.

那么称 $(G, *)$ 是群.
如果还满足交换律 $a * b = b * a$, 就叫阿贝尔群, 也叫交换群. 有限域乘法群 $\mathbb{F}_p^*$ 就是阿贝尔群.
如果群中所有元素都能写成某个元素 $g$ 的幂: $G = \left\{e, g, g^2, g^3, \cdots \right\}$, 则称 $G$ 是循环群, $g$ 是生成元.
在有限域中, 幂写成 $g^k \bmod p$, 如果存在最小正整数 $n$, 使 $g^n \equiv 1 \pmod p$, 那么 $n$ 是 $g$ 的阶.

域是一种集合, 加法和乘法都能良好运算, 除法也有定义.
最常见的是素域: $\mathbb{F}_p = \left\{0, 1, 2, \cdots, p - 1\right\}$.

## ElGamal

给定素数 $p$, 生成元 $g$, 公钥 $y = g^x  \bmod{p}$, 求私钥 $x$ 很难, ElGamal 的全部安全性, 都建立在 "求离散对数很难" 这个问题上.

### 原始 ElGamal (有限域版本)

#### 密钥生成

选取一个大质数 $p$, 以及模 $p$ 的生成元 $g$, 随机选取私钥 $x, 2 \le x \le p - 2$, 计算公钥 $y = g^x \bmod{p}$.
公钥: $(p, g, y)$, 私钥: $x$.

#### 加密

把明文 $m$ 加密给接收方, 随机选择一个数 $k, 2 \le k \le p - 2$, 接收方的公钥为 $(p, g, y)$.
计算 $c_1 = g^k \bmod{p}, c_2 = m \cdot y^k \bmod{p}$.
密文就是 $(c_1, c_2)$.

#### 解密

接收方用自己的私钥 $x$, 计算 $c_1^x \bmod{p}$.

$$
\begin{gathered}
c_1^x = (g^k)^x = g^{kx} = y^k\\
m = c_2 \cdot (c_1^{x})^{-1} \bmod{p}\\
\end{gathered}
$$

> 随机数 $k$ 必须每次重新选取，不能重用。  
> 如果两次加密使用同一个 $k$，攻击者可以通过 $c_1$ 相同发现，并进一步恢复明文或私钥。

### 原始 ElGamal 签名 (有限域版本)

密钥生成: 私钥 $x$, 公钥 $y = g^x \bmod{p}$

#### 签名

对消息 $m$ 进行签名, 计算消息哈希 $H(m)$, 随机选择 $k, 2 \le k \le p - 2$, 且 $\gcd(k, p - 1) = 1$.
计算:
$$
\begin{gathered}
r = g^k \bmod{p}\\
s = (H(m) - x \cdot r) \cdot k^{-1} \bmod{p - 1}\\
\end{gathered}
$$

签名就是 $(r, s)$.

#### 验证

任何人拿到签名 $(r, s)$ 和公钥 $y$, 计算:

$$
\begin{gathered}
v_1 = g^{H(m)} \bmod{p}\\
v_2 = y^r \cdot r^s \bmod{p}\\
v_2 = g^{xr} \cdot g^{ks} = g^{sk + xr} = g^{H(m)} \pmod{p}\\
\end{gathered}
$$

因此 $v_1 = v_2$ 时, 签名有效.

### 原始 ElGamal 代码

```py
from Crypto.Util.number import *
import libnum
import gmpy2

p = getPrime(1024)          # 大素数 p
g = primitive_root(p)       # 模 p 的生成元
x = randint(2, p - 2)       # 私钥 x
y = int(pow(g, x, p))       # 公钥 y = g^x mod p

print(f"公钥: (p, g, y) = ({p}, {g}, {y})")
print(f"私钥: x = {x}")

# 加密
m = 12345                    # 明文（要求 1 <= m < p）
k = randint(2, p - 2)        # 每次加密的随机数 k

c1 = int(pow(g, k, p))              # c1 = g^k mod p
c2 = (m * int(pow(y, k, p))) % p    # c2 = m * y^k mod p
print(f"\n明文 m = {m}, 随机数 k = {k}")
print(f"密文 (c1, c2) = ({c1}, {c2})")

# 解密
s = pow(c1, x, p)              # s = c1^x = g^(kx) = h^k mod p
m2 = (c2 * inverse_mod(s, p)) % p    # m = c2 * s^(-1) mod p
print(f"解密结果 = {m2},  正确? {m2 == m}")

print("end")
```

### 原始 ElGamal 签名代码

```py
from Crypto.Util.number import *
import libnum
import gmpy2
import hashlib

# 密钥生成
p = getPrime(1024)               # 大素数 p
g = primitive_root(p)            # 模 p 的生成元
x = randint(2, p - 2)            # 私钥 x
y = int(pow(g, x, p))            # 公钥 y = g^x mod p

print(f"公钥: (p, g, y) = ({p}, {g}, {y})")
print(f"私钥: x = {x}")

# 签名
msg = b"Hello ElGamal"
Hm = Integer(int(hashlib.sha256(msg).hexdigest(), 16))   # 把消息哈希成整数

# 选 k：必须 gcd(k, p-1) = 1，才能对 k 求逆
while True:
    k = randint(2, p - 2)
    if gcd(k, p - 1) == 1:
        break

r = int(pow(g, k, p))                                  # r = g^k mod p
s = ((Hm - x * r) * gmpy2.invert(k, p - 1)) % (p - 1)  # s 在模 p-1 下计算
print(f"\n消息 m = {msg.decode()}")
print(f"哈希 H(m) = {Hm % (p-1)}")
print(f"签名 (r, s) = ({r}, {s})")

# 验证
left = int(pow(g, Hm, p))                 # g^H(m) mod p
right = (int(pow(y, r, p)) * int(pow(r, s, p))) % p   # y^r * r^s mod p
print(f"\n验证: g^H(m) == y^r * r^s ?  {left == right}")
```
