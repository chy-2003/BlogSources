# Splay教程

[TOC]

## 前言

Splay是名副其实的区间小能手。它会经常出现在一些有关区间的题上。而本蒟蒻只会Treap，感到分外难受，于是就有了这个教程。

## 引入

Splay首先是一颗二叉查找树。也就是说，对于任何一颗子树，左子树内所有的值均小于根，右子树内所有的值均大于根。也就是下图这样，数字代表值。

![1](https://github.com/chy-2003/PicS/raw/master/201903022129.png)

这样的话，中序遍历的结果就是从小到大的。

但是这棵树可能会退化成这样：

![2](https://github.com/chy-2003/PicS/raw/master/201903022148.png)

会使操作奇慢无比。所以我们考虑优化。

## 教程

### Rotate

Rotate操作能在不破坏二叉查找树的性质的情况下更改树的结构：

![3](https://github.com/chy-2003/PicS/raw/master/201903022156.png)

感受一下，如果我们用$Child[0]$表示左孩子，$Child[1]$表示右孩子，那么Rotate就可以这样写：

```C++
void Rotate( int C ) {
    int B = Father[ C ];
    int A = Father[ B ];
    int Tag = Child[ B ][ 1 ] == C;
    Child[ A ][ Child[ A ][ 1 ] == B ] = C;
    Father[ C ] = A;
    Child[ B ][ Tag ] = Child[ C ][ Tag ^ 1 ];
    Father[ Child[ C ][ Tag ^ 1 ] ] = B;
    Child[ C ][ Tag ^ 1 ] = B;
    Father[ B ] = C;
    return;
}
```

### Splay

Splay操作是核心。Splay( x, y )表示把$x$旋转到$y$的儿子，Splay( x, 0 )就是把$x$旋转到根。或许不难想到这样的操作：

```C++
while( Father[ x ] != y ) Rotate( x );
```

但是这样有一个问题：

对于下图

![4](https://github.com/chy-2003/PicS/raw/master/201903022210.png)

如果我们Splay( C, 0 )，按照上面的做法，就会变成这样：

![11](https://github.com/chy-2003/PicS/raw/master/201903022214.png)

我们发现链$A-B-C-y$依旧存在，只是变成了$C-A-B-y$。这样的话时间复杂度就无法保证了。

这种情况只有在C是B的左孩子，B是A的左孩子，或者C是B的右孩子，B是A的右孩子的情况下才会发生。这时，我们考虑先Rotate(B)，然后Rotate(C)。大家可以画图感受一下。

所以Splay可以这样写：

```C++
void Splay( int Index, int Goal ) {
    while( Father[ Index ] != Goal ) {
        int B = Father[ Index ]; 
        int A = Father[ B ];
        if( A != Goal ) 
            if( ( Child[ A ][ 0 ] == B ) ^ ( Child[ B ][ 0 ] == Index ) ) 
                Rotate( Index );
        	else
                Rotate( B );
        Rotate( Index );
    }
    return;
}
```

这样就好了啊！

### 一些其他操作：

插入、删除、查询数x的排名、查询排名第x的数、查询前驱、查询后缀等都与Treap相似，只要最后把目标线Splay到根就行，这里不再赘述。

### 区间翻转

下面通过[文艺平衡树](https://www.luogu.org/problemnew/show/P3391)一题来讲述如何翻转区间。

这一题中，关键字是位置。

假设我们要翻转$[l,r]$。我们首先将$l-1$旋转到根，再将$r+1$转到根的儿子，这样根的右孩子的左孩子就是$[l,r]$了。我们打上标记就可以了。

## 结语

先简略地介绍这么一点，剩下的大家可以慢慢体会。