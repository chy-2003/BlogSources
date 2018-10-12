# POJ 3613 [ Cow Relays ] DP，矩阵乘法

<center>DP，矩阵乘法</center>

##　解题思路

首先考虑最暴力的做法。对于每一步，我们都可以枚举每一条边，然后更新每两点之间经过$k$条边的最短路径。但是这样复杂度无法接受，我们考虑优化。

由于点数较少（其实最多只有$200$个点），$N$较大，考虑优化$N$。我们发现，其实可以直接从经过$i$条边的最短路和经过$j$条边的最短路推出经过$i+j$条边的最短路。这样的话，我们可以把每两点间的最短路保存下来，然后用类似于矩阵快速幂的做法就可以了。

最后为了不超时，我们需要将点的编号离散。最后时间复杂度是$O(T^3logN)​$。

## 参考程序

```C++
#include <cstdio>
#include <cstring>
#include <map>
#define LL long long
using namespace std;

const LL INF = 2147483647;
const LL MaxT = 110, MaxPoint = 210;
LL N, T, S, E;
LL Length[ MaxT ], I1[ MaxT ], I2[ MaxT ];
LL Num;

namespace myHash {	
    LL Reflection[ 1010 ];
    map< LL, LL > Map;
    
    void Hash() {
        memset( Reflection, 0, sizeof( Reflection ) );
        Map.clear();
        for( LL i = 1; i <= T; ++i ) {
            if( Map.find( I1[ i ] ) == Map.end() ) Map[ I1[ i ] ] = 1;
            if( Map.find( I2[ i ] ) == Map.end() ) Map[ I2[ i ] ] = 1;
        }
        LL Last = 0;
        for( map< LL, LL >::iterator it = Map.begin(); it != Map.end(); ++it )
            Reflection[ ( *it ).first ] = ++Last;
        Num = Last;
        for( LL i = 1; i <= T; ++i ) {
            I1[ i ] = Reflection[ I1[ i ] ];
            I2[ i ] = Reflection[ I2[ i ] ];
        }
        return;
    }
}//myHash

struct Matrix {
    LL A[ MaxPoint ][ MaxPoint ];
    void Clear() { 
        for( LL i = 1; i <= 200; ++i ) 
            for( LL j = 1; j <= 200; ++j ) 
                A[ i ][ j ] = INF;
        return;
    }
    Matrix operator * ( const Matrix Other ) const {
        Matrix Ans;
        Ans.Clear();
        for( LL i = 1; i <= Num; ++i )
            for( LL j = 1; j <= Num; ++j ) 
                for( LL k = 1; k <= Num; ++k )
                    Ans.A[ i ][ j ] = min( Ans.A[ i ][ j ], A[ i ][ k ] + Other.A[ k ][ j ] );
        return Ans;
    }
};

Matrix Basic, Ans;

void Init() {
    Basic.Clear();
    Ans.Clear();
    for( int i = 1; i <= 200; ++i ) Ans.A[ i ][ i ] = 0;
    for( int i = 1; i <= T; ++i ) {
        Basic.A[ I1[ i ] ][ I2[ i ] ] = min( Basic.A[ I1[ i ] ][ I2[ i ] ], Length[ i ] );
        Basic.A[ I2[ i ] ][ I1[ i ] ] = min( Basic.A[ I2[ i ] ][ I1[ i ] ], Length[ i ] );
    }
    return;
}

int main() {
    scanf( "%lld%lld%lld%lld", &N, &T, &S, &E );
    for( LL i = 1; i <= T; ++i )
        scanf( "%lld%lld%lld", &Length[ i ], &I1[ i ], &I2[ i ] );
    myHash::Hash();
    Init();
    for( ; N; N >>= 1, Basic = Basic * Basic ) 
        if( N & 1 ) Ans = Basic * Ans;
    printf( "%lld\n", Ans.A[ myHash::Reflection[ S ] ][ myHash::Reflection[ E ] ] );
    return 0;
}
```

