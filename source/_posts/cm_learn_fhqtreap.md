---
title: cm_无旋treap学习笔记&&HDU6162题解（2017多校）
tags: 
  - acm
  - treap
  - 数据结构
  - 平衡树
  - 学习笔记
categories:
  - cm
  - 数据结构
  - 平衡树
date: 2019-8-9
top: False
---

# cm_无旋treap学习笔记&&HDU6162题解（2017多校）


老实讲，关于平衡树的题，我并没有怎么做过，去年为了LCT而学了splay，但也只是做了版子题，一些区间操作、第k问题balabala，我基本上是能用 线段树/主席树 搞就不用别的。但为了练一些**树套树**的题，以及为了在数据结构上学的更深入一些，我决定开始最近研究一下平衡树！

<!-- more -->

## 二叉查找树(BST)

所谓二叉查找树就是在二叉树的基础上，满足这么一个性质：对于树上的每个节点，其左子树的任一节点的权值均小于当前节点，右子树任一节点的权值均大于当前点

BST的形态是可以多种多样的，最坏的时候也】可以退化到一个链，但无论怎么样，其中序遍历的结果始终都是相同的。在一些区间问题上，平衡树的做法中常常以数组位置为BST的关键值，无论怎么操作，只要满足BST性质，都是可以还原原来的数组序列的。

## BST的旋转

这个还是有必要记一下的，之前因为不怎么写，老是记不住。

我们可以通过一些操作改变树的结构，同时仍然保证BST的性质。接下来我们讨论的均是孩子和爸爸互换位置的情况

右旋 ： 假设x是y的左儿子，根据性质，x的值是小于y和y的右子树的，x当爸爸，那么y和y的右子树都成为了x的右儿子，那么原来x的右儿子该去哪里呢？x原来是y的左儿子，现在变成y的父亲了，y的左儿子就空出来了，把x的右儿子和右子树整体接到y的左儿子即可，依然满足性质。
左旋 ： 类似的思想。

## 平衡树

刚才说了，BST最坏情况下退化成链，查找什么的也就成了$O(n)$了，这样一点也不优雅，那么为了让这棵树看起来更像棵树，高度尽可能接近$logn$，我们就需要时刻改变树的结构，来保持”平衡“，同时还要依然保证二叉查找树的性质。常见的平衡树有splay、treap、替罪羊书、红黑树等，他们保证”平衡“的手段各不相同，今天说的treap，是一种利用随机权值来达到期望$logn$复杂度的数据结构。

首先说一下堆。


## 堆

二叉堆在结构上看起来和树很相似。常常用来维护最值。大根堆中，任何一个孩子的权值都比父亲小，小根堆中，任何一个孩子的权值都比父亲大。插入的时候，直接放到堆最后一个位置，然后$logn$的复杂度不断进行父亲和儿子的权值比较及替换，以满足具体的性质。

取最值的时候，O(1)时间取堆顶。



## treap

treap，顾名思义，tree + heap的一种数据结构，每个点有两个权值`val`和`key`，其中`val`满足二叉平衡树的性质，`key`满足小根堆的性质。

简单说就是，我们现在有一颗BST，插入值的时候，按照BST的性质插入，同时我们给这个节点附一个随机的`key`，用来满足小根堆的性质，如果儿子的`key`小于父亲，就进行旋转操作改变结构，将小的放在上面。

因为是随机值，所以我们的树的结构不会因为诡异的插入数据或者超级超级差的人品而退化成链，出题人想卡都比较难，复杂度期望是$logn$的，我也不太会证，大概理解一下就行。



## fhqtreap 无旋treap

看名字大概能猜出来，无旋treap是一种不用旋转的treap。这时候有人说了，你放屁！不通过旋转该别树的结构，怎么既维护二叉查找树的性质，又维护小根堆的性质？！且听我慢慢说。

无旋treap有两个核心操作——**分裂**和**合并**

### 分裂

假设你已经有一颗fhqtreap了，先不管怎么来的，分裂要干的事就是，给定一个value，把你当前的树拆成两颗不同x和y，其中树x的值均小于等于value，树y的值均大于value。

怎么实现的，通过递归和引用即可，非常好写。我们的函数给四个参数，根据val把以root为根的树分裂给x和y

我们首先看一下当前根节点的权值，如果小于val，就把当前根节点和左子树全部接给x，然后把root的右子树分裂给x的右子树和y。大于val同理。

这种分法，可以保证分裂出来的两个数仍然满足BST的性质，树x的值均小于等于value，树y的值均大于value。且第二权值仍然满足小根堆的性质（如果分裂前的树就满足的话）。


```c++
void Split(int &root,int &x,int &y,int val)
{
		if(!root){x=y=0;return;}
		if(tree[root].val<=val){x=root;Split(rson(root),rson(x),y,val);}
		else {y=root;Split(lson(root),x,lson(y),val);}
		Pushup(root);
}
```

