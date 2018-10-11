# POJ3233 [C - Matrix Power Series] 矩阵乘法

<center>矩阵乘法</center>

## 解题思路

题目里要求$\sum_{i=1}^kA^i$，我们不妨再加上一个单位矩阵，求$\sum_{i=0}^kA^i$。然后我们发现这个式子可以写成这样的形式：$A(A(A...)+E)+E)+E$于是，我们可以将$*A+E$看做一次变换，然后尝试构造一个矩阵。我们发现：


$$
(\left[
\begin{matrix}
A & E \\
0 & E
\end{matrix}
\right])^n=
\left[
\begin{matrix}
A^{n+1} & E+A+...+A^n \\
0 & E
\end{matrix}
\right]
$$
然后做法就比较显然了。

~~不清楚矩阵乘法的可以了解一下线性代数~~

## 参考程序

```C++
#include <cstdio>
#include <cstring>
#define LL long long
using namespace std;

LL n, k, m;
struct Matrix {
    LL A[ 70 ][ 70 ];
    Matrix operator * ( const Matrix Other ) const {
        Matrix Ans;
        memset( Ans.A, 0, sizeof( Ans.A ) );
        for( LL i = 1; i <= 2 * n; ++i )
            for( LL j = 1; j <= 2 * n; ++j )
                for( LL k = 1; k <= 2 * n; ++k )
                    Ans.A[ i ][ j ] = ( Ans.A[ i ][ j ] + A[ i ][ k ] * Other.A[ k ][ j ] % m ) % m;
        return Ans;
    }
};
Matrix A, E;

int main() {
    scanf( "%lld%lld%lld", &n, &k, &m );
    ++k;
    memset( A.A, 0, sizeof( A.A ) );
    for( LL i = 1; i <= n; ++i )
        for( LL j = 1; j <= n; ++j ) scanf( "%lld", &A.A[ i ][ j ] );
    for( LL i = 1; i <= n; ++i ) 
        for( LL j = 1; j <= n; ++j ) A.A[ i ][ j ] %= m;
    for( LL i = 1; i <= n; ++i ) 
        A.A[ i ][ i + n ] = 1;
    for( LL i = 1; i <= n; ++i )
        A.A[ i + n ][ i + n ] = 1;
    memset( E.A, 0, sizeof( E.A ) );
    for( LL i = 1; i <= 2 * n; ++i ) E.A[ i ][ i ] = 1;
    for( ; k; k >>= 1, A = A * A )
        if( k & 1 ) E = E * A;
    for( LL i = 1; i <= n; ++i ) E.A[ i ][ i + n ] = ( E.A[ i ][ i + n ] + m - 1 ) % m;
    for( LL i = 1; i <= n; ++i ) {
        for( LL j = 1; j <= n; ++j ) printf( "%lld ", E.A[ i ][ j + n ] );
        printf( "\n" );
    }
    return 0;
}
```

