# HDU 2157 [How many ways??] 

<center>DP</center>

## 解题思路

我们很愉快地发现了这题可以离线。然后我们就尝试预处理所有时间所有点对间的方案数。我们将其维护在$F$里。$F[i][j][k]$表示从$i$到$j$，经过$k$个点的方案数。

当我们从$k$向$k+1$转移的时候，我们枚举点$u$、$v$，如果$u$到$v$有一条边，那么我们将$F[i][v][k+1]$加上$F[i][u][k]$即可。直接做的做法在$O(kn^3)$，完全没问题。

如果我们将边的关系建成邻接表，有边为$1$，没边为$0$，那么就可以写成矩阵乘法的样子。嗯，就像参考程序里那样。

## 参考程序

```C++
#include <bits/stdc++.h>
using namespace std;

const int Mod = 1000;
int n, m, A[ 30 ][ 30 ], F[ 30 ][ 30 ][ 30 ], T;

void Work() {
    memset( A, 0, sizeof( A ) );
    memset( F, 0, sizeof( F ) );
    for( int i = 0; i < n; ++i ) F[ i ][ i ][ 0 ] = 1;
    for( int i = 0; i < m; ++i ) {
        int x, y; scanf( "%d%d", &x, &y );
        A[ x ][ y ] = 1;
    }
    for( int times = 1; times < 20; ++times )
        for( int i = 0; i < n; ++i )
            for( int j = 0; j < n; ++j )
                for( int k = 0; k < n; ++k ) 
                    F[ i ][ j ][ times ] = ( F[ i ][ j ][ times ] + A[ i ][ k ] * F[ k ][ j ][ times - 1 ] % Mod ) % Mod;
    int T; scanf( "%d", &T );
    for( int i = 1; i <= T; ++i ) {
        int x, y, k; scanf( "%d%d%d", &x, &y, &k );
        printf( "%d\n", F[ x ][ y ][ k ] );
    }
    return;
}

int main() {
    scanf( "%d%d", &n, &m );
    while( !( n == 0 && m == 0 ) ) {
        Work();
        scanf( "%d%d", &n, &m );
    }
    return 0;
}
```