### 合并

假设你有了两颗fhqtreap，一颗小于等于val，一颗大于val，怎么合并才能不破坏两种性质呢？

我们比较x和y根节点的key值（key是维护小根堆性质用到的值），如果x的小于y，则y出现在x节点的下面，保证了小根堆得性质。由于树y的每个节点的val值均大于x，此时只需要合并x的右子树和树y即可，保证了新的合并的树的BST性质。大于同理。

```c++
void Merge(int &root,int x,int y)
{
		if(!x || !y){root=x+y;return;}
		if(tree[x].key < tree[y].key){root=x;Merge(rson(root),rson(x),y);}
		else {root=y;Merge(lson(root),x,lson(y));}
		Pushup(root);
}
```

通过上面两个操作，我们就可以改变树的形态来满足性质，而又不使用旋转。同时还能花式解决许多类型的问题，比如把某个值域范围的数提取成一颗树，或者某个区间的数提取成一棵树。


### 插入

新开一个节点z，假设要插入的值为value，z的val为value，key随机生成，根据value把树分为x和y，合并z和x至x，在合并x和y至root即可

```c++
void Erase(int &root,ll val)
{
	int x=0,y=0,z=0,tmp=0;
	Split(root,x,y,val);
	Split(x,x,z,val-1ll);
	Merge(z,lson(z),rson(z));
	Merge(x,x,z);
	Merge(root,x,y);
}
```

### 删除

假设删除值为value的一个点，我们通过分裂操作，把全是value的点放到一棵树里，合并其左右孩子节点（即删掉根节点）,然后在合并回去

```c++
void Erase(int &root,ll val)
{
	int x=0,y=0,z=0,tmp=0;
	Split(root,x,y,val);
	Split(x,x,z,val-1ll);
	Merge(z,lson(z),rson(z));
	Merge(x,x,z);
	Merge(root,x,y);
}
```

无旋treap就说这么多，代码也是比较好写的，前驱后继k大啥的和别的平衡树没太大区别，听说可持久treap大都用这种无旋的方式来写，改天我也会专门写一下可持久化的fhqtreap学习笔记。

## HDU6162题解

给你一棵树，$n$个点，$m$个询问，$1\leq n,m \leq 10^{5}$每个点权值$1\leq c_{i}\leq 10^{9}$,问你$s$到$t$的路径上，权值在$l$到$r$之间的权值之和是多少

主席树在线可以搞，但这里为了练习平衡树，就离线了。利用容斥可以很快得到答案，需要四个关键点，s,t,lca(s,t)和lca(s,t)的父亲

假设`mp[x][y]`表示从根节点到x节点，权值小于等于y的值的总和，那么从根节点到x节点，区间在`[l,r]`的总和是`mp[x][r]-mp[x][l-1]`，从x到y，区间在`[l,r]`的总和是`mp[x][r]-mp[x][l-1]+mp[y][r]-mp[y][l-1]-(mp[lca(x,y)][r]-mp[lca(x,y)][l-1])-(mp[fa[lca(x,y)]][r]-mp[fa[lca(x,y)]][l-1])`

离线做，在关键点上记录要查询哪些东西。dfs这棵树，每次dfs进入这个节点，就把这个点的权值加入平衡树，如果当前是关键点，有一个查询`ql`在这里，去平衡树里，根据`ql`分裂成x树和y树，其中x树的权值和`sum`就是查询的答案（根到当前点权值小于`ql`的值得总和），`sum`通过`Pushup`函数来维护。

离开这个点的时候，再把它删除掉。

一开始我专门用了个map来记录，超内存了......其实不用，我们通过type标记一下加还是减，在dfs过程中把贡献加进ans中即可。

多组用例，记得清空~

