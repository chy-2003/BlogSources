# Topcoder SRM 674 Div.2题解

<center>杂题, 树形DP</center>

## T1
### 解题思路
这题应该不是很难，主要是题意理解问题。
注意给出的两个数组里映射关系已经对应好了，只要判断是否为双射即可
### 参考程序
```C++
#include <bits/stdc++.h>
using namespace std;

class RelationClassifier {
public:
    string isBijection( vector <int> domain, vector <int> range );
};
int reflect[110], used[110];
string RelationClassifier::isBijection(vector <int> domain, vector <int> range) {
    int n = domain.size();
    memset(reflect, 255, sizeof(reflect));
    memset(used, 255, sizeof(used));
    for(int i = 0; i < n; i++){
        if(reflect[domain[i]] != -1 && reflect[domain[i]] != range[i]) return "Not";
        if(used[range[i]] != -1 && reflect[domain[i]] != range[i]) return "Not";
        reflect[domain[i]] = range[i];
        used[range[i]] = 1;
    }
    return "Bijection";
}
```


## T2
### 解题思路
首先理解题目中的两个操作的意思。
将点都向同一个方向移动相等距离，就是人可以在坐标系里随意走；可以将所有点旋转一个角度，就是人可以任意转动。那么我们只要找出一组互相垂直的直线，使它们穿过尽量多的点。
由于`n`的范围只有`50`，我们考虑最为暴力的做法。
首先枚举两个相异点 `A` 和 `B`，确定一条直线，再取异于`A`和`B`的点`C`，确定另一条垂线，最后扫一遍所有点就好了。
注意这里的特殊情况：`A`和`B`横或纵坐标相等时特判（斜率的做法，同时还要注意精度）；`n`小于等于`3`的时候答案就是`n`。
据说有大佬会此题不用浮点运算的做法，欢迎评论指出。
### 参考程序
```C++
#include <bits/stdc++.h>
using namespace std;

class PlaneGame {
public:
    int bestShot( vector <int> x, vector <int> y );
};
long double k1, b1, k2, b2;
int t, ans;
int PlaneGame::bestShot(vector <int> x, vector <int> y) {
    int n = x.size();
    if(n <= 2) return n;//特判
    ans = 0;
    for(int i = 0; i < n - 1; i++)
       for(int j = i + 1; j < n; j++)
           for(int k = 0; k < n; k++){
               if(i == k || j == k) continue;
               if(x[i] != x[j] && y[i] != y[j]){
                   k1 = (long double)(y[j] - y[i]) / (long double)(x[j] - x[i]);
                   b1 = (long double)y[i] - k1 * (long double)x[i];
                   k2 = -1.0/k1;
                   b2 = (long double)y[k] - k2 * (long double)x[k];//两条直线的信息
                   t = 0;
                   for(int l = 0; l < n; l++)
                       if(abs((long double)y[l] - (k1 * x[l] + b1)) <= 0.0001 || abs((long double)y[l] - (k2 * x[l] + b2)) <= 0.0001) t++;//注意精度误差
                   ans = max(ans, t);
                   continue;
               }
               if(y[i] == y[j]){//特判
                   t = 0;
                   for(int l = 0; l < n; l++)
                       if(y[l] == y[i] || x[l] == x[k]) t++;
                   ans = max(ans, t);
                   continue;
               }
               if(x[i] == x[j]){
                   t = 0;
                   for(int l = 0; l < n; l++)
                       if(x[l] == x[i] || y[l] == y[k]) t++;
                   ans = max(ans, t);
                   continue;
               }
           }
    return ans;
}
```

