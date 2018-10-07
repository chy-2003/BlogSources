# ZOJ3649 Social Net

<center>倍增, 最小生成树</center>

题目链接：http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemCode=3649

---

这题倍增维护信息之多，也能算是一道毒瘤题了……
## 解题思路
这题分为两个部分，第一个是最大生成树，第二个是若干个询问，问在生成树上 $ x->y $ 的路径上最大的 $ ck-cj (ck >= cj, j <= k) $ 。
第一个部分我想就不用多讲，直接跑最小生成树（只是把最小换成了最大）。
第二个部分，重点在于$(ck>=cj,j<=k)$这个限制。由于题目特性，看起来是逃不过倍增的了。首先，我们自然地将 $ x->y $ 分成了 $ x->lca(x,y)->y $ 。通过这个分段，我们观察我们倍增时需要维护那些东西。仔细分析发现一共有如下几样：

-  $ d[x][i].d $ ：维护 $ x $ 的第 $ 2^i $ 个祖先
-  $ d[x][i].maxnum $ ：维护从 $ x $ 到　$ x $ 的第 $ 2^i $ 个祖先中的最大值
-  $ d[x][i].minnum $ ：维护从 $ x $ 到　$ x $ 的第 $ 2^i $ 个祖先中的最小值
-  $ d[x][i].max\_up $ ：维护从 $ x $ 到　$ x $ 的第 $ 2^i $ 个祖先中的最大差值（由下至上)
-  $ d[x][i].max\_dn $ ：维护从 $ x $ 到　$ x $ 的第 $ 2^i $ 个祖先中的最大差值（由上至下)

对于后两条的解释：从公式中可以看出，相减具有方向性，而且 $ x->lca(x,y) $ 与 $ lca(x,y)->y $ 方向不同。
另行说明：本文 $ d[x][i] $ 所含信息不包括 $ x $ （这样可以很方便地分成两个不相交的区间）（或许可以不这么做qwq）

然后我们考虑如何维护这五个值。前三个没有什么大问题，后两个要注意整合时要考虑三个值，千万不要漏了两个区间并中的值（最大值与最小值相减）。以下给出维护这五个值的递推式:

 - $ d[x][i].d = d[d[x][i - 1].d][i - 1].d $ 
 - $ d[x][i].maxnum = max(d[x][i - 1].maxnum, d[d[x][i - 1].d][i - 1].maxnum) $
 - $ d[x][i].minnum = min(d[x][i - 1].minnum, d[d[x][i - 1].d][i - 1].minnum) $
 - $ d[x][i].max\_up = max(d[x][i - 1].max\_up, d[d[x][i - 1].d][i - 1].max\_up, d[d[pos][i - 1].d][i - 1].maxnum - d[pos][i - 1].minnum) $
 - $ d[pos][i].max\_up = max(d[pos][i - 1].max\_up, d[d[pos][i - 1].d][i - 1].max\_up, d[pos][i - 1].maxnum - d[d[pos][i - 1].d][i - 1].minnum) $

我们可以以类似的方法合并 $ x->lca(x,y) $ 和 $ lca(x,y)->y $ 两段区间，得到最后的答案。

---

