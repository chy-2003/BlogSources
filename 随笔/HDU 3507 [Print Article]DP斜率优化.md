# HDU 3507 [Print Article]DP斜率优化

<center>DP斜率优化</center>

## 题目大意

给定一个长度为$n(n \leqslant 500000)$的数列，将其分割为连续的若干份，使得 $ \sum ((\sum_{i=j}^kC_i) +M) $ 最小。其中$C_i$为序列中的项的值，$M$为常数。$ j,k $ 表示在原序列中连续的某一段的起始位置和结束位置。

## 解题思路

考虑到$n$的范围巨大，肯定不能用$O(n^2)$的暴力DP，而贪心又显然有问题，所以我们只能尝试对DP优化。

我们设$f[i]$为前$i$项作为子问题的解，$sum[i]$为前$i$项的前缀和。那么，若从$i$转移到$k$优于从$j$转移到$k$（不妨令$i > j$）就有：
$$
f[i]+M+(sum[k]-sum[i])^2 < f[j]+M+(sum[k]-sum[j])^2
$$
化简，得
$$
\frac{f[i]-f[j]+sum[i]^2-sum[j]^2}{2sum[i]-2sum[j]}<sum[k]
$$
到这里，做法就显然了，就是DP斜率优化。

接下来就在这道题的基础上大致分析一下什么是斜率优化。

------

我们不妨令$Y[i]=f[i]-sum[i]^2,X[i]=2sum[i]$。那么上面不等式的左边就变为了$\frac{Y[i]-Y[j]}{X[i]-X[j]}$。这个东西是不是很像斜率呢？$X,Y$可以看成点。我们不妨设现在从左至右有$3$个点$i,j,k$，$i,j$斜率为$l_1$，$j,k$斜率为$l_2$。接下来我们考虑$l_1,l_2$。

当$l_2 \leqslant l_1$时，若$sum[k] \leqslant l_2 \leqslant l_1$，那么最优值不是$j,k$；若$l_2 < sum[k] \leqslant l_1$，那么$k$比$j$优；若$l_2 \leqslant l_1 < sum[k]$，那么$k$在$i,j,k$中最优。所以不论如何，$j$都不会成为当前最优方案，我们不妨删掉$j$。

当$l_1 < l2$时，若$sum[k] \leqslant l_1 < l_2$，那么最优值可能是$i$；若$l_1 < sum[k] \leqslant l_2$，那么$j$在$i,j,k$中最优；若$l_1 < l_2 < sum[k]$，那么最优值可能为$k$。

进过如上分析，我们发现，我们只需要保留在图上逐个连线后样子为下凸的一些点。同时我们又发现，若从点$i$转移为当前最优，那么在图上看来这个点应该与斜率为$sum[k]$的直线“相切”。所以我们转移的时候只需要找在保留的点中，向前斜率小于$sum[k]$，向后斜率大于$sum[k]$的点就可以了。

最后，这里sum[k]单调不减，所以找当前最优的转移可以优化；若遇到$sum[k]$不单调的情况，二分查找即可。

tip:推式子的时候不能忽略取等的情况。我就是因为$Greater$函数中漏了取等的情况，听取WA声一片……
补：后来发现实际上是可能不严格递增，导致判断的时候某个结果为$0$



## 参考程序



```C++
#include <bits/stdc++.h>
#define LL long long
using namespace std;

LL N, M, a[ 500010 ], Sum[ 500010 ], F[ 500010 ];
LL L, R, Queue[ 500010 ];

LL sqr( LL x ) { return x * x; }

bool Less( LL j, LL i, LL t ) {
    return F[ i ] - F[ j ] + sqr( Sum[ i ] ) - sqr( Sum[ j ] ) < 2 * Sum[ t ] * ( Sum[ i ] - Sum[ j ] );
}

bool Greater( LL k, LL j, LL i ) {
    LL X2 = 2 * ( Sum[ i ] - Sum[ j ] );
    LL Y2 = F[ i ] - F[ j ] + sqr( Sum[ i ] ) - sqr( Sum[ j ] );
    LL X1 = 2 * ( Sum[ j ] - Sum[ k ] );
    LL Y1 = F[ j ] - F[ k ] + sqr( Sum[ j ] ) - sqr( Sum[ k ] );
    return X1 * Y2 <= X2 * Y1;
}

int main() {
    while( scanf( "%lld%lld", &N, &M ) == 2 ) {
        memset( a, 0, sizeof( a ) );
        memset( Sum, 0, sizeof( Sum ) );
        memset( F, 0, sizeof( F ) );
        memset( Queue, 0, sizeof( Queue ) );
        L = R = 0;
    	for( LL i = 1; i <= N; ++i ) scanf( "%lld", &a[ i ] );
    	for( LL i = 1; i <= N; ++i ) Sum[ i ] = Sum[ i - 1 ] + a[ i ];
    	R = 1; Queue[ 0 ] = 0;
        for( LL i = 1; i <= N; ++i ) {
            while( L + 1 < R && Less( Queue[ L ], Queue[ L + 1 ], i ) )
                ++L;
            F[ i ] = F[ Queue[ L ] ] + M + sqr( Sum[ i ] - Sum[ Queue[ L ] ] );
            while( L + 1 < R && Greater( Queue[ R - 2 ], Queue[ R - 1 ], i ) )
                --R;
            Queue[ R++ ] = i;
        }
        printf( "%lld\n", F[ N ] );
    }
    return 0;
}
```