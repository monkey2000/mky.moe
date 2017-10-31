---
layout: post
title: CF 贪心题泛做 1
date: 2017-10-30 12:45:00
categories: OI
tags: [贪心]
toc: true
---

按照题目难度降序。

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
说明无解
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

## CF623B. Array GCD

### 题面

> 给一个序列，可以做两种操作
> 
> 1. 最多删掉序列上一段，设这段长度为 `m` ，花费 `m * a` 的代价
> 2. 对于每个位置，加减 1 至多一次，花费 `b`
> 
> 求一个最小花费使得非空新序列的 `gcd` 不为 1

### 做法

考虑序列的第一个元素 `x` 和最后一个元素 `y`，至少有一个没被删掉，所以说答案的 `gcd` 一定是 `{x-1, x, x+1, y-1, y, y+1}` 中某个数的质因子的倍数。

枚举这个质因子，然后在序列上搞一个 `DP` ，计算出来此时的最小开销。

{{< tex display >}} dp[x][y], \qquad x \in [1, N], y \in [0, 2] {{< /tex >}}

表示当前做到第 `i` 个 ， 当前这个 没开始删 (y = 0) ， 删除 (y = 1) ，已经删完 (y = 2) 的最小开销。

复杂度 {{< tex >}} O(N \log( \mathrm{Maxnum} )) {{< /tex >}}

### 代码

第一次写的时候质因数分解忘记特判素数了。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int SIZ = 1e6 + 2007;
const LL llinf = (numeric_limits<LL>::max()>>1) -
        numeric_limits<int>::max();

int N, A, B;
int S[SIZ];

LL ans;

int isP[SIZ], Pri[SIZ], ppos;
void sieve() {
    fill(isP, isP + SIZ, 1);
    isP[0] = isP[1] = 0;
    for(int i=2;i<SIZ;i++) {
        if(isP[i]) {
            Pri[ppos++] = i;
        }

        for(int j=0;j<ppos && i*Pri[j] < SIZ;j++) {
            Pri[i * Pri[j]] = 0;
            if(i % Pri[j] == 0) break;
        }
    }
}

vector<int> factorize(int x) {
    vector<int> ret; int pos = 0;
    while(x != 1 && pos < ppos) {
        if(x % Pri[pos] == 0) {
            ret.push_back(Pri[pos]);

            while(x % Pri[pos] == 0) x /= Pri[pos];
        }

        pos++;
    }

	// !!!!!!!!
    if(x != 1)
        ret.push_back(x);
    // !!!!!!!!

    return ret;
}

int mrk[SIZ];
LL dp[SIZ][3];
void update(int ft) {
    if(ft == 1) return;

    // init
    fill(mrk, mrk + SIZ, 0);
    memset(dp, 0, sizeof(dp));
    for(int i=1;i<=N;i++) {
        if(S[i] % ft != 0) {
            mrk[i] = 1;
            if(((S[i] - 1) % ft != 0) && ((S[i] + 1) % ft != 0)) {
                mrk[i] = 2;
            }
        }
    }

    if(mrk[1] == 1) dp[1][0] = B;
    else if(mrk[1] == 2) dp[1][0] = llinf;

    dp[1][1] = A, dp[1][2] = A;

    for(int i=2;i<=N;i++) {
        if(mrk[i] != 2)
            dp[i][0] = min(llinf, dp[i-1][0] + 1LL * mrk[i] * B);
        else dp[i][0] = llinf;

        dp[i][1] = min(dp[i-1][0] + A, dp[i-1][1] + A);
        
        if(mrk[i] != 2)
	    dp[i][2] = min(min(llinf, dp[i-1][1] + 1LL * mrk[i] * B),
	        dp[i-1][2] + 1LL * mrk[i] * B);
        else dp[i][2] = llinf;
    }

    ans = min(ans, min(dp[N][0], min(dp[N][1], dp[N][2])));
}

