# HDU 2829 [Lawrence] DP斜率优化

## 解题思路

首先肯定是考虑如何快速求出一段铁路的价值。
$$
\sum_{i=1}^k \sum_{j=1, j\neq i}^kA[i]A[j]=(\sum_{i=1}^kA[i])^2-\sum_{i=1}^kA[i]^2
$$
那么我们要维护如下两个东西，就可以在$O(1)$内求出一段铁路的价值了。

```C++
for( LL i = 1; i <= N; ++i ) Sum[ i ] = Sum[ i - 1 ] + A[ i ];
for( LL i = 1; i <= N; ++i ) SumOfSqr[ i ] = SumOfSqr[ i - 1 ] + A[ i ] * A[ i ];
```

然后我们考虑打一个最暴力的DP。

我们令$F[i][j]$为到第$i$个仓库，炸了$j$次的最小总价值：

```C++
for( LL i = 1; i <= N; ++i ) 
    F[ i ][ 0 ] = Sum[ i ] * Sum[ i ] - SumOfSqr[ i ];
for( LL j = 1; j <= M; ++j ) 
    for( LL i = j + 1; j <= N; ++j ) {
        F[ i ][ j ] = INF;
        for( LL k = j; k < i; ++k ) 
            F[ i ][ j ] = min( F[ i ][ j ], F[ k ][ j - 1 ] + sqr( Sum[ i ] - Sum[ k ] ) - ( SumOfSqr[ i ] - SumOfSqr[ k ] ) );
    }
Ans = F[ N ][ M ];
```

为了节省空间，我们滚动掉一维：

```C++
for( LL i = 1; i <= N; ++i ) 
    F1[ i ] = Sum[ i ] * Sum[ i ] - SumOfSqr[ i ];
for( LL j = 1; j <= M; ++j ) {
    for( LL i = j + 1; j <= N; ++j ) {
        F2[ i ] = INF;
        for( LL k = j; k < i; ++k ) 
            F2[ i ] = min( F2[ i ], F1[ k ] + sqr( Sum[ i ] - Sum[ k ] ) - ( SumOfSqr[ i ] - SumOfSqr[ k ] ) );
    }
    memcpy( F1, F2, sizeof( F2 ) );
}
Ans = F1[ N ];
```

最后考虑优化转移复杂度：

设$l > k$，且从$l$转移优于从$k$转移，那么就有：
$$
F_1[l]+(S[i]-S[l])^2-(SOS[i]-SOS[l])<F_1[k]+(S[i]-S[k])^2-(SOS[i]-SOS[k])
$$

$$
(F_1[l]+S[l]^2+SOS[l])-(F_1[k]+S[k]^2+SOS[k])<2S[i](S[k]-S[l])
$$

$$
\frac{(F_1[l]+S[l]^2+SOS[l])-(F_1[k]+S[k]^2+SOS[k])}{2S[l]-2S[k]}<S[i]
$$

然后我们就可以进行斜率优化了。

斜率优化的具体讲解见[这里](https://www.cnblogs.com/chy-2003/p/9749925.html)。

## 参考程序

```C++
#include <bits/stdc++.h>
#define LL long long
using namespace std;

const LL Maxn = 1010;
LL n, m;
LL A[ Maxn ], Sum[ Maxn ], SumOfSqr[ Maxn ], F1[ Maxn ], F2[ Maxn ];
LL L, R, Queue[ Maxn ];

inline LL Sqr( LL x ) { return x * x; }

inline LL GetSum( LL r, LL l ) {
    return Sqr( Sum[ r ] - Sum[ l ] ) - ( SumOfSqr[ r ] - SumOfSqr[ l ] );
}

inline bool Less( LL i, LL j, LL T ) {
    LL X = 2 * ( Sum[ j ] - Sum[ i ] );
    LL Y = ( F1[ j ] + Sqr( Sum[ j ] ) + SumOfSqr[ j ] ) - 
           ( F1[ i ] + Sqr( Sum[ i ] ) + SumOfSqr[ i ] );
    return Y < T * X;
}

inline bool Greater( LL i, LL j, LL k ) {
    LL X1 = 2 * ( Sum[ j ] - Sum[ i ] );
    LL Y1 = ( F1[ j ] + Sqr( Sum[ j ] ) + SumOfSqr[ j ] ) - 
           	( F1[ i ] + Sqr( Sum[ i ] ) + SumOfSqr[ i ] );
    LL X2 = 2 * ( Sum[ k ] - Sum[ j ] );
    LL Y2 = ( F1[ k ] + Sqr( Sum[ k ] ) + SumOfSqr[ k ] ) - 
           	( F1[ j ] + Sqr( Sum[ j ] ) + SumOfSqr[ j ] );
    return X2 * Y1 >= X1 * Y2;
}

void Work() {
    for( LL i = 1; i <= n; ++i ) scanf( "%lld", &A[ i ] );
    Sum[ 0 ] = 0; SumOfSqr[ 0 ] = 0;
    for( LL i = 1; i <= n; ++i ) Sum[ i ] = Sum[ i - 1 ] + A[ i ];
    for( LL i = 1; i <= n; ++i ) SumOfSqr[ i ] = SumOfSqr[ i - 1 ] + Sqr( A[ i ] );
    for( LL i = 1; i <= n; ++i ) 
        F1[ i ] = GetSum( i, 0 );
    for( LL j = 1; j <= m; ++j ) {
        L = R = 0; Queue[ R++ ] = j;
        memset( F2, 0, sizeof( F2 ) );
        for( LL i = j + 1; i <= n; ++i ) {
            while( L + 1 < R && Less( Queue[ L ], Queue[ L + 1 ], Sum[ i ] ) )
                ++L;
            F2[ i ] = F1[ Queue[ L ] ] + GetSum( i, Queue[ L ] );
            while( L + 1 < R && Greater( Queue[ R - 2 ], Queue[ R - 1 ], i ) )
                --R;
            Queue[ R++ ] = i;
        }
        memcpy( F1, F2, sizeof( F2 ) );
    }
    printf( "%lld\n", F1[ n ] / 2 );
    return;
}

int main() {
    scanf( "%lld%lld", &n, &m );
    while( !( n == 0 && m == 0 ) ) {
        Work();
        scanf( "%lld%lld", &n, &m );
    }
    return 0;
}
```