## 参考程序
tip：程序中 $ x -> lca(x,y) -> y $ 是以 $ y -> lca(x,y) -> x $ 的顺序做的。
```C++
#include <cstdio>
#include <iostream>
#include <cmath>
#include <sstream>
#include <cstring>
#include <algorithm>
using namespace std;
const int MAXN = 30010, MAXM = 50010;
const int INF = 1000000000;
int n, m, b[MAXN];
struct Edge {
	int x, y, z;
	Edge(int x_ = 0, int y_ = 0, int z_ = 0) { x = x_; y = y_; z = z_; return; }
};
Edge edge[MAXM];
int x, y, z;
bool cmp(Edge x, Edge y) {
	return x.z > y.z;
}
int father[MAXN];
int lp, f[MAXN], lin[MAXN << 1], nxt[MAXN << 1];
inline void add(int x, int y) { lin[++lp] = y; nxt[lp] = f[x]; f[x] = lp; return; }
int ans;
int get_father(int x) {
	if(father[x] == x) return x;
	father[x] = get_father(father[x]);
	return father[x];
}
int q;

struct Node {
	int d, maxnum, minnum, max_dn, max_up;
};
Node d[MAXN][16];
int deep[MAXN];
void build_tree(int pos, int fa) {//建树，同事处理完成倍增相关的信息
	deep[pos] = deep[fa] + 1;
	d[pos][0].d = fa;
	d[pos][0].maxnum = b[fa];
	d[pos][0].minnum = b[fa];
	d[pos][0].max_dn = -INF;
	d[pos][0].max_up = -INF;
	for(int i = 1; i < 16; i++) {
		d[pos][i].d = d[d[pos][i - 1].d][i - 1].d;
		d[pos][i].maxnum = max(d[pos][i - 1].maxnum, d[d[pos][i - 1].d][i - 1].maxnum);
		d[pos][i].minnum = min(d[pos][i - 1].minnum, d[d[pos][i - 1].d][i - 1].minnum);
		d[pos][i].max_dn = max(d[pos][i - 1].max_dn, d[d[pos][i - 1].d][i - 1].max_dn);
		d[pos][i].max_dn = max(d[pos][i].max_dn, d[d[pos][i - 1].d][i - 1].maxnum - d[pos][i - 1].minnum);
		d[pos][i].max_up = max(d[pos][i - 1].max_up, d[d[pos][i - 1].d][i - 1].max_up);
		d[pos][i].max_up = max(d[pos][i].max_up, d[pos][i - 1].maxnum - d[d[pos][i - 1].d][i - 1].minnum);
	}
	for(int t = f[pos]; t; t = nxt[t]) {
		if(lin[t] == fa) continue;
		build_tree(lin[t], pos);
	}
	return;
}
int get_lca(int x, int y) {//求最近公共祖先
	if(deep[x] < deep[y]) swap(x, y);
	for(int i = 15; i >= 0; i--)
		if(deep[d[x][i].d] >= deep[y]) x = d[x][i].d;
	if(x == y) return x;
	for(int i = 15; i >= 0; i--)
		if(d[x][i].d != d[y][i].d) x = d[x][i].d, y = d[y][i].d;
	return d[x][0].d;
}
int recmin, recmax;
void go_up(int x, int y){//y->lca(x, y)
	if(x == y) return;//由于倍增中设定是不包含ｘ的
	recmax = max(recmax, b[x]);
	for(int i = 15; i >= 0; i--) {
		if(deep[d[x][i].d] <= deep[y]) continue;
		ans = max(ans, recmax - d[x][i].minnum);
		recmax = max(recmax, d[x][i].maxnum);
		ans = max(ans, d[x][i].max_up);
		x = d[x][i].d;
	}
	return;
}
void go_down(int x, int y){//lca(x, y)->x
	recmin = min(recmin, b[x]);
	for(int i = 15; i >= 0; i--) {
		if(deep[d[x][i].d] < deep[y]) continue;
		ans = max(ans, d[x][i].maxnum - recmin);
		recmin = min(recmin, d[x][i].minnum);
		ans = max(ans, d[x][i].max_dn);
		x = d[x][i].d;
	}
	return;
}
int main() {
	while(scanf("%d", &n) == 1) {//多组数据
		memset(b, 0, sizeof(b));
		lp = 0;
		memset(f, 0, sizeof(f)); memset(lin, 0, sizeof(lin)); memset(nxt, 0, sizeof(nxt));
		memset(edge, 0, sizeof(edge));
		memset(deep, 0, sizeof(deep));
		memset(d, 0, sizeof(d));//初始化
		
		for(int i = 1; i <= n; i++) father[i] = i;
		for(int i = 1; i <= n; i++) scanf("%d", &b[i]);
		scanf("%d", &m);
		for(int i = 1; i <= m; i++) {
			scanf("%d%d%d", &x, &y, &z);
			edge[i] = Edge(x, y, z);
		}
		sort(edge + 1, edge + m + 1, cmp);
		z = 1;
		ans = 0;
		for(int i = 1; i < n; i++) {
			x = get_father(edge[z].x);
			y = get_father(edge[z].y);
			while(x == y) {
				x = get_father(edge[++z].x);
				y = get_father(edge[z].y);
			}
			ans += edge[z].z;
			father[y] = x;
			add(edge[z].x, edge[z].y);
			add(edge[z].y, edge[z].x);//加入树边
			z++;
		}
		printf("%d\n", ans);//最大生成树（库鲁斯卡尔）

		build_tree(1, 1);//建树，同事处理完成倍增相关的信息
		scanf("%d", &q);
		for(int i = 1; i <= q; i++) {
			scanf("%d%d", &x, &y);
			int lca = get_lca(x, y);//求最近公共祖先
			ans = 0; recmin = INF; recmax = -INF;
			go_up(y, lca);//y->lca(x, y)
			go_down(x, lca);//lca(x, y)->x
			ans = max(ans, recmax - recmin);//合并答案
			printf("%d\n", ans);
		}
	}
	return 0;
}

```