int main() {
    sieve();

    scanf("%d %d %d", &N, &A, &B);

    for(int i=1;i<=N;i++)
        scanf("%d", &S[i]);

    ans = llinf;

    for(int ft : factorize(S[1])) update(ft);
    for(int ft : factorize(S[1] - 1)) update(ft);
    for(int ft : factorize(S[1] + 1)) update(ft);
    for(int ft : factorize(S[N])) update(ft);
    for(int ft : factorize(S[N] - 1)) update(ft);
    for(int ft : factorize(S[N] + 1)) update(ft);
   
    printf("%lld\n", ans);
}
```

## CF769C. Cycle In Maze

### 题面

> 给一个迷宫，找一个从起点开始的长度恰好为 k 的字典序最小的回路。
> 字典序为 "D, L, R, U" 表示上下左右单词首字母。

### 做法

先跑一个 BFS 求出每个点到起点的最短路距离，我们称一个点可到达仅当它到起点的最短距离不超过剩余能走的步数。

构造答案就是按照字典序枚举下一个能走的位置，然后走过去。

### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int SIZ = 2007;
typedef pair<int, int> pii;

int N, M, K;
char Mat[SIZ][SIZ];

// D L R U
char rep[] = {'D', 'L', 'R', 'U'};
int dx[] = {1, 0, 0,-1};
int dy[] = {0,-1, 1, 0};

bool valid_pos(int x, int y) {
    return 1 <= x && x <= N && 1 <= y && y <= M && Mat[x][y] != '*';
}

int dis[SIZ][SIZ];
void bfs(int ix, int iy) {
    queue<pii> q; q.push(pii(ix, iy));
    dis[ix][iy] = 0;
    while(!q.empty()) {
        int x = q.front().first, y = q.front().second; q.pop();
        for(int i=0;i<4;i++) {
            int cx = x + dx[i], cy = y + dy[i];
            if(valid_pos(cx, cy) && dis[cx][cy] == 0x3f3f3f3f) {
                dis[cx][cy] = dis[x][y] + 1;
                q.push(pii(cx, cy));
            }
        }
    }
}

vector<char> ans;

bool construct(int x, int y) {
    ans.reserve(K);

    for(int i=0;i<K;i++) {
        int nx = -1, ny = -1;
        for(int j=0;j<4;j++) {
            int cx = x + dx[j], cy = y + dy[j];
            if(valid_pos(cx, cy) && dis[cx][cy] < (K - i)) {
                nx = cx, ny = cy;
                ans.push_back(rep[j]);
                break;
            }
        }

        if(nx == -1 && ny == -1) return false;
        else x = nx, y = ny;
    }

    return true;
}

int main() {
    scanf("%d %d %d", &N, &M, &K);

    for(int i=1;i<=N;i++)
        scanf("%s", Mat[i] + 1);

    int ix, iy;
    for(int i=1;i<=N;i++)
        for(int j=1;j<=M;j++)
            if(Mat[i][j] == 'X') { ix = i, iy = j; break; }

    memset(dis, 0x3f, sizeof(dis));
    bfs(ix, iy);

    if(construct(ix, iy) && ans.size() == K) {
        for(char c : ans) putchar(c);
        puts("");
    } else puts("IMPOSSIBLE");
}
```

## CF708C. Centroids

### 题面

> 树上的重心满足删掉它之后最大的一个联通块小于 `N/2`
> 
> 现在给一棵树，求在修改一条边的情况下，有哪些点可以成为树的重心
> 
> 树的大小 <= 4e5

### 做法

考虑树的原重心，如果有一个点可以成为重心的话，那么一定是这个原重心上拽下来了一个儿子，再接到新重心上。

{{< tex >}} O(N) {{< /tex >}} 可过

### 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
const int SIZ = 400000 + 2007;
const int _inf = 0x3f3f3f3f;
typedef pair<int, int> pii;