```c++
#include <iostream>
#include <stdlib.h>
#include <string>
#include <string.h>
#include <stdio.h>
#include <stack>
#include <algorithm>
#include <stdio.h>
#include <queue>
#include <math.h>
#include <set>
#include <map>
#include <vector>
using namespace std;
#define ll long long 
#define lson(x) T[(x)].lc
#define rson(x) T[(x)].rc

const int maxn = 100020;

struct node{
	int lc,rc,key,size;
	ll val,sum;
}T[maxn];
int rt,tot;
void Pushup(int x){
	T[x].size = T[lson(x)].size + T[rson(x)].size + 1;
	T[x].sum = T[lson(x)].sum + T[rson(x)].sum + T[x].val;
}

void Split(int root,int &x,int &y,ll val)
{
	if(!root){x=y=0;return;}
	if(T[root].val <= val){x=root;Split(rson(root),rson(x),y,val);}
	else {y=root;Split(lson(root),x,lson(y),val);}
	Pushup(root);
}

void Merge(int &root,int x,int y)
{
	if(!x || !y){root=x+y;return;}
	if(T[x].key < T[y].key){root=x;Merge(rson(root),rson(x),y);}
	else {root=y;Merge(lson(root),x,lson(y));}
	Pushup(root);
} 
void Insert(int &root,ll val)
{
	int x=0,y=0,z=++tot;
	T[z].val=val;T[z].key=rand();T[z].sum=val;T[z].size=1;
	T[z].lc = T[z].rc = 0;
	Split(root,x,y,val);
	Merge(x,x,z);
	Merge(root,x,y);
}
void Erase(int &root,ll val)
{
	int x=0,y=0,z=0,tmp=0;
	Split(root,x,y,val);
	Split(x,x,z,val-1ll);
	Merge(z,lson(z),rson(z));
	Merge(x,x,z);
	Merge(root,x,y);
}
int Knum(int root,int k)
{
	if(root == 0)return 0;
	if(T[lson(root)].size == k-1)return T[root].val;
	if(T[lson(root)].size >= k)return Knum(lson(root),k);
	else return Knum(rson(root),k-T[lson(root)].size - 1);
}
ll cc[maxn];
int head[maxn],cnt;
int n,m;
struct _edge{
	int to,next;
}edge[2*maxn];
struct qst{
	int id;
	ll ql,type;
};
vector<qst>G[maxn];
void addedge(int u,int v){
	edge[cnt].to = v;
	edge[cnt].next = head[u];
	head[u] = cnt ++;
}
int fa[maxn][21],deep[maxn],vis[maxn];
void dfs1(int now)
{
	vis[now] = true;
	for(int i = 1;i<=19;i++){
		fa[now][i] = fa[fa[now][i-1]][i-1];
	}
	for(int i = head[now];~i;i=edge[i].next){
		int to = edge[i].to;
		if(!vis[to]){
			fa[to][0] = now;
			deep[to] = deep[now] + 1;
			dfs1(to);
		}
	}	
}
int lca(int u,int v)
{
	if(deep[u] < deep[v])swap(u,v);
	int dis = deep[u] - deep[v];
	for(int i = 19;i>=0;i--){
		if((1 << i) & dis)u=fa[u][i];
	}
	if(u==v)return u;
	for(int i = 19;i>=0;i--){
		if(fa[u][i] != fa[v][i]){
			u = fa[u][i];
			v= fa[v][i];
		}
	}
	return fa[u][0];
}
ll cal(int &root,ll val)
{
	int x=0,y=0;
	Split(root,x,y,val);
	ll ret = T[x].sum;
	Merge(root,x,y);
	return ret;
}
ll ans[maxn];
void dfs2(int now,int f)
{
	//cout << now << endl;
	Insert(rt,cc[now]);
	int sz = G[now].size();
	for(int i = 0;i<sz;i++){
		int id = G[now][i].id;
		ans[id] += cal(rt, G[now][i].ql)*G[now][i].type;
	}
	for(int i = head[now];~i;i=edge[i].next)	
	{
		int to = edge[i].to;
		if(to != f){
			dfs2(to,now);
		}
	}
	Erase(rt,cc[now]);
}
void init(int n){
	cnt = tot = rt = 0;
	for(int i = 0;i<=n;i++){
		G[i].clear();
		deep[i]=0;
		head[i] = -1;
		vis[i] = false;
		ans[i]=0;
		for(int j=1;j<=19;j++)fa[i][j]=0;
	}
}
int main()
{
	while(~scanf("%d%d",&n,&m))
	{
		init(n+5);
		for(int i = 1;i<=n;i++)scanf("%lld",&cc[i]);
		for(int i = 0;i<n-1;i++){
			int aa,bb;scanf("%d%d",&aa,&bb);
			addedge(aa,bb);
			addedge(bb,aa);
		}
		deep[1] = 1;
		dfs1(1);
		//cout << "fuck" << endl;
		for(int i=0;i<m;i++){
			int u,v;	
			ll ql,qr;
			scanf("%d%d%lld%lld",&u,&v,&ql,&qr);
			int fx = lca(u,v);
			int fxx = fa[fx][0];
			G[u].push_back((qst){i,ql-1,-1});
			G[u].push_back((qst){i,qr,1});
			G[v].push_back((qst){i,ql-1,-1});
			G[v].push_back((qst){i,qr,1});
			G[fx].push_back((qst){i,ql-1,1});
			G[fx].push_back((qst){i,qr,-1});
			G[fxx].push_back((qst){i,ql-1,1});
			G[fxx].push_back((qst){i,qr,-1});
		}
		dfs2(1,0);	
		for(int i = 0;i<m;i++){
			if(i)printf(" ");
			printf("%lld",ans[i]);
		}	
		printf("\n");
		
	}
}
```
