# CodeForces 352C Jeff and Rounding

<center>杂题</center>

# 题意
有一个含有$2n(n \leqslant2000)$个实数的数列，取出$n$个向上取整，另$n$个向下取整。问取整后数列的和与原数列的和的差的绝对值。
就是说，令$a$为原数列，$b$为取整后数列，求
$$
min(abs(\sum_{i=1}^{2n}a-\sum_{i=1}^{2n}b))
$$
# 解题思路
刚开始大力猜了一波贪心结论，然后怒WA n发……
我也不知道怎么会想到以下这个奇怪的结论的……
如果我们设$n$个被向上取整的数的小数部分的和为$a$（其中本来为整数的有$x$个），设$n$个被向下取整的数的小数部分的和为$b$。那么答案就是$ans=abs(b-(n-a-x))=abs(b+a-n+x)$。同时我们注意到，原数列中所有数的和为$sum=a+b$，所以呢，就有$ans=abs(sum-n+x)$。
这，个，东，西，和，$a$，$b$，没，有，关，系！！！
然后容易发现$max(0,n-x)\leqslant x \leqslant min(n,x)$。所以就容易得出答案了！
# 参考程序
```C++
#include <cmath>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int Maxn = 2010;
double a[ Maxn << 1 ];
int n, t;//t就是上面说的x
double sum, ans;

int main() {
	scanf( "%d", &n );
	for( int i = 1; i <= n * 2; ++i ) scanf( "%lf", &a[ i ] );
	for( int i = 1; i <= n * 2; ++i ) 
		if( a[ i ] - floor( a[ i ] ) <= 1e-18 ) t++;
	for( int i = 1; i <= n * 2; ++i ) sum += a[ i ] - floor( a[ i ] );
	ans = 1e9;
	for( int i = max( 0, t - n ); i <= min( n, t ); ++i ) 
		ans = min( ans, abs( sum - n + i ) );
	printf( "%.3lf\n", ans );
	return 0;
}
```