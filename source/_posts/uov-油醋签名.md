---
title: uov 油醋签名
date: 2026-09-18 22:17:43
tags: [crypto, 学习笔记, 后量子密码]
categories: [学习笔记]
description: "uov-油醋签名"
mathjax: true
---

UOV, 非平衡油醋, 是一种基于多变量二次方程组的数字签名方案, 是后量子密码中多变量密码方向最具代表性的数字签名方案之一, 其安全性保证在于求解多变量二次方程组解的困难性, 目前该问题已被证明是 NP 难的.  

## 核心设计思想: 油醋变量的不平衡性

UOV 的核心创新在于将变量分为两类: 油变量和醋变量, 并利用它们之间的不对称关系构造一个容易求逆的陷门.  
在原始的 OV 方案中, 油变量与醋变量数量相等, 但该方案后来被不变子空间破解, UOV 的改进在于使醋变量多余油变量, 即非平衡, 从而有效抵抗攻击.  
设有限域 $\mathbb{F}_q$, 有 $v$ 个醋变量, $o$ 个油变量, $n = v + o$, 而中心映射 $\mathcal{f} = (f_1, f_2, \cdots, f_m)$ 是一组特殊构造的二次多项式, 其关键性质是油变量的二次项系数为 $0$. 而这个性质带来的关键性质就是如果我们固定了醋变量的值, 则这个多远二次方程就退化成了多元一次方程(关于油变量的线性方程), 可以通过高斯消元求解. 这是 uov 陷门可逆性的数学基础.  

## 密钥生成

私钥: 包含中心映射 $\mathcal{F}$ 和一个随机可逆线性变化 $\mathcal{T}$.  
公钥: 公钥是复合映射 $\mathcal{P} = \mathcal{F} \circ \mathcal{T}$, 其中 $\mathcal{T}$ 的作用是"隐藏"油空间, 使得攻击者无法从公钥中直接识别出哪些变量是油变量, 哪些是醋变量.  
举个🌰:  
如果没有 $\mathcal{T}$, 我们假设 $\mathcal{F} = x_1x_2 + x_1x_3 + x_2x_3$, 此时 $\mathcal{P} = \mathcal{F}$, 我们可以直接看出 $x_3$ 是油变量, $x_1, x_2$ 是醋变量, 从而得到油空间, 可以自己构造签名.  
而 $\mathcal{T}$ 是一个随机可逆线性变化, 令公开坐标为 $(s_1, s_2, s_3)$, 秘密坐标为 $(x_1, x_2, x_3)$, 定义:  

