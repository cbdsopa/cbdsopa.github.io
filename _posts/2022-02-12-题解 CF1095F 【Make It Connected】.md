---
layout: post
title: 题解 CF1095F 【Make It Connected】
subtitle: 
categories: 题解
tags: [最小生成树]
---

### 题意简述

$n$( $1≤n≤2×10^5$ )个点，每个点 $i$ 有一个点权 $a_i$ ( $1≤a_i≤2×10^{12}$ )，将两个点 $i$,$j$ 直接相连的花费是两个点的点权和 $a_i+a_j$，并且对于特别的$m$( $1≤m≤2×10^5$ )条边 $u_i$ , $v_i$ 可以通过花费 $w$ 点费用连接，求使得所有点互相连通的最小费用。

我们可以从数据范围看出需要注意的事项：

分析1.从$n$和$m$的范围可以看出我们最终的复杂度是带 ```log``` 的。

分析2.从$a_i$，很显然需要 ```long long```。

接下来就从分析1导入正题。

### Solution

看到关键词"连边","最小费用","边权"。

想到什么了？这不是我们可爱的最小生成树吗！

然后上来敲了一手 Kruskal[(不会最小生成树可以点这里)](https://oi-wiki.org/graph/mst/#kruskal)，然后越打越不对劲——是不是忘了点东西——还可以用 $a_i+a_j$ 直接连边。

好，于是就用 $O(n^2)$ 来加边....不对啊，这复杂度直接爆了。

真的 $n^2$ 条边都要用吗？似乎不是的。

对于一个图最少需要连 $n-1$ 条边来使图连通，**我们只需要以权值最小的点作为核心，他连出来的那一组边是最小的，且是能使图连通的，我们只需找到权值最小的点，把它与其他点相连的费用计算出来并进行 Kruskal。所以总共只需 $O(n+m)$ 加边即可**

之后用最小生成树算法就可以了，复杂度约为 $O((n+m)log(n+m))$。

### 代码

那么代码如下:
```cpp
#include<bits/stdc++.h>
using namespace std;
#define file(a) freopen(#a".in","r",stdin),freopen(#a".out","w",stdout);
int n,m,fa[200010],cnt,deals[200010];
long long v[200010],ans,minn=1000000000010,mid;
struct edge
{
	int u,v;
	long long w;
}q[500010];//记得开到两倍n
bool cmp(edge a,edge b)
{
	return a.w<b.w;
}
int find(int x)
{
	return x==fa[x]?x:fa[x]=find(fa[x]); 
}
int main()
{
	scanf("%d%d",&n,&m);
	for(int i=1;i<=n;++i)
	{
		scanf("%lld",&v[i]);
		fa[i]=i;
		if(v[i]<minn)
		{
			minn=v[i];
			mid=i;
		}
	}
	for(int i=1;i<=n;++i)
	{
		if(i==mid) continue;
		q[i].u=mid;q[i].v=i;
		q[i].w=v[mid]+v[i];
	}
	for(int i=n+1;i<=n+m;++i)
	{
		scanf("%d%d%lld",&q[i].u,&q[i].v,&q[i].w);
	}
	sort(q+1,q+1+n+m,cmp);
	for(int i=1;i<=n+m;++i)
	{
		int x=find(q[i].u),y=find(q[i].v);
		if(x!=y)
		{
			fa[y]=x;
			ans+=q[i].w;
		}
	}
	printf("%lld",ans);
	return 0;
}

```
