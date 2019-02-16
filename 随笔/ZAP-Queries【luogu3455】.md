# ZAP-Queries【luogu3455】

<center>数学_莫比乌斯反演</center>

---

## 题目大意
有不超过$50000$个询问，每次询问有多少正整数对$x$,$y$，满足$x\leqslant a$，$y \leqslant b$，并且$gcd(x,y)=c$。其中$a,b,c\leqslant 50000$
## 解题思路
我们发现
$$
Ans=f(n)=\sum_{x=1}^{a}\sum_{y = 1}^{b}[gcd( i, j)=c]
$$
当括号内表达式为真时，值为$1$，否则为$0$。
同时，我们设
$$
F(n)=\sum_{n|d}f(d)=\lfloor\frac{a}{n}\rfloor\lfloor\frac{b}{n}\rfloor
$$

由莫比乌斯反演，我们得
$$
f(d)=\sum_{d|n}\mu(\frac{n}{d})F(n)=\sum_{d|n}\mu(\frac{n}{d})\lfloor\frac{a}{n}\rfloor\lfloor\frac{b}{n}\rfloor=\sum_{i=1}^{min(\lfloor\frac{a}{n}\rfloor,\lfloor\frac{b}{n}\rfloor)}\mu(i)\lfloor\frac{a}{ni}\rfloor\lfloor\frac{b}{ni}\rfloor
$$

到这里，我们通过预处理$\mu$的前缀和和整除分块，就可以在$O(T*\sqrt{n})$解决。

## 参考程序
```c++
// luogu-judger-enable-o2
#include <bits/stdc++.h>
#define LL long long
using namespace std;

const LL Maxn = 50010;
LL Mu[ Maxn ], Sum[ Maxn ];
int Vis[ Maxn ];
LL Num;
LL Prime[ Maxn ];

void Init() {
    Mu[ 1 ] = 1;
    for( LL i = 2; i <= Maxn; ++i ) {
        if( !Vis[ i ] ) Mu[ i ] = -1, Prime[ ++Num ] = i;
        for( LL j = 1; j <= Num && Prime[ j ] * i <= Maxn; ++j ) {
            Vis[ Prime[ j ] * i ] = 1;
            if( i % Prime[ j ] == 0 ) break;
            Mu[ i * Prime[ j ] ] = -Mu[ i ];
        }
    }
    for( LL i = 1; i <= Maxn; ++i ) Sum[ i ] = Sum[ i - 1 ] + Mu[ i ];
    return;
}

void Work() {
    LL a, b, c;
    scanf( "%lld%lld%lld", &a, &b, &c );
    if( a > b ) swap( a, b );
    LL Ans = 0;
    for( LL x = 1, y; x <= a / c; x = y + 1 ) {
        y = min( ( a / c ) / ( ( a / c ) / x ), ( b / c ) / ( ( b / c ) / x ) );
        Ans += a / c / x * ( b / c / x ) * ( Sum[ y ] - Sum[ x - 1 ] );
    }
    printf( "%lld\n", Ans );
    return;
}

int main() {
    Init();
    LL t; scanf( "%lld", &t );
    for( ; t; --t ) Work();
    return 0;
}
```