$$
\left \{ \begin{matrix}
x_1 = s_1 + s_2\\
x_2 = s_2 + s_3\\
x_3 = s_1 + s_3\\
\end{matrix} \right.
$$

写成矩阵为 $T = \begin{bmatrix}1 & 1 & 0\\0 & 1 & 1\\1 & 0 & 1\\\end{bmatrix}$, 这样我们看到的 $P = s_1^2 + s_2^2 + s_3^2 + 3(s_1s_2 + s_1s_3 + s_2s_3)$, 此时 $s_1, s_2, s_3$ 是等价的, 我们无法直接判断哪个是油变量, 因此实现了"隐藏"油空间.  

## 签名生成与验证

### 签名生成

首先我们得到消息 $m$, 进行哈希 $t = H(m)$, 这个 $t$ 是方程组需要等于的目标值, 我们在有限域内随机选取 $v$ 个醋变量 $x_1, x_2, \cdots, x_v$ 的值, 代入方程组中求解出油变量 $x_{v+1}, \cdots, x_{n}$ 的线性方程组, 最后我们将醋变量与油变量拼凑在一起, 得到完整的秘密坐标解向量 $x = (x_1, \cdots, x_v, x_{v+1}, \cdots, x_n)$, 此时 $x$ 满足了 $\mathcal{F}(x) = t$.  
利用私钥中的 $\mathcal{T}^{-1}$, 将秘密解 $x$ 转化为公开坐标 $s$, 即 $s = \mathcal{T}^{-1}(x)$, 这个 $s$ 就是公开签名.  

### 签名验证

签名者将 $(m, s)$ 发送给验证者, 验证者用公钥 $\mathcal{P}$ 进行验证, 计算公钥值 $\mathcal{P}(s) = \mathcal{F}(\mathcal{T}(s))$ 是否等于哈希值 $t' = H(m)$, 相等则验证通过.  

## 代码

签名生成代码:  

```sage
from Crypto.Util.number import *
import random

q = 31
v = 4
o = 3
n = v + o
F = GF(q)
m = o
R = PolynomialRing(F, n, 'x') # 生成在有限域 F 中, 含有 n 个变量的多项式, 变量名的前缀是 x
x = R.gens() # 把所有变量提取出来, 打包成元组
S = PolynomialRing(F, n, 's')
s = S.gens()
print(f"UOV 签名演示 (q = {q}, v = {v}, o = {o}, n = {n})")

# 0, 1, 2, 3 是醋变量, 4, 5, 6 是油变量
F_polys = [
    x[0]*x[1] + 2*x[0]*x[4] + 3*x[1]*x[4] + x[1]*x[5] + x[2]*x[4] + 5*x[0] + 7*x[4] + 9 * x[6] + 12,
    x[0]*x[2] + x[1]*x[5] + 4*x[2]*x[5] + 2*x[3]*x[4] + 3*x[3]*x[6] + 8*x[1] + 6*x[5] + 20,
    x[0] * x[1] + x[2] * x[4] + 5 * x[3] * x[6] + 21
] # 构造 m 个方程
# for i, p in enumerate(F_polys):
#     print(f"F{i + 1} = {p}")

# 生成 T
while True:
    T = random_matrix(F, n, n)
    if T.is_invertible():
        break

# 生成公钥 P = F(T(s))
subs_dict = {}
for i in range(n):
    subs_dict[x[i]] = sum(T[i, j] * s[j] for j in range(n))
# 到此T完成转化
P_polys = [F_polys[i].subs(subs_dict) for i in range(o)]
print("公钥已生成")
for i, p in enumerate(P_polys):
    print(f"P{i + 1} = {p}")

# 签名生成
message_hash = [12, 25, 27]
print(f"消息哈希 t = {message_hash}")

# 拒绝采样
while True:
    # 随机选择醋变量的值
    y_vals = [F.random_element() for _ in range(v)]
    # 代入醋变量的值, F 退化为线性方程组
    F_y = [F_polys[i].subs({x[j]: y_vals[j] for j in range(v)}) for i in range(o)]
    A = matrix(F, o, o)
    b = vector(F, o)
    for i in range(o):
        # 常数项移到右边
        b[i] = F(message_hash[i]) - F_y[i].subs({x[v + j]: 0 for j in range(o)})
        # 构造系数矩阵
        for j in range(o):
            A[i, j] = F_y[i].monomial_coefficient(x[v + j])
    # A 能够求解
    if A.is_invertible():
        x_oil = A.solve_right(b)
        break

x_secret = vector(F, y_vals + list(x_oil))
print(f"秘密完整解: {x_secret}")
s_sig = T.inverse() * x_secret
print(f"最终公开签名: {s_sig}")
print("至此数字签名完成")
```

签名验证代码:  

```sage
# 验证环节
verify_res = [P_polys[i].subs({s[j]: s_sig[j] for j in range(n)}) for i in range(o)]
if verify_res == [F(val) for val in message_hash]:
    print("验证通过!")
else:
    print("验证失败!!")
```

总代码:  

```sage
from Crypto.Util.number import *
# import gmpy2
import random

q = 31
v = 4
o = 3
n = v + o
F = GF(q)
m = o
R = PolynomialRing(F, n, 'x') # 生成在有限域 F 中, 含有 n 个变量的多项式, 变量名的前缀是 x
x = R.gens() # 把所有变量提取出来, 打包成元组
S = PolynomialRing(F, n, 's')
s = S.gens()
print(f"UOV 签名演示 (q = {q}, v = {v}, o = {o}, n = {n})")

# 0, 1, 2, 3 是醋变量, 4, 5, 6 是油变量
F_polys = [
    x[0]*x[1] + 2*x[0]*x[4] + 3*x[1]*x[4] + x[1]*x[5] + x[2]*x[4] + 5*x[0] + 7*x[4] + 9 * x[6] + 12,
    x[0]*x[2] + x[1]*x[5] + 4*x[2]*x[5] + 2*x[3]*x[4] + 3*x[3]*x[6] + 8*x[1] + 6*x[5] + 20,
    x[0] * x[1] + x[2] * x[4] + 5 * x[3] * x[6] + 21
] # 构造 m 个方程
# for i, p in enumerate(F_polys):
#     print(f"F{i + 1} = {p}")

# 生成 T
while True:
    T = random_matrix(F, n, n)
    if T.is_invertible():
        break

# 生成公钥 P = F(T(s))
subs_dict = {}
for i in range(n):
    subs_dict[x[i]] = sum(T[i, j] * s[j] for j in range(n))
# 到此T完成转化
P_polys = [F_polys[i].subs(subs_dict) for i in range(o)]
print("公钥已生成")
for i, p in enumerate(P_polys):
    print(f"P{i + 1} = {p}")

# 签名生成
message_hash = [12, 25, 27]
print(f"消息哈希 t = {message_hash}")

# 拒绝采样
while True:
    # 随机选择醋变量的值
    y_vals = [F.random_element() for _ in range(v)]
    # 代入醋变量的值, F 退化为线性方程组
    F_y = [F_polys[i].subs({x[j]: y_vals[j] for j in range(v)}) for i in range(o)]
    A = matrix(F, o, o)
    b = vector(F, o)
    for i in range(o):
        # 常数项移到右边
        b[i] = F(message_hash[i]) - F_y[i].subs({x[v + j]: 0 for j in range(o)})
        # 构造系数矩阵
        for j in range(o):
            A[i, j] = F_y[i].monomial_coefficient(x[v + j])
    # A 能够求解
    if A.is_invertible():
        x_oil = A.solve_right(b)
        break

x_secret = vector(F, y_vals + list(x_oil))
print(f"秘密完整解: {x_secret}")
s_sig = T.inverse() * x_secret
print(f"最终公开签名: {s_sig}")
print("至此数字签名完成")

# 验证环节
verify_res = [P_polys[i].subs({s[j]: s_sig[j] for j in range(n)}) for i in range(o)]
if verify_res == [F(val) for val in message_hash]:
    print("验证通过!")
else:
    print("验证失败!!")
print("end")
```