## T3
### 解题思路
首先我们简化问题。如果祖先关系形成一棵树，那么问题就变成了十分简单的树形DP，相信大家都会做。
那么我们看看这道题的麻烦之处。这里有至多`15`个点有两个父亲。所以再按照原先的做法，有两个父亲的节点的子树就会被计算两次。
如何解决这个问题？最朴素的想法，可能就是切掉和其中一个父亲的关系，使它变为一棵树。但是问题又来了，这样无法维护和那个父亲间的关系（如果那个父亲不选，那么这个点就必须选啊qwq）。所以，这时候`15`这个数就派上用场了。由于它比较小，所以我们直接暴力枚举有两个父亲的节点，规定它们必须选，或者必须不选。如果我们认定这个节点必须不选，那么它的两个父亲就必须选了。
解决了？不，还有一个问题。仔细思考一下，发现删边也不能乱删。我们必须删指向深度较小的父亲的那条边。
至此，问题解决。最坏时间复杂度 $O(2^{15}*100)$ 然而由于十分暴力，最慢的点跑了1.2秒（然而时限是两秒qwq）。如果有大佬有更好的做法，可以在品论区提出。
### 参考程序
程序中通过按最大深度排序及打标记实现删边。
```C++
#include <bits/stdc++.h>
using namespace std;

class VampireTreeDiv2 {
public:
    int countMinSamples( vector <int> A, vector <int> B );
};
const long long mod = 1000000007;
long long n, cnt;
long long lp, f[1010], lin[2010], nxt[2010], deep[1010];
inline void add(long long x, long long y) { lin[++lp] = y; nxt[lp] = f[x]; f[x] = lp; return; }
void get_deep(long long pos, long long father){
    deep[pos] = deep[father] + 1;
    for(long long t = f[pos]; t; t = nxt[t])
        if(deep[lin[t]] < deep[pos] + 1 && lin[t] != father) get_deep(lin[t], pos);
    return;
}
struct Node{
    long long deep, pos;
    Node(long long deep_ = 0, long long pos_ = 0) { deep = deep_; pos = pos_; return; }
};
Node aa[1010];
bool cmp(Node x, Node y){
    return x.deep > y.deep;
}
long long rec[1010], recc[1010];
long long caculated[1010];//标记
long long dp[1010][2], c[1010][2];//dp[i][0]表示i不选，dp[i][1]表示i选，c数组维护方案数
long long t;
long long ans, minnum;
int VampireTreeDiv2::countMinSamples(vector <int> A, vector <int> B) {
    n = A.size();
    cnt = 0;
    memset(f, 0, sizeof(f));
    lp = 0;
    for(long long i = 0; i < n; i++){
        add(A[i], i + 1);
        if(B[i] != -1){
            add(B[i], i + 1);
            rec[++cnt] = i + 1;
        }
    }
    ans = 0; minnum = 100010;
    memset(deep, 0, sizeof(deep));
    get_deep(0, 0);
    for(long long i = 0; i <= n; i++) aa[i] = Node(deep[i], i);
    sort(aa, aa + n + 1, cmp);//按最大深度排序
    for(long long i = 0; i < (1 << cnt); i++){
        memset(recc, 0, sizeof(recc));
        memset(caculated, 0, sizeof(caculated));
        for(long long j = 1; j <= cnt; j++)
            if((i >> (j - 1)) & 1)
                recc[rec[j]] = 1; else recc[rec[j]] = 2;
        for(long long j = 0; j <= n; j++){
            t = aa[j].pos;
            dp[t][0] = 0; dp[t][1] = 1;
            c[t][0] = 1; c[t][1] = 1;
            for(long long k = f[t]; k; k = nxt[k]){
                if(caculated[lin[k]]){
                    if(dp[lin[k]][1] > 100000){
                        dp[t][0] = 100010;
                        c[t][0] = 0;
                    }
                    continue;
                }
                if(recc[lin[k]] != 0) caculated[lin[k]] = 1;
                dp[t][0] += dp[lin[k]][1];
                c[t][0] = (c[t][0] * c[lin[k]][1]) % mod;
                if(dp[lin[k]][0] < dp[lin[k]][1]){
                    dp[t][1] += dp[lin[k]][0];
                    c[t][1] = (c[t][1] * c[lin[k]][0]) % mod;
                }
                if(dp[lin[k]][0] > dp[lin[k]][1]){
                    dp[t][1] += dp[lin[k]][1];
                    c[t][1] = (c[t][1] * c[lin[k]][1]) % mod;
                }
                if(dp[lin[k]][0] == dp[lin[k]][1]){
                    dp[t][1] += dp[lin[k]][1];
                    c[t][1] = (c[t][1] * ((c[lin[k]][0] + c[lin[k]][1]) % mod)) % mod;
                }
            }
            if(recc[t] == 1) dp[t][0] = 100010, c[t][0] = 0;
            if(recc[t] == 2) dp[t][1] = 100010, c[t][1] = 0;
        }
        if(dp[0][0] < minnum){
            minnum = dp[0][0];
            ans = c[0][0];
        } else
            if(dp[0][0] == minnum) ans = (ans + c[0][0]) % mod;
        if(dp[0][1] < minnum){
            minnum = dp[0][1];
            ans = c[0][1];
        } else
            if(dp[0][1] == minnum) ans = (ans + c[0][1]) % mod;
    }
    return ans;
}
```