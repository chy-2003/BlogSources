# 「TJOI / HEOI2016」字符串

<center>数据结构_SAM, 数据结构_线段树</center>

[题目链接](https://loj.ac/problem/2059)

## 问题分析

看见字符串，还涉及到子串和前缀的，当然就想到了SAM(~~其实我不会SA~~)。

首先我们把串反一下，就变成了求S[a..b]的所有子串与S[c..d]的最长后缀的长度。

我们把问题说成这样：求是S[a..b]子串的S[c..d]的最长后缀长度。于是我们就可以二分答案，然后判定是否在S[a..b]中出现就可以啦！

**考虑二分答案后如何判定**

假设我们现在判断Len是否可行。我们首先找到一个SAM上的状态，它的Right集合中含有d这个位置，并且Len介于这个状态的MinLen和MaxLen之间。那么只需要查询Right集合中有没有a+Len-1到b的元素即可。

具体细节是这样的：首先我们应该在建SAM的时候直接保存d最早出现的状态。这个状态就是Right集合中含有d，并且在Parent树上深度最深的节点。然后对于一个长度Len，我们在Parent树上倍增，找到[MinLen..MaxLen]包含Len的节点，判断的后缀就包含在这个状态里。同时，如果这个状态的Right集合中有[a+Len-1..b]的元素，那么说明这个串也存在于[a..b]中，否则就没有。

所以我们需要线段树合并求Right集合。然后就做完了！

时间复杂度$O(n\log^2n)$。

## 参考程序

~~我不知道为什么写的这么翔~~

```C++
#include <bits/stdc++.h>
//#ifndef DEBUG
//	#define DEBUG
//#endif
using namespace std;

const int Maxn = 100010;
const int MaxLog = 20;
int n, m;
char Ch[ Maxn ];
int Index[ Maxn ];

namespace Seg { //线段树
	struct segmentTree {
		segmentTree *LeftChild, *RightChild;
		int Num;
		void Init() {
			Num = 0;
			LeftChild = RightChild = NULL;
		}
	};
	segmentTree *SegmentTree[ Maxn << 1 ];

	void Init() {
		for( int i = 0; i < Maxn << 1; ++i )
			SegmentTree[ i ] = NULL;
		return;
	}

	segmentTree *Insert( segmentTree *Index, int Left, int Right, int Key ) {
		if( Index == NULL ) {
			Index = new segmentTree;
			Index -> Init();
		}
		++Index -> Num;
		if( Left == Right ) return Index;
		int Mid = ( Left + Right ) >> 1;
		if( Key <= Mid ) Index -> LeftChild = Insert( Index -> LeftChild, Left, Mid, Key );
		if( Key > Mid ) Index -> RightChild = Insert( Index -> RightChild, Mid + 1, Right, Key );
		return Index;
	}

	void Insert( int Pos, int Key ) {
#ifdef DEBUG
		printf( "Inform(Seg) : Add [ %d ] -> [ %d ]\n", Key, Pos );
#endif
		SegmentTree[ Pos ] = Insert( SegmentTree[ Pos ], 1, n, Key );
		return;
	}

	segmentTree *Merge( segmentTree *x, segmentTree *y, int Left, int Right ) {
		if( x == NULL && y == NULL ) return x;
		if( x != NULL && y == NULL ) return x;
		segmentTree *z = new segmentTree;//注意要新建节点
		if( x == NULL && y != NULL ) {
			*z = *y;
			return z;
		}
		*z = *x;
		z -> Num += y -> Num;
		if( Left == Right ) return z;
		int Mid = ( Left + Right ) >> 1;
		z -> LeftChild = Merge( z -> LeftChild, y -> LeftChild, Left, Mid );
		z -> RightChild = Merge( z -> RightChild, y -> RightChild, Mid + 1, Right );
		return z;
	}//线段树合并求Right集合

	void Play( segmentTree *Index, int Left, int Right ) {
		if( Index == NULL ) return;
		if( Index -> Num == 0 ) return;
		if( Left == Right ) {
			printf( "%d ", Left );
			return;
		}
		int Mid = ( Left + Right ) >> 1;
		Play( Index -> LeftChild, Left, Mid );
		Play( Index -> RightChild, Mid + 1, Right );
		return;
	}//Debug用

	void Merge( int x, int y ) {
#ifdef DEBUG
		printf( "Inform(Seg) : Merge %d <-- %d\n", x, y );
#endif
		SegmentTree[ x ] = Merge( SegmentTree[ x ], SegmentTree[ y ], 1, n );
#ifdef DEBUG
		printf( "  Right : " ); Play( SegmentTree[ x ], 1, n ); printf( "\n" );
#endif
		return;
	}//这是一个接口

	int Count( segmentTree *Index, int Left, int Right, int L, int R ) {
		if( Index == NULL ) return 0;
		if( L <= Left && Right <= R ) return Index -> Num;
		int Mid = ( Left + Right ) >> 1;
		int Ans = 0;
		if( L <= Mid ) Ans += Count( Index -> LeftChild, Left, Mid, L, R );
		if( R > Mid ) Ans += Count( Index -> RightChild, Mid + 1, Right, L, R );
		return Ans;
	}//求一段区间内的元素个数，用于判断区间内是否存在元素
} //Seg


namespace SAM {
	int Now, Used = 0, Last;
	struct suffixAutomaton {
		int Parent, Child[ 26 ], Len, MinLen;
	};
	suffixAutomaton SuffixAutomaton[ Maxn << 1 ];

	void Init() {
		Used = 0;
		memset( SuffixAutomaton, 0, sizeof( SuffixAutomaton ) );
		++Used; Last = Used;
		return;
	}

	void Insert( int t ) {
		Now = ++Used;
		SuffixAutomaton[ Now ].Len = SuffixAutomaton[ Last ].Len + 1;
		int p = Last;
		for( ; p && !SuffixAutomaton[ p ].Child[ t ]; p = SuffixAutomaton[ p ].Parent )
			SuffixAutomaton[ p ].Child[ t ] = Now;
		Last = Now;
		if( !p ) {
			SuffixAutomaton[ Now ].Parent = 1;
			return;
		}
		int q = SuffixAutomaton[ p ].Child[ t ];
		if( SuffixAutomaton[ p ].Len + 1 == SuffixAutomaton[ q ].Len ) {
			SuffixAutomaton[ Now ].Parent = q;
			return;
		}
		int Clone = ++Used;
		SuffixAutomaton[ Clone ].Len = SuffixAutomaton[ p ].Len + 1;
		SuffixAutomaton[ Clone ].Parent = SuffixAutomaton[ q ].Parent;
		for( int i = 0; i < 26; ++i )
			SuffixAutomaton[ Clone ].Child[ i ] = SuffixAutomaton[ q ].Child[ i ];
		for( ; p && SuffixAutomaton[ p ].Child[ t ] == q; p = SuffixAutomaton[ p ].Parent )
			SuffixAutomaton[ p ].Child[ t ] = Clone;
		SuffixAutomaton[ Now ].Parent = SuffixAutomaton[ q ].Parent = Clone;
		return;
	}
} //SAM

namespace Tree {//维护Parent树
	struct edge {
		int To, Next;
	};
	edge Edge[ Maxn << 2 ];
	int Start[ Maxn << 1 ], Used = 0;
	int Deep[ Maxn << 1 ], D[ Maxn << 1 ][ MaxLog ];//倍增需要的数组

	inline void AddEdge( int x, int y ) {
#ifdef DEBUG
		printf( "Inform(Tree) : Link %d <---> %d\n", x, y );
#endif
		Edge[ ++Used ] = ( edge ) { y, Start[ x ] };
		Start[ x ] = Used;
		return;
	}

	void Build( int Index, int Father ) {//建树，顺便求出MinLen
		D[ Index ][ 0 ] = Father;
		Deep[ Index ] = Deep[ Father ] + 1;
		if( Index > 1 )
			SAM::SuffixAutomaton[ Index ].MinLen = SAM::SuffixAutomaton[ Father ].Len + 1;
		for( int i = 1; i < MaxLog; ++i ) 
			D[ Index ][ i ] = D[ D[ Index ][ i - 1 ] ][ i - 1 ];
		for( int t = Start[ Index ]; t; t = Edge[ t ].Next ) {
			int v = Edge[ t ].To;
			if( v == Father ) continue;
			Build( v, Index );
		}
		return;
	}

	void Init() {
		for( int i = 2; i <= SAM::Used; ++i ) {
			AddEdge( SAM::SuffixAutomaton[ i ].Parent, i );
			AddEdge( i, SAM::SuffixAutomaton[ i ].Parent );
		}
		Build( 1, 1 );
		return;
	}

	void Union( int Index, int Father ) {
		for( int t = Start[ Index ]; t; t = Edge[ t ].Next ) {
			int v = Edge[ t ].To;
			if( v == Father) continue;
			Union( v, Index );
			Seg::Merge( Index, v );
		}
		return;
	}//求Right集合

	void Union() {
		Union( 1, 1 );
		return;
	}//这是一个接口

	void Play( int Index, int Father ) {
		printf( "Index = %d, Father = %d\n", Index, Father );
		printf( "  Min = %d, Max = %d\n", SAM::SuffixAutomaton[ Index ].MinLen, SAM::SuffixAutomaton[ Index ].Len );
		printf( "  Right : " );
		Seg::Play( Seg::SegmentTree[ Index ], 1, n );
		printf( "\n" );
		for( int t = Start[ Index ]; t; t = Edge[ t ].Next ) {
			int v = Edge[ t ].To;
			if( v == Father ) continue;
			Play( v, Index );
		}
		return;
	}//Debug用
} //Tree

void Debug() {
	printf( "Total Used Point in SAM : %d\n", SAM::Used );
	Tree::Play( 1, 0 );
	printf( "Start : " );
	for( int i = 1; i <= n; ++i ) printf( "%d ", Index[ i ] );
	printf( "\n\n\n" );
	system( "pause" );
	return;
}

void Read() {
	scanf( "%d%d", &n, &m );
	scanf( "%s", Ch + 1 );
	for( int i = 1; i + i <= n; ++i ) 
		swap( Ch[ i ], Ch[ n - i + 1 ] );
#ifdef DEBUG
	printf( "%d %d\n", n, m );
	printf( "%s\n", Ch + 1 );
	system( "pause" );
#endif
	return;
}

void Build() {
	Seg::Init();
	SAM::Init();
	for( int i = 1; i <= n; ++i ) {
		SAM::Insert( Ch[ i ] - 'a' );
		Index[ i ] = SAM::Now;//存下Right集合包含位置i并且在Parent树中深度最深的点
		Seg::Insert( SAM::Now, i );//插入位置i
	}
	Tree::Init();
#ifdef DEBUG
	Debug();
#endif
	Tree::Union();
#ifdef DEBUG
	Debug();
#endif
	return;
}

bool Check( int Len, int Index, int Left, int Right ) {
#ifdef DEBUG
	printf( "Inform(Check) : Len = %d, Left = %d, Right = %d, Index = %d\n", Len, Left, Right, Index );
	printf( "Inform(Check) : Min = %d, Max = %d\n", SAM::SuffixAutomaton[ Index ].MinLen, SAM::SuffixAutomaton[ Index ].Len );
#endif
	if( !Len ) return true;
	if( Len > SAM::SuffixAutomaton[ Index ].Len ) return false;
	if( SAM::SuffixAutomaton[ Index ].MinLen <= Len && Len <= SAM::SuffixAutomaton[ Index ].Len ) {
#ifdef DEBUG
		printf( "Count = %d\n", Seg::Count( Seg::SegmentTree[ Index ], 1, n, Left, Right ) );
#endif
		return Seg::Count( Seg::SegmentTree[ Index ], 1, n, Left, Right );
	}
	for( int i = MaxLog - 1; i >= 0; --i ) //倍增
		if( SAM::SuffixAutomaton[ Tree::D[ Index ][ i ] ].MinLen > Len )
			Index = Tree::D[ Index ][ i ];
	Index = Tree::D[ Index ][ 0 ];
#ifdef DEBUG
	printf( "Inform(Check) : Min = %d, Max = %d\n", SAM::SuffixAutomaton[ Index ].MinLen, SAM::SuffixAutomaton[ Index ].Len );
	printf( "Count = %d\n", Seg::Count( Seg::SegmentTree[ Index ], 1, n, Left, Right ) );
#endif
	return Seg::Count( Seg::SegmentTree[ Index ], 1, n, Left, Right );
}

void Solve() {
	for( int i = 1; i <= m; ++i ) {
		int a, b, c, d;
		scanf( "%d%d%d%d", &a, &b, &c, &d );
		a = n - a + 1; b = n - b + 1; c = n - c + 1; d = n - d + 1;
		swap( a, b ); swap( c, d );
		int Left = 0, Right = d - c + 1;
		while( Left < Right ) {//二分答案
			int Mid = ( Left + Right + 1 ) >> 1;
			if( Check( Mid, Index[ d ], a + Mid - 1, b ) ) Left = Mid; else Right = Mid - 1;
		}
		printf( "%d\n", Left );
	}
	return;
}

int main() {
	Read();
	Build();
	Solve();
	return 0;
}
```