int N;
vector<int> G[SIZ];

int siz[SIZ], mx[SIZ], center;
void dfs1(int u = 1, int pre = 0) {
    siz[u] = 1;
    
    for(int v : G[u]) {
        if(v != pre) {
            dfs1(v, u);
            mx[u] = max(mx[u], siz[v]);
            siz[u] += siz[v];
        }
    }

    mx[u] = max(mx[u], N - siz[u]);

    if(mx[u] < mx[center]) {
//      assert(mx[u] != mx[center]);
        center = u;
    }
}

vector<pii> ch;
int ans[SIZ];

void dfs2(int u, int pre, int sel, int cnt) {
    if(cnt <= (N>>1)) ans[u] = 1;
    
    int k = 0;
    for(pii p : ch) {
        k++; if(k > 2) break;
        if(p.second == sel) continue;
        if(N - siz[u] - p.first <= (N>>1)) ans[u] = 1;
    }

    for(int v : G[u]) {
        if(v != pre) dfs2(v, u, sel, cnt);
    }
}

int main() {
    cin >> N;

    int u, v;
    for(int i=1;i<N;i++) {
        cin >> u >> v;
        G[u].push_back(v);
        G[v].push_back(u);
    }

    mx[center] = _inf;
    dfs1();
    assert(center != 0);
    dfs1(center);

    ans[center] = 1;

    for(int v : G[center]) {
        ch.push_back(pii(siz[v], v));
    }

    sort(ch.begin(), ch.end(), greater<pii>());

    for(int v : G[center]) {
        dfs2(v, center, v, N - siz[v]);
    }

    for(int i=1;i<=N;i++) printf("%d ", ans[i]);
    puts("");
}
```

## CF870B. Maximum of Maximums of Minimums

能分三段以上的时候一定可以把最大值单独分在一段。

其他时候暴力。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int SIZ = 4e5 + 2007;

int N, K;
int A[SIZ], Pre[SIZ], Pos[SIZ];

int main() {
    cin >> N >> K;
    for(int i=1;i<=N;i++) cin >> A[i];
    Pre[0] = Pos[N+1] = numeric_limits<int>::max();
    for(int i=1;i<=N;i++) Pre[i] = min(Pre[i-1], A[i]);
    for(int i=N;i>=1;i--) Pos[i] = min(Pos[i+1], A[i]);

    int ans = numeric_limits<int>::min();
    if(K == 1) ans = Pos[1];
    else if(K == 2) {
        for(int i=1;i<N;i++) {
            ans = max(ans, max(Pre[i], Pos[i+1]));
        }
    } else
        ans = *max_element(A+1, A+N+1);

    printf("%d\n", ans);
}
```

## CF870C. Maximum splitting

注意到样例答案中总是有 4 ，而 4 也是最小的质完全平方数，所以往这个方向想。

1. 如果模 4 余 0，可以分解成 n * 4
2. 如果模 4 余 1，可以分解成 n * 4 + 9
3. 如果模 4 余 2，可以分解成 n * 4 + 6
4. 如果模 4 余 3，可以分解成 n * 4 + 6 + 9

```cpp
#include <bits/stdc++.h>
using namespace std;

int Q;
int ans[61];

int main() {
    cin >> Q;
    for(int i=1;i<=60;i++) ans[i] = -1;
    for(int i=4;i<=60;i+=4) ans[i] = i / 4;
    for(int i=9;i<=60;i+=4) ans[i] = ((i - 9) / 4) + 1;
    for(int i=6;i<=60;i+=4) ans[i] = ((i - 6) / 4) + 1;
    for(int i=15;i<=60;i+=4) ans[i] = ((i - 15) / 4) + 2;

    int x;
    while(Q--) {
        cin >> x;
        int a = (x / 20) * 5;
        int b = x % 20;
        int k = min(3, a);
        a -= k, b += 4 * k;
        cout << a + ans[b] << endl;
    }
}
```

