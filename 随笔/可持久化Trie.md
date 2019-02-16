# 可持久化Trie

<center>教程_数据结构</center>

> 例题：[luoguP4735](https://www.luogu.org/problemnew/show/P4735)

可持久化$Trie$嘛，就和可持久化线段树差不多。这篇文章只是借例题讲一讲如何截取一段时间的信息。

直接讲题大家就可理解。

## 题目大意

有两种操作，第一种在数组末尾加上一个数，第二种在$l\leqslant p\leqslant r$中求最大的$ a[p] \bigoplus  a[ p + 1 ] \bigoplus ...\bigoplus a[ N ] \bigoplus  x$。

## 问题分析

我们先求一个异或前缀和，记为$A$。那么第二问就变成了求在$l-1 \leqslant p \leqslant r-1$中最大的$A[p]\bigoplus A[N]\bigoplus x$。

然后麻烦之处就是在$p$的范围限制了。

右端点限制非常方便，左端点的限制可以这样解决：

我们记录一个数组$Cnt$，$Cnt_i$表示$Trie$中节点$i$被多少个数经过。那么我们只要判断$Cnt_{Right}-Cnt_{Left}$就可以知道在这个范围内是否可行。

解释得不是很清楚，大家根据程序理解一下吧。

## 参考程序

```C++
// luogu-judger-enable-o2
#include <bits/stdc++.h>
#define LL long long
using namespace std;

const int MaxBit = 24;
const int MaxN = 600010;
int N, M, A[ MaxN ];
int Trie[ MaxBit * MaxN ][ 2 ], Cnt[ MaxBit * MaxN ], Used, Root[ MaxN ];
char Ch[ 10 ];

inline void Add( int Index, int History, int Bit, int Value );
inline int Query( int Left, int Right, int Bit, int Value );

int main() {
    scanf( "%d%d", &N, &M );
    Root[ 0 ] = ++Used;
    Add( Root[ 0 ], 0, MaxBit, 0 );
    int i;
    for( i = 1; i <= N; ++i ) 
        scanf( "%d", &A[ i ] );
    for( i = 1; i <= N; ++i ) 
        A[ i ] = A[ i - 1 ] ^ A[ i ];
    for( i = 1; i <= N; ++i ) {
        Root[ i ] = ++Used;
        Add( Root[ i ], Root[ i - 1 ], MaxBit, A[ i ] );
    }
    for( i = 1; i <= M; ++i ) {
        scanf( "%s", Ch );
        if( Ch[ 0 ] == 'A' ) {
            scanf( "%d", &A[ ++N ] );
            A[ N ] = A[ N - 1 ] ^ A[ N ];
            Root[ N ] = ++Used;
            Add( Root[ N ], Root[ N - 1 ], MaxBit, A[ N ] );
        }
        if( Ch[ 0 ] == 'Q' ) {
            int l, r, x;
            scanf( "%d%d%d", &l, &r, &x );
            x ^= A[ N ];
            --l; --r;
            if( l == 0 ) 
                printf( "%d\n", Query( 0, Root[ r ], MaxBit, x ) );
            else 
                printf( "%d\n", Query( Root[ l - 1 ], Root[ r ], MaxBit, x ) );
        }
    }
    return 0;
}


inline void Add( int Index, int History, int Bit, int Value ) {
    if( Bit < 0 ) return;
    int T = ( Value >> Bit ) & 1;
    Trie[ Index ][ !T ] = Trie[ History ][ !T ];
    Trie[ Index ][ T ] = ++Used;
    Cnt[ Trie[ Index ][ T ] ] = Cnt[ Trie[ History ][ T ] ] + 1;
    Add( Trie[ Index ][ T ], Trie[ History ][ T ], Bit - 1, Value );
    return;
}

inline int Query( int Left, int Right, int Bit, int Value ) {
    if( Bit < 0 ) return 0;
    int T = ( Value >> Bit ) & 1;
    if( Cnt[ Trie[ Right ][ !T ] ] - Cnt[ Trie[ Left ][ !T ] ] > 0 ) 
        return ( 1 << Bit ) + Query( Trie[ Left ][ !T ], Trie[ Right ][ !T ], Bit - 1, Value );
    else 
        return Query( Trie[ Left ][ T ], Trie[ Right ][ T ], Bit - 1, Value );
}

```

