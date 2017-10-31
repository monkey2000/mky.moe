---
layout: post
title: CF 贪心题泛做 1
date: 2017-10-30 12:45:00
categories: OI
tags: [贪心]
---

按照题目难度降序。

{{ .TableOfContents }}

## CF576B. Invariance of Tree

### 题面

> 给定一个排列 {{< tex >}} P {{< /tex >}} ，定义一棵树是符合要求的，当且仅当
> {{< tex >}} \forall (u, v) \in T, (P_u, P_v) \in T {{< /tex >}}
> 
> 构造一个合法的树。
> 
> 树的大小 <= 1e5

### 做法

没想出来。

考虑这个这个排列上的一阶不动点 {{< tex >}} x = P_x {{< /tex >}} 和二阶不动点。

1. 如果排列上存在一个一阶不动点，直接把所有点向它连边
2. 如果排列上存在一个二阶不动点 {{< tex >}} a = P_b = P_{P_a} {{< /tex >}}，对这个排列上其他
的环奇偶染色分别连给 {{< tex >}} a {{< /tex >}} 和 {{< tex >}} b {{< /tex >}}，没法奇偶染色
说明无解。
3. 不满足上述情况即无解

### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int SIZ = 2e5 + 2007;

int N;
int P[SIZ];
int vis[SIZ];
int cnt[2], divi[2][SIZ];

int main() {
    cin >> N;
    
    for(int i=1;i<=N;i++)
        cin >> P[i];

    int idx1 = 0, idx2 = 0;

    for(int i=1;i<=N;i++) {
        if(!idx1 && P[i] == i)
            idx1 = i;
        if(!idx2 && P[P[i]] == i)
            idx2 = i;
    }

    if(idx1) {
        puts("YES");
        for(int i=1;i<=N;i++)
            if(i != idx1)
                cout << i << " " << idx1 << endl;
        return 0;
    } else if(idx2) {
        bool flag = true;
        vis[idx2] = vis[P[idx2]] = 1;
        for(int i=1;i<=N;i++)
            if(!vis[i]) {
                vis[i] = 1;
                divi[0][cnt[0]++] = i;
                int cp = P[i], sel = 1;
                while(cp != i) {
                    vis[cp] = 1;
                    divi[sel][cnt[sel]++] = cp;
                    sel ^= 1;
                    cp = P[cp];
                }

                if(sel == 1) { flag = false; break; }
            }

        if(flag) {
            puts("YES");
            cout << idx2 << " " << P[idx2] << endl;

            for(int i=0;i<cnt[0];i++)
                cout << idx2 << " " << divi[0][i] << endl;

            for(int i=0;i<cnt[1];i++)
                cout << P[idx2] << " " << divi[1][i] << endl;

        } else puts("NO");
        return 0;
    }

    puts("NO");
}
```

