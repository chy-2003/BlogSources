# HDU 3480 Division DP斜率优化

<center>DP_斜率优化</center>

## 解题思路

第一步显然是将原数组排序嘛……然后分成一些不相交的子集，这样显然最小。重点是怎么分。

首先，我们写出一个最暴力的$DP$：

我们令$F[ i ][ j ] $为到第$i$位，分成$j$组的代价，我们可以写出如下$DP$

```C++
for( LL i = 1; i <= N; ++i ) F[ i ][ 1 ] = sqr( A[ i ] - A[ 1 ] );
for( LL j = 2; j <= M; ++j ) 
    for( LL i = j; i <= N; ++i ) {
        F[ i ][ j ] = INF;
        for( LL k = j - 1; k <= i - 1; ++k ) 
            F[ i ][ j ] = min( F[ i ][ j ], F[ k ][ j - 1 ] + sqr( A[ j ] - A[ k + 1 ] ) );
    }
```

滚动掉一维节省空间：

```C++
for( LL i = 1; i <= N; ++i ) F1[ i ] = sqr( A[ i ] - A[ 1 ] );
for( LL j = 2; j <= M; ++j ) {
    for( LL i = j; i <= N; ++i ) {
        F2[ i ] = INF;
        for( LL k = j - 1; k <= i - 1; ++k )
            F2[ i ] = min( F2[ i ], F1[ K ] + sqr( A[ j ] - A[ k + 1 ] ) );
    }
    memcpy( F1, F2, sizeof( F2 ) );
}
```

但是时间上依然难以接受。我们考虑优化最内层的转移复杂度。

不妨设转移的时候$l>k$且从$l$转移优于从$k$转移，那么我们就能得到如下不等式：
$$
F_1[l]+(A[j]-A[l+1])^2<F_1[k]+(A[j]-A[k+1])^2
$$
化简，得
$$
\frac{F_1[l]-F_1[k]+A[l+1]^2-A[k+1]^2}{2A[l+1]-2A[k+1]}<A[j]
$$
所以我们可以进行斜率优化。具体优化讲解可以看[这里]( https://www.cnblogs.com/chy-2003/p/9749925.html)。过程十分相似。

## 参考程序

```C++
#include <bits/stdc++.h>
#define LL long long
using namespace std;

LL N, M, A[ 10010 ], F1[ 10010 ], F2[ 10010 ];
LL L, R, Queue[ 10010 ];

inline LL sqr( LL x ) { return x * x; }

inline void Clear() {
    memset( A, 0, sizeof( A ) );
    memset( F1, 0, sizeof( F1 ) );
    return;
}

inline void Clear2() {
    memset( Queue, 0, sizeof( Queue ) );
    memset( F2, 0, sizeof( F2 ) );
    L = R = 0;
    return;
}

inline bool Less( LL i, LL j, LL T ) {
    LL DeltaX = 2 * ( A[ j + 1 ] - A[ i + 1 ] );
    LL DeltaY = F1[ j ] - F1[ i ] + sqr( A[ j + 1 ] ) - sqr( A[ i + 1 ] );
    return DeltaY < T * DeltaX;
}

inline bool Greater( LL i, LL j, LL k ) {
    LL DeltaX1 = 2 * ( A[ j + 1 ] - A[ i + 1 ] );
    LL DeltaX2 = 2 * ( A[ k + 1 ] - A[ j + 1 ] );
    LL DeltaY1 = F1[ j ] - F1[ i ] + sqr( A[ j + 1 ] ) - sqr( A[ i + 1 ] );
    LL DeltaY2 = F1[ k ] - F1[ j ] + sqr( A[ k + 1 ] ) - sqr( A[ j + 1 ] );
    return DeltaX2 * DeltaY1 >= DeltaX1 * DeltaY2;
}

void Work() {
    Clear();
    scanf( "%lld%lld", &N, &M );
    for( LL i = 1; i <= N; ++i ) scanf( "%lld", &A[ i ] );
    sort( A + 1, A + N + 1, less< LL >() );
    for( LL i = 1; i <= N; ++i ) F1[ i ] = sqr( A[ i ] - A[ 1 ] );
    for( LL j = 2; j <= M; ++j ) {
        Clear2();
        Queue[ R++ ] = j - 1;
        for( LL i = j; i <= N; ++i ) {
            while( L + 1 < R && Less( Queue[ L ], Queue[ L + 1 ], A[ i ] ) )
                ++L;
            F2[ i ] = F1[ Queue[ L ] ] + sqr( A[ i ] - A[ Queue[ L ] + 1 ] );
            while( L + 1 < R && Greater( Queue[ R - 2 ], Queue[ R - 1 ], i ) )
                --R;
            Queue[ R++ ] = i;
        }
        memcpy( F1, F2, sizeof( F2 ) );
    }
    printf( "%lld\n", F1[ N ] );
    return;
}

int main() {
    LL t; scanf( "%lld", &t );
    for( LL i = 1; i <= t; ++i ) {
        printf( "Case %lld: ", i );
        Work();
    }
    return 0;
}
```

