---
layout: post
title: Primal_Dual原始对偶
subtitle: 类似Johnson
categories: 图论
tags: [网络流,费用流]
---

不是费用流都需要用 SPFA 吗。

众所周知，SPFA 去世了，然后网络流显然有负边。于是我们可以像 Johnson 全源最短路一样，给边加上势能，具体实现看我之前的 [博客](https://cbdsopa.github.io/%E5%9B%BE%E8%AE%BA/2022/02/12/Johnson%E5%85%A8%E6%BA%90%E6%9C%80%E7%9F%AD%E8%B7%AF.html) 啦。

然后对于每一次跑 Dijkstra ，然后得到最短路，把势能要再加上这个最短路，可以证明这样操作一次图上不会再有负边。

也正因如此我们不能用 $dinic$ ，我们不保证在走了多条增广路后仍然边权非负，所以我们可以记录最短路的路径，然后每次增广一条，这样就能保证正确。这个似乎是 KM算法吗？反正都是增广一条，没有学过KM，所以不敢下定论。

### code:
```cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
#define file(a) freopen(#a".in","r",stdin),freopen(#a".out","w",stdout)
inline int read(){
	int s=0,f=1;char ch=getchar();
	while(ch<'0'||'9'<ch) {if(ch=='-') f=-1;ch=getchar();}
	while('0'<=ch&&ch<='9') {s=s*10+(ch^48);ch=getchar();}
	return s*f;
}
const int N=5e3+3;
const int INF=0x3f3f3f3f;
int n,m,S,T;
struct Edge{
	int v,w,c;
};
vector<int>head,nxt;
vector<Edge>to;
inline void join(int u,int v,int w,int c){
	nxt.push_back(head[u]);
	head[u]=to.size();
	to.push_back({v,w,c});
}
queue<int>Q;
int h[N];
bool inq[N];
inline void spfa(){
	for(int i=1;i<=n;++i) h[i]=INF,inq[i]=0;
	h[S]=0;Q.push(S);inq[S]=1;h[S]=0;
	while(!Q.empty() ){
		int u=Q.front();Q.pop();inq[u]=0;
		for(int i=head[u];~i;i=nxt[i]){
			int v=to[i].v,c=to[i].c,w=to[i].w;
			if(w and h[v]>h[u]+c){
				h[v]=h[u]+c;
				if(!inq[v]){
					inq[v]=1;
					Q.push(v);
				}
			}
		}
	}
}
int dis[N];
struct node{
	int d,x;
	inline friend bool operator <(node x,node y){
		return x.d<y.d;
	}
};
int cur[N];bool vis[N];
struct way{
	int v,id;
}e[N];
priority_queue<node>q;
inline bool dijkstra(){
	while(!q.empty() ) q.pop();
	for(int i=1;i<=n;++i) dis[i]=INF,vis[i]=0;
	dis[S]=0;q.push({0,S});
	while(!q.empty() ){
		int u=q.top().x;q.pop();
		if(vis[u]) continue;vis[u]=1;
		for(int i=head[u];~i;i=nxt[i]){
			int v=to[i].v,w=to[i].w,c=to[i].c+h[u]-h[v];
			if(w and dis[v]>dis[u]+c){
				e[v].v=u;
				e[v].id=i;
				dis[v]=dis[u]+c;
				if(!vis[v]){
					q.push({-dis[v],v});
				}
			}
		}
	}
	return dis[T]!=INF;
}
int minc,maxf;
inline void PD(){
	while(dijkstra() ){
		for(int i=S;i<=T;++i) h[i]+=dis[i];
		ll minf=INF;
		for(int i=T;i!=S;i=e[i].v){
			minf=min(minf,to[e[i].id].w);
		}
		for(int i=T;i!=S;i=e[i].v){
			to[e[i].id].w-=minf;
			to[e[i].id^1].w+=minf;
            minc+=minf*to[e[i].id].c;
		}
		maxf+=minf;
	}
}
int main(){
	//file(a);
	//freopen("a.in","r",stdin);
	n=read();m=read();S=read();T=read();
	head.resize(n+1,-1);
	for(int i=1;i<=m;++i){
		int u=read(),v=read(),w=read(),c=read();
		join(u,v,w,c);join(v,u,0,-c);
	}
	spfa();
	PD();
	printf("%d %d\n",maxf,minc);
	return 0;
}
```
