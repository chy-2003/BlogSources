# Codeforces 980D Perfect Groups

<center>数学_数论</center>

## 题目大意

设有一个数组A和对A的一个操作f。那么f(A)返回一个最小的k，使得A能分成k组（每组元素不一定连续，但每个元素一定属于某一组），且每组里的任意两个元素的乘积是完全平方数。
现有一个长度为n的数组A，问有多少中情况，从A中取出一段连续的子序列B，使f(B)=k（k = 1...n）
## 解题思路

首先考虑操作f中的某一组中的某两个数a和b。因为$$ab = k^2(k \in Z)$$所以我们有$$a = \alpha^2t_1$$和$$b=\beta^2t_2$$（α、β为整数且t1、t2不含平方因子）
由质因数分解易推出t1 = t2
所以那一组中的所有数去掉平方因子后都相等
又因为分的组数要最小，所以任意两个数若去掉平方因子后相等，那么他们一定在同一组中。

好了，在回头看如何实现题目要求。
首先第一步一定是去掉每个数的平方因子，记在a数组。之后，我们有一个巧妙的预处理。我们用f[i]保存在i前面第一个a[i] = a[f[i]]的位置。那么在一个区间中若a[i]不为0且f[i]小于这个区间的左端点，就一定不能与它之前的数并为一组，否则一定可以。然后看如何求答案。若我们对于每个k，都枚举区间左端点，再进行推进，那么效率就有些低了，有可能会TLE。所以我们换个角度思考。如果枚举左端点，然后一边按顺序扫左端点后面的点，一边记录到最后答案里，那么时间复杂度就只有O(n^2)。问题解决。

## 程序

```C++
#include<bits/stdc++.h>
using namespace std;
map<int, int> ct;//map能更快地完成初始化，也可以用O(n^2)的时间暴力初始化
int n, a[10010], f[10010], ans[10010], cnt;//a、f的意义如上面所说，ans保存最终答案
int l = 0, b[10010], p[10010];//p保存素数，b为临时数组
int main(){
	memset(b, 0, sizeof(b));
	memset(ans, 0, sizeof(ans));
	memset(p, 0, sizeof(p));
	memset(f, 0, sizeof(f));
	memset(a, 0, sizeof(a));
	n = 0;
	cnt = 0;
	l = 0;
	for(int i = 2; i < 10010; i++){
		if(b[i] == 0){
			l++; p[l] = i;
		} else continue;
		for(int j = i + i; j < 10010; j += i)
			if(b[j] == 0) b[j] = 1;
	}//以上完成素数筛选
	scanf("%d", &n);
	for(int i = 1; i <= n; i++) scanf("%d", &a[i]);
	for(int i = 1; i <= n; i++)
		for(int j = 1; (j <= l) && ((p[j] * p[j]) <= abs(a[i])); j++){//小心有负数
			while(a[i] != 0 && a[i] % (p[j] * p[j]) == 0) a[i] /= (p[j] * p[j]);
			if(a[i] == 0 || a[i] == 1 || a[i] == -1) break;//已经结束，不如退出
		}
		//以上完成去除所有数的平方因子
	for(int i = 1; i <= n; i++){
		f[i] = ct[a[i]];
		ct[a[i]] = i;
	}//完成初始化
	for(int i = 1; i <= n; i++){
		cnt = 0;
		for(int j = i; j <= n; j++){
			if(a[j] && f[j] < i) cnt++; //注意0的情况
			ans[max(1, cnt)]++;
		}
	}//统计答案
	for(int i = 1; i <= n; i++) 
		if(i == 1) printf("%d", ans[i]); else printf(" %d", ans[i]);
	printf("\n");
	return 0;
}
```