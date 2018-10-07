# AtCoder4351 Median of Medians 二分, 树状数组

<center>二分, 树状数组</center>

## 题目大意

定义一个从小到大的数列的中位数为第$ \frac{n}{2}+1 $项。求一个序列的所有连续子序列的中位数的中位数。$(n \leqslant 100000)$

## 问题分析

由于$n$的范围较大，所以不可能把序列构造出来。我们不妨换个角度分析。我们设最后的序列总共有$N=\frac{n(n-1)}{2}$项。

若最终答案为$x$，那么也就是说，有$\frac{N}{2}+1$项的中位数不大于$x$。如果我们令原序列中小于等于$x$的数为$1$，否则为$-1$，那么这个又等价于有$\frac{N}{2}+1$段子区间和为正。所以我们可以二分答案，求最小的$x$，使得上述条件成立。

至于如何求和为正的子区间数，我们用前缀和+树状数组即可。$i$对答案的贡献就是$\sum_{j=1}^{i-1}sum[j]<sum[i]$。

## 参考程序

```C++
#include <bits/stdc++.h>
using namespace std;

long long n, a[ 100010 ];
long long Sum[ 100010 ], l, r;
long long Ans;
long long Tree[ 200010 ];

long long Lowbit( long long x ) { return x & -x; }

void Add( long long x ) {
    while( x <= 200001 ) {
        ++Tree[ x ];
        x += Lowbit( x );
    }
    return;
}

long long Query( long long x ) {
    long long ans = 0;
    while( x ) {
        ans += Tree[ x ];
        x -= Lowbit( x );
    }
    return ans;
}

int main() {
    scanf( "%lld", &n );
    Ans = n * ( n + 1 ) / 4 + 1;
    for( long long i = 1; i <= n; ++i ) scanf( "%lld", &a[ i ] );
    l = 0; r = 1e9 + 1;
    while( l < r ) {
        long long mid = l + r >> 1;
        for( long long i = 1; i <= n; ++i ) 
            if( a[ i ] > mid ) Sum[ i ] = -1; else Sum[ i ] = 1;
        Sum[ 0 ] = 0;
        for( long long i = 1; i <= n; ++i ) Sum[ i ] += Sum[ i - 1 ];
        for( long long i = 0; i <= n; ++i ) Sum[ i ] += 100001;
        memset( Tree, 0, sizeof( Tree ) );
        Add( Sum[ 0 ] );
        long long Cnt = 0;
        for( long long i = 1; i <= n; ++i ) {
            Cnt += Query( Sum[ i ] - 1 );
            Add( Sum[ i ] );
        }
        if( Cnt >= Ans ) r = mid; else l = mid + 1;
    }
    printf( "%lld\n", l );
    return 0;
}
```

