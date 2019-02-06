# TopCoder SRM 667 Div.2题解

<center>DP, 杂题, DP_状压DP</center>

---
概览：
T1 枚举 T2 状压DP T3 DP

------
# TopCoder SRM 667 Div.2 T1

## 解题思路
由于数据范围很小，所以直接枚举所有点，判断是否可行。时间复杂度O(δX × δY)，空间复杂度O(1)。

## 参考程序段

```C++
#include <bits/stdc++.h>
using namespace std;

class PointDistance {
public:
    vector <int> findPoint( int x1, int y1, int x2, int y2 );
};
int sqr(int x){ return x * x; }
vector<int> ans;
vector <int> PointDistance::findPoint(int x1, int y1, int x2, int y2) {
	ans.clear();
	for(int x = -100; x <= 100; x++)
		for(int y = -100; y <= 100; y++)
			if(sqr(x1 - x) + sqr(y1 - y) > sqr(x2 - x) + sqr(y2 - y)){
				ans.push_back(x);
				ans.push_back(y);
				return ans;
			}
	return ans;
}


```

------

# TopCoder SRM 667 Div.2 T2
## 解题思路
可能大家的第一反应都是每一步选花费最小来贪心，但是发现，某一步有多个花费最小时，会出问题。
我们看一组数据
0000
1100
0011
0111
如果贪心， 可能顺序是1,2,3,4， 这样答案为8， 而事实上按顺序1,3,4,2,最优，为6。
于是我们考虑DP。
首先，我们令dp[i]表示在i的二进制表示下为1的那些指令处理完毕所需的最小花费。则可以从任何二进制表示下比i少一个1的j转移过来，我们取其最小。开始状态为dp[0] = 0，结束状态为dp[(1 << n) - 1]。
## 参考程序段
程序中，并非枚举dp[j]向dp[i]转移，而是对于dp[j]向每个可能的dp[i]转移，并使用了类似于spfa的写法。
同时，程序中q数组保存的数指向dp数组的下标，q_used对应花费。

```C++
#include <bits/stdc++.h>
using namespace std;

class OrderOfOperationsDiv2 {
public:
    int minTime( vector <string> s );
};
int n;
int rec[20];
int l, r, q[1 << 23], q_used[1 << 23];
int dp[1 << 21];
int len;
int OrderOfOperationsDiv2::minTime(vector <string> s) {  
	n = s.size();
	len = s[0].size();
	for(int i = 0; i < n; i++){
		rec[i] = 0;
		for(int j = 0; j < len; j++)
			if(s[i][j] == '1') rec[i] |= (1 << j);//转成二进制方便处理
	}
	memset(dp, 255, sizeof(dp));
	dp[0] = 0;
	l = 0; r = 1; q[1] = 0; q_used[1] = 0;
	while(l < r){
		l++;
		int rec_vis = q[l];
		int rec_used = q_used[l];
		for(int i = 0; i < n; i++){//枚举可能被更新的状态
			if((rec_vis >> i) & 1) continue;
			int x = rec_vis | (1 << i);
			int y = rec_used | rec[i];
			int tt = 0;
			for(int j = 0; j < len; j++)
				if(((y >> j) & 1) == 1 && ((rec_used >> j) & 1) == 0) tt++;
			tt = tt * tt;
			if(dp[x] == -1 || dp[x] > dp[rec_vis] + tt){
				dp[x] = dp[rec_vis] + tt;
				r++;
				q[r] = x;
				q_used[r] = y;
			}
		}
	}
	return dp[(1 << n) - 1];
}

```
------
# TopCoder SRM 667 Div.2 T3
## 解题思路
乍一看好像很麻烦呐。看起来有后效性？仿佛不能用dp？唉？然而数据只有30？所以考虑以下乱搞？
emmm
我们还是dp，大不了扩大维数，增加状态数。嗯。我们令dp[i][j][k]表示前i幢楼，在第i幢楼这里开了j家店，同时计划在下一幢楼开k家能获得的利益。那么我们可以得到状态方程
$$
dp[i][j][k] = max(dp[i][j][k], dp[i - 1][l][j] + j * c[i * 3 * m + (j + k + l) - 1])(0 <= l <= m)
$$
特判一下第一栋楼和最后一栋楼。
同时注意下小细节
1、题目里并没有给出c[-1]的定义，所以任意相邻三栋楼中一定要开一家：）
2、只有一栋楼的情况：）
时间复杂度O(n^4), 空间复杂度O(n^3)【这个我觉得还是叫暴力比较好：捂脸：】

## 参考代码段

```C++
#include <bits/stdc++.h>
using namespace std;

class ShopPositions {
public:
    int maxProfit( int n, int m, vector <int> c );
};
int f[40][40][40];
int ShopPositions::maxProfit(int n, int m, vector <int> c) {
    memset(f, 0, sizeof(f));
	int ans = 0;
	for(int j = 0; j <= m; j++)//特判第一栋楼
		for(int k = 0; k <= m; k++){
			if(j + k == 0) continue;
			f[0][j][k] = max(f[0][j][k], j * c[0 * 3 * m + (j + k) - 1]);
			ans = max(ans, f[0][j][k]);
		}
	for(int i = 1; i < n - 1; i++)
		for(int j = 0; j <= m; j++)
			for(int k = 0; k <= m; k++)//O(n^4)DP
				for(int l = 0; l <= m; l++){  
					if(j + k + l == 0) continue;
					f[i][j][k] = max(f[i][j][k], f[i - 1][l][j] + j * c[i * 3 * m + (j + k + l) - 1]);
					ans = max(ans, f[i][j][k]);
				}
	if(n > 1)//如果不止一栋楼，就特判最后一栋楼
	for(int j = 0; j <= m; j++)
		for(int l = 0; l <= m; l++){
			if(j + l == 0) continue;
			f[n - 1][j][0] = max(f[n - 1][j][0], f[n - 2][l][j] + j * c[(n - 1) * 3 * m + (j + l) - 1]);
			ans = max(ans, f[n - 1][j][0]);
		}
	return ans;
}
```