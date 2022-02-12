---
layout: post
title: Dancing Links
subtitle: 覆盖问题解决利器
categories: 图论
tags: [搜索,链表,Dancing Links]
---

冬令营突然发现的一个比较厉害的东西。

### Dancing Links

用于解决覆盖性问题的利器。

两个不同分支，一个精准覆盖问题，一个可重复覆盖问题。

先讲相对基础的精准覆盖问题。

### 精确覆盖问题

就是说全集由多个集合恰好覆盖，各个集合间无交。

这个问题我们先考虑一个具体一些的问题。

我们用下面的那个 [模板题](https://www.luogu.com.cn/problem/P4205) 为例。是不是看到就想到爆搜。

确实是爆搜，我们考虑怎么优化这个搜索。我们发现如果我们把不可能的情况删掉，就可以减少冗余的操作，就可以做到优化。

具体就是删除含有了已经被加入了的集合。这样可以非常优秀地剪枝。然后回溯的时候再把删除掉的重新恢复即可。这几个过程我们都可以一个十字链表维护。

具体十字链表怎么维护？或者说十字链表是个什么东西？实际上就是一个链表，其节点保存了其左、右、上、下的节点。我们可以把这个链表抽象到矩阵上面。每一行上下连通且每一列左右连通以此方便判断是否遍历完全。其他的插入删除与链表差不多，就是更新插入位置上下左右的节点的方位信息。

对此我们还可以再加入一个剪枝优化，我们每次选取被删除元素最少的那一列，这样可以使得接下来呢选择的那一列尽可能不与已经被删掉的列冲突，提高正确的可能性，以达到剪枝的目的。


模板题代码：
```cpp
#include<bits/stdc++.h>
#define ll long long
#define db double
#define filein(a) freopen(#a".in","r",stdin)
#define fileot(a) freopen(#a".out","w",stdout)
template<class T>
inline void read(T &s){
	s=0;char ch=getchar();bool f=0;
	while(ch<'0'||'9'<ch) {if(ch=='-') f=1;ch=getchar();}
	while('0'<=ch&&ch<='9') {s=s*10+(ch^48);ch=getchar();}
	if(ch=='.'){
		int p=10;ch=getchar();
		while('0'<=ch&&ch<='9') {s=s+(db)(ch^48)/p;p*=10;;ch=getchar();};
	}
	s=f?-s:s;
}
const int N=500+1;
const int MAXN=5500+1;
int n,m;
struct DLX{
	struct node{
		int x,y;
		int l,r,u,d;
	};
	int n,m,idx;
	node p[MAXN];
	int head[N],sz[N];
	inline void build(int _n,int _m){
		n=_n;m=_m;
		for(int i=0;i<=m;++i){
			p[i].l=i-1;p[i].r=i+1;
			p[i].u=p[i].d=i;
		}
		p[0].l=m;p[m].r=0;idx=m;
	}
	inline void ins(int x,int y){
		int i=++idx;
		p[i].x=x;p[i].y=y;
		p[i].u=y;p[i].d=p[y].d;
		++sz[y];
		p[p[y].d].u=i;p[y].d=i;
		if(!head[x]){
			p[i].l=p[i].r=head[x]=i;
		}else{
			p[i].l=head[x];p[i].r=p[head[x] ].r;
			p[p[head[x] ].r].l=i;p[head[x] ].r=i;
		}
	}
	inline void remove(int c){
		p[p[c].l].r=p[c].r;p[p[c].r].l=p[c].l;
		for(int i=p[c].d;i!=c;i=p[i].d){
			for(int j=p[i].r;j!=i;j=p[j].r){
				p[p[j].u].d=p[j].d;p[p[j].d].u=p[j].u;
				--sz[p[j].y];
			}
		}
	}
	inline void resume(int c){
		for(int i=p[c].u;i!=c;i=p[i].u){
			for(int j=p[i].l;j!=i;j=p[j].l){
				p[p[j].u].d=p[p[j].d].u=j;
				++sz[p[j].y];
			}
		}
		p[p[c].l].r=p[p[c].r].l=c;
	}
	int chs[MAXN];
	bool dance(int t){
		if(!p[0].r){
			for(int i=1;i<t;++i){
				printf("%d ",p[chs[i] ].x);
			}
			return 1;
		}
		int c=p[0].r;
		for(int i=p[0].r;i!=0;i=p[i].r){
			if(sz[c]>sz[i]){
				c=i;
			}
		}
		remove(c);
		for(int i=p[c].d;i!=c;i=p[i].d){
			chs[t]=i;
			for(int j=p[i].r;j!=i;j=p[j].r){
				remove(p[j].y);
			}
			if(dance(t+1) ) return 1;
			for(int j=p[i].l;j!=i;j=p[j].l){
				resume(p[j].y);
			}
		}
		resume(c);
		return 0;
	}
}s;
int main(){
	filein(a);fileot(a);
	read(n);read(m);
	s.build(n,m);
	for(int i=1;i<=n;++i){
		for(int j=1;j<=m;++j){
			int x;read(x);
			if(x) s.ins(i,j);
		}
	}
	if(!s.dance(1) ) printf("No Solution!\n");
	return 0;
}
```

[虽然是黄色但是比刚才那个题难的题](https://www.luogu.com.cn/problem/P1784)

我们发现对于具体问题抽象化，发现可以把十字链表转换一下，其行用来存储选择的方案，列用来存储限制。可以自己思考一下。

```cpp
#include<bits/stdc++.h>
#define ll long long
#define db double
#define filein(a) freopen(#a".in","r",stdin)
#define fileot(a) freopen(#a".out","w",stdout)
template<class T>
inline void read(T &s){
	s=0;char ch=getchar();bool f=0;
	while(ch<'0'||'9'<ch) {if(ch=='-') f=1;ch=getchar();}
	while('0'<=ch&&ch<='9') {s=s*10+(ch^48);ch=getchar();}
	if(ch=='.'){
		int p=10;ch=getchar();
		while('0'<=ch&&ch<='9') {s=s+(db)(ch^48)/p;p*=10;;ch=getchar();};
	}
	s=f?-s:s;
}
const int N=9+1;
int a[N][N];
struct DLX{
	struct node{
		int num,x,y,g;
		int d,u,l,r;
	};
	int idx;
	int n,m;
	node p[(729*4)+1];
	int head[729+1];
	int sz[324+1];
	inline void build(int _n,int _m){
		n=_n;m=_m;
		for(int i=0;i<=m;++i){
			p[i].l=i-1;p[i].r=i+1;
			p[i].u=p[i].d=i;
		}
		p[0].l=m;p[m].r=0;idx=m;
	}
	inline void ins(int x,int y){
		int i=++idx;
		p[i].x=x;p[i].y=y;++sz[y];
		p[i].u=y;p[i].d=p[y].d;
		p[p[y].d].u=i;p[y].d=i;
		if(!head[x]){
			head[x]=p[i].l=p[i].r=i;
		}else{
			p[i].l=head[x];p[i].r=p[head[x] ].r;
			p[p[head[x] ].r].l=i;p[head[x] ].r=i;
		}
	}
	inline void ins(node x){
		int line=(x.x-1)*81+(x.y-1)*9+x.num;
		ins(line,(x.x-1)*9+x.num);ins(line,81+(x.y-1)*9+x.num);
		ins(line,162+(x.g-1)*9+x.num);ins(line,243+(x.x-1)*9+x.y);
	}/*
	inline void dead_saying(int i){
		int x,y,z;
		x=((int)(p[i].x-1) )/81+1;
		y=((int)(p[i].x-1) )/9%9+1;
		z=((int)(p[i].x-1) )%9+1;
		int line=(x-1)*81+(y-1)*9+z;
		if(i<=m) printf("std ");
		printf("dead %d:(%d,%d) [%d][%d,%d]{%d}\n",i,x,y,z,p[i].x,p[i].y,line);
	}
	*/
	inline void remove(int c){
		p[p[c].r].l=p[c].l;p[p[c].l].r=p[c].r;
		//dead_saying(c);
		for(int i=p[c].d;i!=c;i=p[i].d){
			for(int j=p[i].r;j!=i;j=p[j].r){
				p[p[j].d].u=p[j].u;p[p[j].u].d=p[j].d;
				//dead_saying(j);
				--sz[p[j].y];
			}
		}
	}
	inline void resume(int c){
		for(int i=p[c].u;i!=c;i=p[i].u){
			for(int j=p[i].l;j!=i;j=p[j].l){
				p[p[j].d].u=p[p[j].u].d=j;
				++sz[p[j].y];
			}
		}
		p[p[c].l].r=p[p[c].r].l=c;
	}
	int chs[(729*4)+1];
	bool dance(int t){
		if(!p[0].r){
			for(int i=1;i<t;++i){
				int x,y,z;
				x=((int)(chs[i]-1) )/81+1;
				y=((int)(chs[i]-1) )/9%9+1;
				z=((int)(chs[i]-1) )%9+1;
				a[x][y]=z;
			}
			return 1;
		}
		int c=p[0].r;
		for(int i=p[0].r;i!=0;i=p[i].r){
			if(sz[c]>sz[i]){
				c=i;
			}
		}
		remove(c);
		for(int i=p[c].d;i!=c;i=p[i].d){
			chs[t]=p[i].x;
			for(int j=p[i].r;j!=i;j=p[j].r){
				//printf("%d\n",p[j].y);
				remove(p[j].y);
			}
			if(dance(t+1) ) return 1;
			for(int j=p[i].l;j!=i;j=p[j].l){
				resume(p[j].y);
			}
		}
		resume(c);
		return 0;
	}
}s;
int main(){
	//filein(a);fileot(a);
	s.build(729,324);
	for(int i=1;i<=9;++i){
		for(int j=1;j<=9;++j){
			read(a[i][j]);
			for(int num=1;num<=9;++num){
				if(a[i][j] and a[i][j]!=num) continue;
				int g=((i-1)/3)*3+(j-1)/3+1;
				s.ins((DLX::node){num,i,j,g});
			}
		}
	}
	s.dance(1);
	for(int i=1;i<=9;++i){
		for(int j=1;j<=9;++j){
			printf("%d ",a[i][j]);
		}putchar('\n');
	}
	return 0;
}
```

[情况多一点的模板题](https://www.luogu.com.cn/problem/P4205)

这个就自己练手吧~

### 可重复覆盖问题

可重复覆盖问题就在精准覆盖问题上改一下即可了。

我们删除的时候只封锁竖直方向上的链，于是就可以存在重复了。因为这样会有更大的搜素空间，所以还需要增加剪枝，如果当前深度加上未被封锁的竖直方向能被封锁的最小步数（就对于没封锁且没被标记的竖直方向向下然后每一层左右标记一下，看有多少个执行了这个操作即可）大于我们之前求出的最佳步数，就直接返回了。
