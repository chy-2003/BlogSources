# Codeforces 343D Water Tree　＆　树链剖分教程

<center>数据结构_树链剖分</center>

---

[原题链接](http://codeforces.com/problemset/problem/343/D)

#题目大意
给定一棵根为１，初始时所有节点值为０的树，进行以下三个操作：
>*将以某点为根的子树节点值都变为１
>*将某个节点及其祖先的值都变为０
>*询问某个节点的值

#解题思路
这是一道裸的树链剖分题。下面详细地介绍一下树链剖分。
###树链剖分预备知识：
线段树、DFS序
###树链剖分想法｜起源
首先，如果一棵树退化成一条链，那么它会有非常好的性质。我们可以用线段树等数据结构来维护相关操作，使得效率更高。那么我们考虑一般的树，它是否能被分成一些链，使它们也能更高效地进行某些操作？
###算法流程
以下以点带权的树为例，介绍最为常用的轻重边剖分。
我们令当前点到它儿子中size最大的点的边为重边，其余为轻边。
由此定义，我们可以得到以下结论：
>*若u（父）到v（子）的边为轻边，则size(u) >= 2 * size(v)【1】（结论平凡）
>*从根节点到某叶节点的轻边数不超多log(n)【2】（由上条结论易得）

接下来，我们定义由重边组成的链叫重链。那么重链又有以下几条性质：
>*根节点到某叶节点的重链数不超过log(n)【3】（由于重链的头或尾都是轻边）
>*任何一个节点在且仅在一条重链上【4】

那么我们对树作修改的时候只需要对重链上的点作修改（由性质【4】）。那么我们可以尝试将原树拉成一条链，同时使得重链上的点在拉成后的链上连续。
我们可以用两遍dfs来实现这个操作。第一步，我们求出每个点的父亲信息(father)，深度信息(depth)，子树大小(size)和重儿子(son)(这么叫应该可以)。第二遍求出每个点的所在重链的顶部节点(top)，当前点在线段树上位置(seg)，同时求出线段树上点对应的点的编号(rev)。
代码实现（其中用数组模拟链表保存原树信息）：
```C++
int father[MAXN], dep[MAXN], size[MAXN], son[MAXN], top[MAXN], seg[MAXN], rev[MAXN];
int used_space = 0;

void first_dfs(int fa, int pos){
    father[pos] = fa;
    dep[pos] = dep[fa] + 1;
    size[pos] = 1;
    for(int t = f[pos]; t; t = nex[t]){
        if(lin[t] == fa) continue;
        first_dfs(pos, lin[t]);
        size[pos] += size[lin[t]];
        if(size[lin[t]] > size[son[pos]]) son[pos] = lin[t];//选取size最大的为重儿子
    }
    return;
}



void second_dfs(int fa, int pos){
    if(size[pos] > 1){//为了使重链在线段树上连续，所以先访问重儿子
        ++used_space;
        seg[son[pos]] = used_space;
        top[son[pos]] = top[pos];
        rev[used_space] = son[pos];
        second_dfs(pos, son[pos]);
    }
    for(int t = f[pos]; t; t = nex[t])
        if(lin[t] != fa && lin[t] != son[pos]){
            ++used_space;
            seg[lin[t]] = used_space;
            top[lin[t]] = lin[t];
            rev[used_space] = lin[t];
            second_dfs(pos, lin[t]);
        }
    return;
}

//主程序调用

    first_dfs(1, 1);
    top[1] = 1; seg[1] = 1; rev[1] = 1; used_space++;
    second_dfs(1, 1);
```




接下来考虑查询/修改操作。
第一类是在某两点之间的路径上操作。设这两点为x,y，则此问题实则考虑x->lca(x,y)->y。
若top[x] != top[y]， 那么它们不在同一条链上。此时我们不如假设top[x]的深度较深，那么top[x]一定不为lca(x,y)。那么我们可以让x跳到father[top[x]]，同时在线段树中处理seg[top[x]]->seg[x]这段区间。不断这样地操作，直到top[x]==top[y]。现在，x, y就在同一条链上了。不妨还是设x的深度较深，那么我们只需在线段树中seg[y]->seg[x]这段区间。
代码实现（以最大值询问为例）：
```C++
int ask_maxnum(int x, int y){
    int fx = top[x], fy = top[y], ans = -INF;
    while(fx != fy){
        if(dep[fx] < dep[fy]){
            swap(x, y);
            swap(fx, fy);
        }
        ans = max(ans, segask_maxnum(seg[fx], seg[x], 1, 1, used_space));
        x = father[fx]; fx = top[x];
    }
    if(dep[x] < dep[y]) swap(x, y);
    ans = max(ans, segask_maxnum(seg[y], seg[x], 1, 1, used_space));
    return ans;
}
```

还有一类是对某子树的操作。
我们由dfs序可知，以x为根的子树在线段树上为seg[x]->seg[x]+size[x]-1这一段区间。代码实现较为简单，这里就不详细贴出。
还有单点的修改与查询就不用多说了吧：）

###建议习题
模板题：[洛谷P3384](https://www.luogu.org/problemnew/show/P3384)
#题目CF343D代码:
```C++
#include<bits/stdc++.h>
#define mid ((l + r) >> 1)
using namespace std;
const int MAXN = 500010;
int n, m, x, y, X, Y, tag;

int lp = 0, f[MAXN], lin[MAXN << 1], nex[MAXN << 1];
void add(int x, int y){ lin[++lp] = y; nex[lp] = f[x]; f[x] = lp; return; }

int father[MAXN], deep[MAXN], son[MAXN], size[MAXN], top[MAXN], seg[MAXN], rev[MAXN];

int used = 0, tree[MAXN << 3]; //0 : all empty;  1 : all filled;  2 : unknow;

int get_int(){
    char ch = getchar();
    while(ch < '0' || ch > '9') ch = getchar();
    int t = 0;
    while(ch >= '0' && ch <= '9'){
        t = t * 10 + ch - '0';
        ch = getchar();
    }
    return t;
}



void dfs1(int fa, int pos){
    father[pos] = fa;
    deep[pos] = deep[fa] + 1;
    size[pos] = 1;
    for(int t = f[pos]; t; t = nex[t])
        if(lin[t] != fa){
            dfs1(pos, lin[t]);
            size[pos] += size[lin[t]];
            if(size[lin[t]] > size[son[pos]]) son[pos] = lin[t];
        }
    return;
}


void give_(int x){
    used++;
    seg[x] = used;
    rev[used] = x;
    return;
}


void dfs2(int fa, int pos){
    if(size[pos] > 1){
        top[son[pos]] = top[pos];
        give_(son[pos]);
        dfs2(pos, son[pos]);
    }
    for(int t = f[pos]; t; t = nex[t])
        if(lin[t] != fa && lin[t] != son[pos]){
            top[lin[t]] = lin[t];
            give_(lin[t]);
            dfs2(pos, lin[t]);
        }
    return;
}


void tag_down(int pos, int l, int r){
    if(l == r) return;
    if(tree[pos] == 2) return;
    tree[pos << 1] = tree[pos];
    tree[(pos << 1) + 1] = tree[pos];
    return;
}


void seg_set(int pos, int l, int r){
    if(X <= l && r <= Y){
        tree[pos] = tag;
        return;
    }
    if(tree[pos] != 2) tag_down(pos, l, r);
    if((tree[pos] != 2) && (tag != tree[pos])) tree[pos] = 2;
    if(X <= mid) seg_set(pos << 1, l, mid);
    if(Y > mid) seg_set((pos << 1) + 1, mid + 1, r);
    return;
}


int seg_ask(int pos, int l, int r){
    if(tree[pos] != 2) return tree[pos];
    if(X <= mid) return seg_ask(pos << 1, l, mid);
    if(X > mid) return seg_ask((pos << 1) + 1, mid + 1, r);
    return -1;
}


void fill(int x){
    X = seg[x]; Y = X + size[x] - 1; tag = 1;
    seg_set(1, 1, used);
    return;
}


void empty(int x){
    int fx;
    while(x){
        fx = top[x];
        X = seg[fx]; Y = seg[x]; tag = 0;
        seg_set(1, 1, used);
        x = father[fx];
    }
    return;
}


int ask(int x){
    X = seg[x];
    return seg_ask(1, 1, used);
}


int main(){
    n = get_int();
    for(int i = 1; i < n; i++){
        x = get_int(); y = get_int();
        add(x, y); add(y, x);
    }
    dfs1(0, 1); //get first four values
    give_(1); top[1] = 1;
    dfs2(0, 1);
    m = get_int();
    for(int i = 1; i <= m; i++){
        x = get_int(); y = get_int();
        if(x == 1) fill(y); else
        if(x == 2) empty(y); else
        printf("%d\n", ask(y));
    }
    return 0;
}

```