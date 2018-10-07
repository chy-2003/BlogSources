# CodeForces 538F A Heap of Heaps

<center>树状数组</center>

## 题意
给定一个长度为n的数组A，将它变为一颗k叉树（1 <= k <= n -  1)（堆的形式编号）。
问对于每一个k，有多少个节点小于它的父节点。

----------

## 解题

显然，最初的想法是暴力。因为树的层数到后来好像挺小的。但是仔细看复杂度，好像稳稳的O(n^2)。
那就尴尬了。考虑用数据结构优化。

看其他dalao切此题，用的都是主席树、函数式线段树之类的，但juruo我都没学过怎么办qwq
于是只能好生考虑一番.
我们发现：1、一个节点的儿子在原序列中一定是连续的；2、能对某个节点造成贡献的儿子节点的值一定小于该节点。
于是我们就有如下做法：把节点按值排序，从小到大枚举每个节点。对于k叉数，它儿子中被处理过的节点数，就是这个点对答案中k叉树情况的贡献。
由于2，我们不难得到该做法的正确性。我们再来考虑这个做法的时间复杂度。对于每个点，我们仅考虑它不是树叶的情况。所以实际上对于每个k，我们只考虑了n/k个点。这是O(nlogn)的。然后查询由于结论1，我们可以由树状数组维护，复杂度O(logn)。所以总复杂度O(nlogn^2)。并不会超时。

----------
## 代码实现
我们将编号从0开始，这样更方便。
我们每次处理好一个节点后就在它原数组的位置上标上1，这样就可一直接用树状数组求和。

```C++
#include<bits/stdc++.h>
using namespace std;
struct Node{
	int x, id;
	Node(int x_ = 0, int id_ = 0) {x = x_; id = id_;}
};
const int MAXN = 200010;
int n, x, ans[MAXN], tree[MAXN];
Node a[MAXN];

bool cmp(Node x, Node y){
	return x.x < y.x || x.x == y.x && x.id < y.id;
}

int lowbit(int x){ return x & (-x); }

int ask(int x){
	int t = 0;
	while(x){
		t += tree[x];
		x -= lowbit(x);
	}
	return t;
}

void add(int x){
	if(x == 0) return;
	while(x < n){
		tree[x]++;
		x += lowbit(x);
	}
	return;
}

int main(){
	scanf("%d", &n);
	for(int i = 0; i < n; i++){
		scanf("%d", &x);
		a[i] = Node(x, i);
	}
	sort(a, a + n, cmp);
	memset(ans, 0, sizeof(ans));
	for(int i = 0; i < n; i++){
		for(int k = 1; k * a[i].id + 1 < n && k < n; k++){
			ans[k] += ask(min(k * a[i].id + k, n - 1)) - ask(k * a[i].id);
		}
		add(a[i].id);
	}
	for(int i = 1; i < n; i++) printf("%d ", ans[i]); printf("\n");
	return 0;
}
```