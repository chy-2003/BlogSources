# YY的GCD【luoguP2257】

<center>数学_莫比乌斯反演</center>

---

## 题目大意
有至多$10000$组询问，问$1 < i \leqslant N \leqslant 10000000, 1 < j \leqslant M \leqslant 10000000$，并且$gcd(i, j)$为质数的有多少对。

## 解题思路
为了方便描述，我们定义$[]$，当$[]$中表达式为真时为$1$，否则为$0$。同时定义$Prime$为素数集合。
下面的讨论中，我们不妨设$N \leqslant M$。
我们设
$$
f(d)=\sum_{i=1}^M\sum_{j=1}^N[gcd( i, j ) \in Prime]\\
F(n)=\sum_{n|d}^Nf(d)=\lfloor\frac{M}{n}\rfloor\lfloor\frac{N}{n}\rfloor
$$
即，$f(d)$是当$gcd=d$时的答案数，$F(n)$是当$gcd$的答案数。
我们发现，求$F(n)$十分的方便，于是我们考虑能否通过$F(n)$将$f(n)$表述出来。

由莫比乌斯反演，得
$$
f(n)=\sum_{n|d}\mu(\frac{d}{n})F(d)
$$
其中$\mu$是莫比乌斯函数。

那么答案就可以表示为
$$
\begin{aligned}
Ans & = \sum_{n\in Prime}^Nf(n)\\
& = \sum_{n\in Prime}^N\sum_{n|d}\mu(\frac{d}{n})F(d)\\
& = \sum_{n\in Prime}^N\sum_{n|d}\mu(\frac{d}{n})\lfloor\frac{N}{d}\rfloor\lfloor\frac{M}{d}\rfloor\\
& = \sum_{d}^N\sum_{n|d,n\in Prime}\mu(\frac{d}{n})\lfloor\frac{N}{d}\rfloor\lfloor\frac{M}{d}\rfloor\\
& = \sum_{d}^N\lfloor\frac{N}{d}\rfloor\lfloor\frac{M}{d}\rfloor\sum_{n|d, n\in Prime} \mu(\frac{d}{n})
\end{aligned}
$$

通过稍微修改线筛，我们可以与处理出$\mu$，然后可以预处理出所有的$\sum_{n|d,n\in Prime}\mu(\frac{d}{n})$。最后再整除分块统计答案就可以了。

## 参考程序
程序中，mu即为$\mu$，$Sum$为前缀和。
```C++
#include <bits/stdc++.h>
using namespace std;

const int MaxN = 10000010;
int Mu[ MaxN ], Vis[ MaxN ];
long long Sum[ MaxN ];
int Num, Prime[ 1000010 ];

void Init() {
    Mu[ 1 ] = 1;
    for( int i = 2; i <= MaxN; ++i ) {
        if( !Vis[ i ] ) Prime[ ++Num ] = i, Mu[ i ] = -1;
        for( int j = 1; j <= Num && ( long long ) i * Prime[ j ] <= ( long long ) MaxN; ++j ) {
            Vis[ i * Prime[ j ] ] = 1;
            if( i % Prime[ j ] == 0 ) break;
            Mu[ i * Prime[ j ] ] = - Mu[ i ];
        }
    }
    for( int i = 1; i <= MaxN; ++i ) 
        for( int j = 1; j <= Num && ( long long ) i * Prime[ j ] <= ( long long ) MaxN; ++j )
            Sum[ i * Prime[ j ] ] += Mu[ i ];
    for( int i = 2; i <= MaxN; ++i ) Sum[ i ] += Sum[ i - 1 ];
    return;
}

void Work() {
    int N, M; 
    scanf( "%d%d", &N, &M );
    if( N > M ) swap( N, M );
    long long Ans = 0;
    for( int x = 1, y; x <= N; x = y + 1 ) {
        y = min( N / ( N / x ), M / ( M / x ) );
        Ans += 1LL * ( N / x ) * ( M / x ) * ( Sum[ y ] - Sum[ x - 1 ] );
    }
    printf( "%lld\n", Ans );
    return;
}

int main() {
    Init();
    int T; scanf( "%d", &T );
    for( ; T; --T ) Work();
    return 0;
}
```

