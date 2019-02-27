# luogu3768 简单的数学题

<center>数学_莫比乌斯反演, 数学_杜教筛</center>

---

[题目链接](https://www.luogu.org/problemnew/show/P3768)

## 问题分析

题目就是要求
$$
Ans=\sum_{i=1}^n\sum_{j=1}^nij\gcd(i,j)
$$
首先乱搞一波：
$$
\begin{aligned}
Ans&=\sum_{t=1}^n\sum_{i=1}^n\sum_{j=1}^nijt[\gcd(i,j)==t]\\
&=\sum_{t=1}^nt^3\sum_{i=1}^{\lfloor\frac{n}{t}\rfloor}\sum_{j=1}^{\lfloor\frac{n}{t}\rfloor}ij[\gcd(i,j)==1]
\end{aligned}
$$
然后对后面那一部做莫比乌斯反演：
$$
f(d)=\sum_{i=1}^n\sum_{j=1}^nij[\gcd(i,j)==1]\\
F(d)=\sum_{i=1}^n\sum_{j=1}^nij[d|\gcd(i,j)]=d^2\sum_{i=1}^{\lfloor\frac{n}{d}\rfloor}\sum_{j=1}^{\lfloor\frac{n}{d}\rfloor}ij\\
F(d)=\sum_{d|k}f(k)\\
f(d)=\sum_{d|k}\mu(\frac{k}{d})F(k)=\sum_{d|k}\mu(\frac{k}{d})k^2\sum_{i=1}^{\lfloor\frac{n}{k}\rfloor}\sum_{j=1}^{\lfloor\frac{n}{k}\rfloor}ij\\
f(1)=\sum_{k=1}^n\mu(k)k^2\sum_{i=1}^{\lfloor\frac{n}{k}\rfloor}\sum_{j=1}^{\lfloor\frac{n}{k}\rfloor}ij=\sum_{k=1}^n\mu(k)k^2(\sum_{i=1}^{\lfloor\frac{n}{k}\rfloor}i)^2
$$
那么就有：
$$
Ans=\sum_{t=1}^{n}t^3\sum_{k=1}^{n/t}\mu(k)k^2(\sum_{i=1}^{\lfloor\frac{n}{tk}\rfloor}i)^2
$$
令$K=tk$，
$$
\begin{aligned}
Ans&=\sum_{K=1}^n\sum_{t|K}t\mu(\frac{K}{t})K^2(\sum_{i=1}^{\lfloor\frac{n}{K}\rfloor}i)^2\\
&=\sum_{K=1}^nK^2(\sum_{i=1}^{\lfloor\frac{n}{K}\rfloor}i)^2\sum_{t|K}t\mu(\frac{K}{t})
\end{aligned}
$$
因为$\phi*1=id_1$，$1*\mu=e$，所以$id_1*\mu=\phi*1*\mu=\phi*e=\phi$。（其中$*$表示狄利克雷卷积）

那么
$$
\sum_{t|K}\frac{K}{t}\mu(t)=\phi(K)\\
Ans=\sum_{K=1}^nK^2(\sum_{i=1}^{\lfloor\frac{n}{K}\rfloor}i)^2\sum_{t|K}\frac{K}{t}\mu(t)=\sum_{K=1}^n(\sum_{i=1}^{\lfloor\frac{n}{K}\rfloor}i)^2K^2\phi(K)
$$
其中$(\sum_{i=1}^{\lfloor\frac{n}{K}\rfloor}i)^2$可以整除分块，后面我们考虑杜教筛：
$$
f(n)=n^2\phi(n)\\
S(n)=\sum_{i=1}^nf(i)\\
g(1)S(n)=\sum_{i=1}^n(g*f)(i)-\sum_{i=2}^ng(i)S(\lfloor\frac{n}{i}\rfloor)\\
g=id_2\\
S(n)=\sum_{i=1}^ni^3-\sum_{i=2}^ni^2S(\lfloor\frac{n}{i}\rfloor)
$$
那么就可以啦！时间复杂度$O(n^{\frac{2}{3}})​$。

## 参考程序

```C++
#include <bits/stdc++.h>
#define LL long long
using namespace std;

const LL Maxn066 = 6000000;
LL Phi[ Maxn066 + 10 ], Vis[ Maxn066 + 10 ], Prime[ Maxn066 ], Count;
LL Mod, n, Ans, Inv6;

void EulerSieve();
void Cal();
LL Inv( LL a );

int main() {
    scanf( "%lld%lld", &Mod, &n );
    Inv6 = Inv( 6 );
    EulerSieve();
    Cal();
    return 0;
}

void EulerSieve() {
    Phi[ 1 ] = 1;
    for( LL i = 2; i <= Maxn066; ++i ) {
        if( !Vis[ i ] ) {
            Phi[ i ] = i - 1;
            Prime[ ++Count ] = i;
        }
        for( LL j = 1; j <= Count && Prime[ j ] * i <= Maxn066; ++j ) {
            Vis[ Prime[ j ] * i ] = 1;
            if( !( i % Prime[ j ] ) ) {
                Phi[ Prime[ j ] * i ] = Phi[ i ] * Prime[ j ] % Mod;
                break;
            }
            Phi[ Prime[ j ] * i ] = Phi[ i ] * Phi[ Prime[ j ] ] % Mod;
        }
    }
    for( LL i = 2; i <= Maxn066; ++i ) Phi[ i ] = ( Phi[ i - 1 ] + Phi[ i ] * i % Mod * i % Mod ) % Mod;
    return;
}

map< LL, LL > Map;

LL S( LL Key ) {
    if( Key <= Maxn066 ) return Phi[ Key ];
    if( Map.find( Key ) != Map.end() ) return Map[ Key ];
    LL Ans = Key % Mod * ( Key % Mod + 1 ) / 2 % Mod; Ans = Ans * Ans % Mod;
    for( LL i = 2, j; i <= Key; i = j + 1 ) {
        j = Key / ( Key / i );
        LL T = j % Mod * ( j % Mod + 1 ) % Mod * ( 2 * j % Mod + 1 ) % Mod * Inv6 % Mod;	
        T = T - ( i % Mod - 1 ) * ( i % Mod ) % Mod * ( 2 * i % Mod - 1 ) % Mod * Inv6 % Mod;
        T = ( T % Mod + Mod ) % Mod;
        T = T * S( Key / i ) % Mod;
        Ans = ( ( Ans - T ) % Mod + Mod ) % Mod;
    }
    Map[ Key ] = Ans;
    return Ans;
}

void Cal() {
    for( LL i = 1, j; i <= n; i = j + 1 ) {
        j = n / ( n / i );
        LL T = ( 1 + n / i % Mod ) * ( n / i % Mod ) / 2 % Mod; T = T * T % Mod;
        LL Delta = ( ( S( j ) - S( i - 1 ) ) % Mod + Mod ) % Mod;
        Ans = ( Ans + T * Delta % Mod ) % Mod;
    }
    printf( "%lld\n", Ans );
    return;
}

void Exgcd( LL a, LL b, LL &x, LL &y ) {
    if( b == 0 ) {
        x = 1; y = 0; return;
    }
    Exgcd( b, a % b, y, x );
    y -= a / b * x;
    return;
}

LL Inv( LL a ) {
    LL x, y;
    Exgcd( a, Mod, x, y );
    if( x < 0 ) x += Mod;
    return x;
}

```



