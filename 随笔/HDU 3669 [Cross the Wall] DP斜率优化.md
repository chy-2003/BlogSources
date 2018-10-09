# HDU 3669 [Cross the Wall] DP斜率优化

<center>DP斜率优化</center>

## 问题分析

首先，如果一个人的$w$和$h$均小于另一个人，那么这个人显然可以被省略。如果我们将剩下的人按$w[i]$递增排序，那么$h[i]$就是递减。

之后我们考虑DP。

我们设$f[i][j]$为到第$i$个人，打了$j$个洞的花费。于是我们可以得到如下DP过程：

```C++
for( LL i = 1; i <= N; ++i ) F[ i ][ 1 ] = w[ i ] * h[ 1 ];
for( LL j = 2; j <= K; ++j ) 
    for( LL i = j; i <= N; ++i ) {
        f[ i ][ j ] = INF;
        for( LL k = j - 1; k < i; ++k ) 
            F[ i ][ j ] = min( F[ i ][ j ], F[ k ][ j - 1 ] + w[ i ] * h[ k + 1 ] );
    }
Ans = F[ N ][ K ];
```

我们将第二维滚动掉，节省空间：

```C++
for( LL i = 1; i <= N; ++i ) F1[ i ] = w[ i ] * h[ 1 ];
for( LL j = 2; j <= K; ++j ) {
    for( LL i = j; i <= N; ++i ) {
        F2[ i ] = INF;
        for( LL k = j - 1; k < i; ++k )
            F2[ i ] = min( F2[ i ], F1[ k ] + w[ i ] * h[ k + 1 ] );
    }
    memcpy( F1, F2, sizeof( F2 ) );
}
Ans = F1[ N ][ K ];
```

再考虑优化最里面一层循环：

设$l>k$且从$l$转移优于从$k$转移，那么就有
$$
F1[l]+w[i]*h[l+1]<F1[k]+w[i]*h[k+1]
$$
化简，得
$$
\frac{F_1[l]-F_1[k]}{h[k+1]-h[l+1]}<w[i]
$$
然后就可以斜率优化了。具体的斜率优化讲解可以看[这里](https://www.cnblogs.com/chy-2003/p/9749925.html)。

## 参考程序

```C++
#include <bits/stdc++.h>
#define LL long long
using namespace std;

const LL INF = 1e18 + 10;
const LL MaxN = 50010, MaxK = 110;
LL N, K;
struct CitizenAttribute {
    LL Width, Hight;
    CitizenAttribute( LL Width_ = 0, LL Hight_ = 0 ) {
        Width = Width_; Hight = Hight_; return;
    }
    bool operator < ( const CitizenAttribute Other ) const {
        return Width < Other.Width || Width == Other.Width && Hight > Other.Hight;
    }
};
CitizenAttribute Citizens[ MaxN ];
bool IsSkiped[ MaxN ];
LL L, R, Queue[ MaxN ], F1[ MaxN ], F2[ MaxN ];
LL NumAfterSkip;

inline void Clear() {
    memset( Citizens, 0, sizeof( Citizens ) );
    memset( IsSkiped, false, sizeof( IsSkiped ) );
    memset( F1, 0, sizeof( F1 ) );
    return;
}

inline void SkipContainedCitizen() {
    CitizenAttribute Last = CitizenAttribute( 0, 0 );
    for( LL i = N; i >= 1; --i )
        if( Citizens[ i ].Hight <= Last.Hight ) IsSkiped[ i ] = true;
    	else Last = Citizens[ i ];
    NumAfterSkip = 0;
    for( LL i = 1; i <= N; ++i ) 
    	if( !IsSkiped[ i ] ) 
    		Citizens[ ++NumAfterSkip ] = Citizens[ i ];
    return;
}

inline bool Less( LL i, LL j, LL Limit ) {
    LL DeltaY = F1[ j ] - F1[ i ];
    LL DeltaX = Citizens[ i + 1 ].Hight - Citizens[ j + 1 ].Hight;
    return DeltaY <= Limit * DeltaX;
}

inline bool Greater( LL i, LL j, LL k ) {
    LL DeltaY1 = F1[ j ] - F1[ i ];
    LL DeltaY2 = F1[ k ] - F1[ j ];
    LL DeltaX1 = Citizens[ i + 1 ].Hight - Citizens[ j + 1 ].Hight;
    LL DeltaX2 = Citizens[ j + 1 ].Hight - Citizens[ k + 1 ].Hight;
    return DeltaY1 * DeltaX2 >= DeltaY2 * DeltaX1;
}

void Work() {
	LL Ans = INF;
    Clear();
    for( LL i = 1; i <= N; ++i ) 
        scanf( "%lld%lld", &Citizens[ i ].Width, &Citizens[ i ].Hight );
    sort( Citizens + 1, Citizens + N + 1 );
    SkipContainedCitizen();
    for( LL i = 1; i <= NumAfterSkip; ++i ) F1[ i ] = Citizens[ i ].Width * Citizens[ 1 ].Hight;
    Ans = min( Ans, F1[ NumAfterSkip ] );
    for( LL j = 2; j <= K && j <= NumAfterSkip; ++j ) {
    	memset( F2, 0, sizeof( F2 ) );
    	L = R = 0; memset( Queue, 0, sizeof( Queue ) );
    	Queue[ R++ ] = j - 1;
	    for( LL i = j; i <= NumAfterSkip; ++i ) {
	        while( L + 1 < R && Less( Queue[ L ], Queue[ L + 1 ], Citizens[ i ].Width ) )
	            ++L;
	        F2[ i ] = F1[ Queue[ L ] ] + Citizens[ Queue[ L ] + 1 ].Hight * Citizens[ i ].Width;
	        while( L + 1 < R && Greater( Queue[ R - 2 ], Queue[ R - 1 ], i ) ) 
	            --R;
	        Queue[ R++ ] = i;
	    }
	    memcpy( F1, F2, sizeof( F2 ) );
	    Ans = min( Ans, F1[ NumAfterSkip ] );
	}
	printf( "%lld\n", Ans );
	return;
}

int main() {
    while( scanf( "%lld%lld", &N, &K ) == 2 ) 
        Work();
    return 0;
}
```

