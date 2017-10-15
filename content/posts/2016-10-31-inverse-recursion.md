---
layout: post
title: 数论 - 递推求逆元
date: 2016-10-31 17:49:00
categories: OI
tags: [数论, 逆元]
---

## 递推求逆元的推导

设 {{< tex >}} t = \lfloor M / i \rfloor, k = M \bmod i {{< /tex >}} ，那么有：

{{< tex display >}} it + k \equiv 0 \pmod M {{< /tex >}}

{{< tex display >}} -it \equiv k \pmod M {{< /tex >}}

两边同乘 {{< tex >}} (ik)^{-1} {{< /tex >}}：

{{< tex display >}} -tk^{-1} \equiv i^{-1} \pmod M {{< /tex >}}

带回：

{{< tex display >}} Inv[i] = (M - M/i) * Inv[M \% i] \% M {{< /tex >}}

易知递推边界：

{{< tex display >}} Inv[1] = 1 {{< /tex >}}


## 代码模板

```cpp
const int MOD = 1E9+7;
int Inv[100000];
void getInv(int c) {
    Inv[1] = 1;
    for(int i=2;i<=c;i++) Inv[i] = ((MOD - MOD/i) * Inv[MOD % i]) % MOD;
}
```
