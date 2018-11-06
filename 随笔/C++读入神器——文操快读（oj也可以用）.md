# C++读入神器——文操快读（oj也可以用）

当某天，本蒟蒻沉迷于卡常的时候：

![图](https://github.com/chy-2003/PicS/blob/master/1.png?raw=true)

我……

突然，YYKdalao说：用文操快读啊！

然后

![图2](https://github.com/chy-2003/PicS/blob/master/2.png?raw=true)

喔～目瞪口呆

不多说，上源码：

本来用的读入方式：

```c++
inline void Read( int &x ) {
	x = 0; char ch = getchar();
	for( ; ch < '0' || ch > '9'; ch = getchar() );
	for( ; ch >= '0' && ch <= '9'; ch = getchar() )
		x = x * 10 + ch - '0';
	return;
}
```

文操快读：

```C++
const int MAX_SIZE = 1 << 15;
char *l, *r, buf[ MAX_SIZE ];
inline int gc() {
	if( l == r )
		if( l == ( r = ( l = buf ) + fread( buf, 1, MAX_SIZE, stdin ) ) ) return -1;
	return *l++; 
}
inline void Read( int &x ) {
	char c = gc();
	for(; c < 48 || c > 57; c = gc() );
	for( x = 0; c > 47 && c < 58; c = gc() )
		x = ( x << 3 ) + ( x << 1 ) + ( c ^ 48 );
	return;
}
```

