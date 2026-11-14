---
title: ECC
date: 2026-10-03 22:15:06
tags: [crypto, 学习笔记, 非对称加密, 离散对数]
categories: [学习笔记]
description: "ECC"
mathjax: true
---

## 群、域

ECC 的核心是椭圆曲线上的点构成一个群.  
一个集合 $G$ 和一个运算 $*$, 如果满足:  

1. 封闭性: $a, b \in G$, $a * b \in G$.  
2. 结合律: $(a * b) * c = a * (b * c)$.  
3. 单位元: 存在 $e$, 使得 $e * a = a * e = a$.  
4. 逆元: 每个 $a$ 都有 $a^{-1}$, 使 $a * a^{-1} = e$.  

那么称 $(G, *)$ 是群.  
如果还满足交换律 $a * b = b * a$, 就叫阿贝尔群, 也叫交换群. 椭圆曲线点群就是阿贝尔群.  
如果群中所有元素都能写成某个元素 $g$ 的幂或倍数: $G = \left\{e, g, g^2, g^3, \cdots \right\}$, 则称 $G$ 是循环群, $g$ 是生成元.  
在椭圆曲线中, 点加法写成倍数: $P = P, 2P = P + P, 3P = P + P + P, \cdots$, 如果存在最小正整数 $n$, 使 $nP = O$, 那么 $n$ 是点 $P$ 的阶, 这里的 $O$ 是无穷远点, 相当于单位元.  

---

域是一种集合, 加法和乘法都能良好运算, 除法也有定义.
> 最常见的是素域: $\mathbb{F}_p = \left\{0, 1, 2, \cdots, p - 1\right\}$.  

## 椭圆曲线

函数: $y^2 = x^3 + ax + b$, 同时要求 $4a^3 + 27b^2 \not= 0$.  
椭圆曲线的加法定义:  
对于曲线上两点 $P, Q$, 画直线经过 $P$ 和 $Q$, 直线与曲线交于第三点 $R'$, 此时定义 $P + Q + R' = O$, $R'$ 关于 $x$ 轴对称过去得到的点就是 $R$, $R = -R' = P + Q$.  
如果 $P = Q$, 那么就画 $P$ 的切线, 如果 $P, Q$ 关于 $x$ 轴对称, 那么 $P + Q = O$.  

### 有限域上的椭圆曲线

$E\left(\mathbb{F}_p\right): y^2 \equiv x^3 + ax + b \pmod{p}$, 其中 $4a^3 + 27b^2 \not\equiv 0 \pmod{p}$.  
点集包括了所有满足方程的 $x, y$, 同时包括无穷远点 $O$.
加法公式:  
设 $P(x_1, y_1), Q(x_2, y_2)$, 且 $P \not = \pm Q$, 斜率为 $\lambda = \dfrac{y_2 - y_1}{x_2 - x_1} \pmod{p}$.  
我们可以得到 $x_3 = \lambda^2 - x_1 - x_2 \pmod{p}, y_3 = \lambda(x_1 - x_3) - y_1 \pmod{p}$.  
证明过程:  

