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
