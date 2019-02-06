# CodeForces - 28C Bath Queue  概率与期望

<center>数学_概率与期望</center>

~~我概率期望真是垃圾……，这题搞了两个钟头……~~

## 题意
有$n$个人，$m$个浴室，每个浴室里有$a_i$个浴缸。每个人会等概率随机选择一个浴室，然后每个浴室中尽量平分到每个浴缸。问期望最长排队队伍长度是多少？
## 解题思路
我看网上的题解都是直接$DP$期望，然而本蒟蒻看不懂那个递推式是什么鬼……有没有$dalao$来解释一下啊……
付一下别人的题解，摘自https://www.cnblogs.com/LzyRapx/p/7692702.html
用状态$ dp[i][j][k] $表示还剩$ i$ 间浴室，还剩 $j$ 个人，之前最长队伍的长度为 $k$ 的期望最长队伍长度。
那么状态转移方程为:
$$
dp[i][j][k]=∑^m_{i=1}∑^n_{j=0}∑^n_{k=1}∑^j_{c=1}(dp[i−1][j−c][max(k,(c+a[i]−1)/a[i])]\times
(i−1)^{j−c}/i^j \times (j,c))
$$
其中 $c$ 是枚举当前去第 $j$ 间浴室的人数。
那么答案就是$ dp[m][n][0]$ 。


----伪分割线----

下面是我自己的概率做法。
我们令$dp[i][j][k]$表示已经选了$i$个浴室，还剩$j$个人，最长长度为$k$的概率。**注意下标意义的定义与上面的不一样。**
那么我们可以得到状态转移方程：
$$
dp[i+1][j-c][max(k, \frac{c+a[i+1]-1}{a[i+1]})] += dp[i][j][k]\times {j \choose c} / m^c
$$
这里感谢$PSCdalao$帮我斧正这个方程，并阐述了为什么是除以$m^c$。
下面我们解释这个方程的意义。
现在在状态$dp[i][j][k]$，那么我们可以从剩下$j$个人中，选出$c$个。这时候下标就变为了$dp[i+1][j-c][max(k, \frac{c+a[i+1]-1}{a[i+1]})]$。而从前一个状态转到这一个方案的概率就是${j \choose c} / m^c$（从$j$个人中选$c$个，而这$c$个人本来可以有$m^c$种选法）。我们再来仔细看一下为什么是$m^c$而不是$(m-i)^c$。
我们把n个人分到m个澡堂的某种分法的概率应当为
$$
\frac{{n\choose p_1}{n-p_1\choose p_2}{n-p_1-p_2\choose p_3}...}{m^n}
$$
然后我们拆开分别乘在每一步中，就成了
$$
\frac{{n\choose p_1}}{m^{p_1}} \times \frac{{n-p_1\choose p_2}}{m^{p_2}} \times \frac{{n-p_1-p_2\choose p_3}}{m^{p_3}}...
$$
所以就有方程中的“$/m^c$”。
同时，我们可以看出，我们同样可以先求出方案数，最后再除以$m^c$。（感谢$DYTdalao$提供这个好主意）

到这里，我们求出了概率。最后把每一个概率乘以长度，累加就好了。

~~不要问我精度问题，反正全部开$double$，然后什么都不管就可以了QwQ~~

# 参考程序
```C++
#include <cmath>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int Maxn = 60;
int n, m, a[ Maxn ];
double dp[ Maxn ][ Maxn ][ Maxn ], C[ 60 ][ 60 ];

double Power( double x, int y ) {
	if( y == 0 ) return 1.0;
	double t = Power( x, y / 2 );
	t = t * t;
	if( y % 2 == 1 ) t = t * x;
	return t;
}

void init() {
	memset( C, 0, sizeof( C ) );
	C[ 0 ][ 0 ] = 1.0;
	for( int i = 1; i <= 50; ++i ) {
		C[ i ][ 0 ] = 1.0;
		for( int j = 1; j <= i; ++j ) C[ i ][ j ] = C[ i - 1 ][ j - 1 ] + C[ i - 1 ][ j ];
	}
	return;
}

int main() {
	init();
	memset( dp, 0, sizeof( dp ) );
	scanf( "%d%d", &n, &m );
	dp[ 0 ][ n ][ 0 ] = 1.0;
	for( int i = 1; i <= m; ++i ) scanf( "%d", &a[ i ] );
	for( int i = 0; i < m - 1; ++i ) 
		for( int j = 0; j <= n; ++j ) 
			for( int k = 0; k <= n; ++k )
				for( int c = 0; c <= j; ++c ) 
					dp[ i + 1 ][ j - c ][ max( k, ( c + a[ i + 1 ] - 1 ) / a[ i + 1 ] ) ] += 
						dp[ i ][ j ][ k ] * C[ j ][ c ] / Power( m * 1.0, c );
	for( int j = 0; j <= n; ++j )
		for( int k = 0; k <= n; ++k ) 
			dp[ m ][ 0 ][ max( k, ( j + a[ m ] - 1 ) / a[ m ] ) ] += dp[ m - 1 ][ j ][ k ] * 1.0 / Power( m * 1.0, j );
	double ans;
	for( int i = 0; i <= n; ++i )
		ans += i * dp[ m ][ 0 ][ i ];
	printf( "%.10lf\n", ans );
	return 0;
}
```