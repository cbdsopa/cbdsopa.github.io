---
layout: post
title: Dsu on tree
subtitle: 树上启发式合并，解决子树问题的神！
categories: 图论
tags: [Dsu on tree]
---

考模拟赛考到了线段树合并，于是学了树上启发式合并(?)

它相较于线段树合并的优点在于:
- 空间开销小
- 无线段树的常数
- 写法简单易理解

好那么开始进入正题。

---

首先说一下它的用途，可以用来解决一些子树问题。

具体的做法是先 dfs 预处理，然后像树剖那样剖出重儿子，可以剖出 dfs 序为之后加上一点常数优化（不需要优先遍历重儿子）。

然后我先给出核心函数的伪代码:

```cpp
void dfs(int u,int f,bool k){
	for(int i=head[u];~i;i=nxt[i]){
		if(to[i]==son[u]||to[i]==f) continue;
		dfs(to[i],u,0);
	}
	if(son[u]) dfs(son[u],u,1);
	for(int i=head[u];~i;i=nxt[i]){
		if(to[i]==son[u]||to[i]==f) continue;
		for(int j=dfn[to[i]];j<=dfn[to[i]]+sz[to[i]]-1;++j){
			Add(rk[j]);
		}
	}
	Add(u);
	ans[u]=getans();
	if(k==0){
		for(int i=dfn[u];i<=dfn[u]+sz[u]-1;++i){
			Sub(rk[i]);
		}
		clear();
	}
}
```
其中 Add 和 Sub 函数视具体题目而定。

在上面的代码中我们发现它会优先遍历一遍非重儿子，并且在遍历完其非重儿子的子树后将其对答案的贡献清空。这样做是为了保证各子树答案不会互相影响。

然后再遍历重儿子子树并计算出其答案，并且保留其对答案的贡献，因为接下来要计算当前子树的答案，把其他非重儿子的答案贡献再加回来，这里考虑用 dfs 序优化这个过程，最后不要忘了加上当前点的贡献。这个重儿子最后遍历的操作实际上是一个小优化，因为重儿子的子树大小最大，这样重儿子子树只遍历了一遍。最后再把需要清空的子树清空即可。

你会发现这个并不难理解。并且大部分情况下线段树合并能做它也能做（显然不包括和线段树分裂配套的那种）。

### 具体题目分析

[Lomsat gelral](https://www.luogu.com.cn/problem/CF600E)

Dsu on tree 模板题。

考虑用 cnt 数组存储颜色出现次数，只需要在 Add 中实现更新出现次数最大的编号和即可。

code：
```cpp
#include<bits/stdc++.h>
using namespace std;
#define file(a) freopen(#a".in","r",stdin),freopen(#a".out","w",stdout)
#define LL long long
#define N 100000+3
inline int read(){
	int s=0,f=1;char ch=getchar();
	while(ch<'0'||'9'<ch) {if(ch=='-') f=-1;ch=getchar();}
	while('0'<=ch&&ch<='9') {s=s*10+(ch^48);ch=getchar();}
	return s*f;
}
inline void write(LL x){
	if(x<0) x=-x,putchar('-');
	static char Wstack[20],top=0;
	do{Wstack[top]=x-x/10*10;x/=10;++top;}while(x);
	while(top){putchar(Wstack[--top]+'0');}
}
int cnt[N],n,col[N],mx; 
vector<int>head,to,nxt;
void join(int u,int v){
	nxt.push_back(head[u]);
	head[u]=to.size();
	to.push_back(v);
}
int son[N],sz[N],dfn[N],rk[N],idx;
void dfs1(int u,int f){
	dfn[u]=++idx;
	rk[idx]=u;
	sz[u]=1;
	int tmp=-1;
	for(int i=head[u];~i;i=nxt[i]){
		if(to[i]==f) continue;
		dfs1(to[i],u);
		sz[u]+=sz[to[i]];
		if(tmp<sz[to[i]]){
			tmp=sz[to[i]];
			son[u]=to[i];
		}
	}
}
LL sum,maxn,ans[N];
inline void Add(int u){
	++cnt[col[u]];
	if(cnt[col[u]]>maxn){
		maxn=cnt[col[u]];
		sum=col[u];
	}else if(cnt[col[u]]==maxn){
		sum+=col[u];
	}
}
inline void Sub(int u){
	--cnt[col[u]];
}
inline LL getans(){
	return sum;
}
void dfs(int u,int f,bool k){
	for(int i=head[u];~i;i=nxt[i]){
		if(to[i]==son[u]||to[i]==f) continue;
		dfs(to[i],u,0);
	}
	if(son[u]) dfs(son[u],u,1);
	for(int i=head[u];~i;i=nxt[i]){
		if(to[i]==son[u]||to[i]==f) continue;
		for(int j=dfn[to[i]];j<=dfn[to[i]]+sz[to[i]]-1;++j){
			Add(rk[j]);
		}
	}
	Add(u);
	ans[u]=getans();
	if(k==0){
		for(int i=dfn[u];i<=dfn[u]+sz[u]-1;++i){
			Sub(rk[i]);
		}
		sum=maxn=0;
	}
}
int main(){
	n=read(); 
	head.resize(n+3,-1);
	for(int i=1;i<=n;++i){
		col[i]=read();
	}
	for(int i=1;i<n;++i){
		int u=read(),v=read();
		join(u,v);join(v,u);
	}
	dfs1(1,1);
	dfs(1,1,1);
	for(int i=1;i<=n;++i){
        write(ans[i]);
        putchar('\n');
	}
	return 0;
}
```

如果想做相关题目可以去找线段树合并的题目(?)

之后又一次在学校无意中提及 $dsu$ 的常数，发现好像不单单是常数小，它本身的复杂度就要更优于 $log$。 