## CF873C. Strange Game On Matrix

每列分开做，滑动窗口维护。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int SIZ = 207;
typedef pair<int, int> pii;

int N, M, K;
int Mat[SIZ][SIZ];

pii solve(int *p) {
    int tot = 0;
    for(int i=1;i<=K;i++) tot += p[i];
    int rem = 0, ansrem = 0, anscnt = 0;
    for(int i=1;i<=N;i++) {
        if(p[i] && tot > anscnt) {
            ansrem = rem, anscnt = tot;
        }

        tot -= p[i];
        rem += p[i];

        tot += p[i+K];
    }

    return pii(anscnt, ansrem);
}

int main() {
    cin >> N >> M >> K;

    for(int i=1;i<=N;i++) {
        for(int j=1;j<=M;j++) {
            cin >> Mat[j][i];
        }
    }

    pii ans(0, 0);

    for(int i=1;i<=M;i++) {
        pii cur = solve(Mat[i]);
        ans.first += cur.first;
        ans.second += cur.second;
    }

    cout << ans.first << " "<< ans.second << endl;
}
```

## CF864C. Bus

模拟即为最优情况，当时似乎写了个假二分答案增加常数（大雾）。

```cpp
#include <iostream>
#include <cstdio>
#include <cstdlib>
#include <algorithm>
using namespace std;
typedef long long LL;

int A, B, F, K;

bool check(int tot) {
    //printf("Check %d\n", tot);
    LL pos = 0, end = 1LL * A * K;

    // P_1 = 2m * K + F
    // P_2 = 2(m+1) * K - F
    // m \in N

    while(pos < end && tot > 0) {
        if(pos + B >= end) { pos += B; break; }

        LL m1 = -1, m2 = -1;

        m1 = (pos + B - F) / (A<<1);
        if(F != A) m2 = (pos + B + F - (A<<1)) / (A<<1);

        LL pos1 = m1 >= 0 ? (m1<<1) * A + F : -1;
        LL pos2 = m2 >= 0 ? ((m2+1)<<1) * A - F : -1;

        if(pos1 - pos > B) pos1 = -1;
        if(pos2 - pos > B) pos2 = -1;

        //printf("%d %d %d %d\n", m1, m2, pos1, pos2);

        LL mmpos = max(pos1, pos2);

        //printf("%lld -> %lld\n", pos, mmpos);

        if(mmpos > pos) pos = mmpos;
        else return false;

        --tot;
    }

    return (pos >= end) || (pos + B >= end && tot >= 0);
}

int main() {
    scanf("%d %d %d %d", &A, &B, &F, &K);

    int L = 0, R = K + 2; // [L, R]

    while(L != R) {
        int Mid = (L + R) >> 1;

        bool cur = check(Mid);
        //printf("Check %d = %d\n", Mid, cur);

        if(cur) {
            R = Mid;
        } else {
            L = Mid + 1;
        }
    }

    if(check(L)) {
        printf("%d\n", L);
    } else {
        printf("%d\n", -1);
    }
}
```

## CF863B. Kayaking

暴力

```cpp
#include <bits/stdc++.h>
using namespace std;
const int SIZ = 207;

int N, M;
int W[SIZ];

int main() {
    cin >> N;
    M = N<<1;
    
    for(int i=1;i<=M;i++) {
        cin >> W[i];
    }

    sort(W+1, W+M+1);

    int ans = numeric_limits<int>::max();
    for(int i=1;i<=M;i++) {
        for(int j=i+1;j<=M;j++) {
            int tot = 0;
            for(int k=1;k<=M;) {
                if(k == i || k == j) { k++; continue; }
                int a = k, b = k + 1;
                while(b == i || b == j) b++;
                tot += W[b] - W[a];
                k = b + 1;
            }
            
            ans = min(ans, tot);
        }
    }

    cout << ans << endl;
}
```