$$
\begin{gathered}
设 PQ: y = \lambda x + v \pmod{p}\\
代入原方程: (\lambda x + v)^2 = x^3 + ax + b \pmod{p}\\
x^3 - \lambda^2x^2 + (a - 2\lambda v)x + b - v^2 = 0 \pmod{p}\\
由韦达定理可得 x_1 + x_2 + x_3 = \lambda^2.\\
由此得到 x_3 = \lambda^2 - x_1 - x_2\\
斜率公式可得 \dfrac{y_3' - y_1}{x_3 - x_1} = \lambda\\
y_3' = \lambda(x_3 - x_1) + y_1\\
此时得到 R' 的 y 坐标, R 关于 x 轴与 R' 对称.\\
y_3 = \lambda(x_1 - x_3) - y_1\\
由此, x_3 = \lambda^2 - x_1 - x_2 \pmod{p}\\
y_3 = \lambda(x_1 - x_3) - y_1 \pmod{p}\\
\end{gathered}
$$

如果 $P = Q$, 那么斜率为 $\lambda = \dfrac{3x_1^2 + a}{2y_1}$ (求导即可证明).  

## ECC 的核心困难问题

我们知道, 一个加密算法想要抵抗暴力破解, 其破解过程都是建立在一个困难问题上的, ECC 的困难问题就是:  

$$
对于点 P, 有 Q = kP, 已知 Q, P, 求解 k 十分困难.
$$

这就是 **ECDLP: Elliptic Curve Discrete Logarithm Problem** (椭圆曲线离散对数问题), ECC 的安全性就建立在此之上.  

## ECC 密码协议

曲线参数: $\left(p, a, b, G, n, h \right)$.  
其中: $p$ 为有限域大小, $a, b$ 为椭圆曲线参数, $G$ 为基点, $n$ 为基点 $G$ 的阶数, $h$ 是余因子.  
> $n \times h = 椭圆曲线上点的总数$, $h$ 越小越好, 当 $h$ 很大时容易受到林-李主动小子群攻击 (Lim-Lee Active Small Subgroup Attack)

私钥为一个随机数: $d \in \left\{1, 2, 3, \cdots, n - 1\right\}$.  
公钥是 $Q = dG$

## ECDH 密钥交换

Alice 的私钥为 $d_a$, 公钥为 $Q_a = d_aG$.  
Bob 的私钥为 $d_b$, 公钥为 $Q_b = d_bG$.  
Alice 计算: $S = d_a Q_b = d_a d_b G$.  
Bob 计算: $S = d_b Q_a = d_a d_b G$.  
两人得到一个共享点 $S$, 通常取 $S$ 的 $x$ 坐标, 再经过 KDF 得到对称密钥.  
> KDF 的核心是哈希函数, 将 $S$ 的 $x$ 坐标作为输入, 经过哈西运算, 输出固定长度的伪随机字节串.  

## ECDSA 签名

### 签名

有签名者私钥 $d$, 公钥 $Q = dG$.  
对于消息哈希 $z$, 随机选 $k$, 计算 $R = kG$, 取 $r = R.x \bmod{n}$, 计算 $s = k^{-1}(z + rd) \bmod{n}$.  
签名即为 $\left(r, s\right)$
> 如果 $r, s$ 为 $0$, 必须重新选择 $k$ 重新计算.  
> $r$ 等于 $0$ 时, 签名验证则用不上公钥, 私钥也失去了它的作用
> $s$ 等于 $0$ 时, 则会导致私钥 $d$ 泄露!

### 验证

$$
\begin{gathered}
w = s^{-1} \bmod{n}\\
u_1 = zw \bmod{n}\\
u_2 = rw \bmod{n}\\
X = u_1G + u_2Q\\
X = zs^{-1}G + rs^{-1}dG = s^{-1}(z + rd)G =kG = R\\
\end{gathered}
$$

此时验证 $X.x \bmod{n}$ 是不是等于 $r$ 即可.  
在此过程中, $k$ 绝对不可以泄露, 一旦 $k$ 泄露, 即可算出 $d = (s \cdot k - z) \cdot r^{-1} \bmod{n}$, 此时将 $s, k, z, r$ 代入可以得到私钥 $d$, 私钥泄露!  

## 椭圆曲线 ElGamal (EC ElGamal)

将上面的有限域乘法群换成椭圆曲线点群, 就得到了 EC ElGamal.  

### 密钥生成

曲线参数: $(p, a, b, G, n)$.  
私钥 $d$, 公钥 $Q = dG$.  

### 加密

先将消息 $m$ 编码成椭圆曲线上的一个点 $M$ (消息到点的映射), 这一步比较麻烦.  
随机选择临时数 $k$, 计算:  

$$
\begin{gathered}
c_1 = kG
c_2 = M + kQ
\end{gathered}
$$

密文: $(c_1, c_2)$

### 解密

接收方用私钥 $d$, 计算: $M = c_2 - c_1 \cdot d$, 得到 $M$ 解码回 $m$.  

## 攻击

### ECDLP 中的 Pohlig-Hellman 攻击

这个攻击专门针对**群阶光滑**的离散对数问题, 在椭圆曲线中就是: 如果椭圆曲线的阶 $N$ 可以被分解成很多个小素数幂的乘积, 那么 ECDLP 可以被快速破解.  
我们称可以被分解为小素数乘积的正整数为光滑数. 即:  

$$
N = \prod_{i = 1}^{m}p_i^{t_i}
$$

我们设 $G$ 的阶为 $N$, 想求 $k \bmod{N}$, 根据中国剩余定理, 只需要求:  

$$
\begin{gathered}
\left\{\begin{matrix}
k \bmod{p_1^{t_1}}\\
k \bmod{p_2^{t_2}}\\
\cdots\\
k \bmod{p_m^{t_m}}\\
\end{matrix}
\right.
\end{gathered}
$$

最后将这些方程合并, 即可得到 $k$.  
对于每个因子 $p_i^{t_i}$, 令 $N = p_i^{t_i} \cdot q, \gcd(p, q) = 1$.  
定义 $G' = qG, Q' = qQ$, 那么 $G'$ 的阶为 $p_i^{t_i}$, 并且有 $Q' = qQ = qkG = k(qG) = kG'$, 所以在由 $G'$ 生成的子群里, 有 $Q' = kG'$, 子群的阶为 $p_i^{t_i}$, 这个子群的阶只有 $p_i^{t_i}$, 比原来的 $N$ 要小很多, 现在只要求 $k \equiv k_i \pmod{p_i^{t_i}}$, $k_i$ 是 $Q$ 相对于 $G$ 的离散对数, 对每个 $p_i^{t_i}$ 单独求解, 最后用 CRT 合并即可.  
sagemath 中的 `discrete_log(Q, G, ord=N, operation='+')` 内部就是 Pohlig-Hellman.  
> 对于一些曲线 $y^2 = x^3 + ax \pmod{p}$, 如果 $p \equiv 3 \pmod{4}$, 那么这条曲线就是超奇异的, 它的阶恰好是 $p + 1$.  
