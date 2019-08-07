---
title: cm_可持久化trie树学习笔记 + BZOJ 3261/P4592 [TJOI2018]题解
tags: 
  - acm
  - 可持久化trie树
  - 学习笔记
categories:
  - cm
  - 数据结构
  - 可持久trie树
date: 2019-8-6
top: False
---

# cm_可持久化trie树学习笔记 + BZOJ 3261/P4592 [TJOI2018]题解

*主席树的题，虽然嘴巴ac了不少，但是还是得练啊*

*当然，每天除了做主席树，在别的方面也得看啊！*

刚好今天学习了fail树上的一些应用，既然是字符串，刚好最近在看可持久化的数据结构，不如就顺便看看可持久化的trie树吧。

顺便，安利一本书，**《算法竞赛进阶指南》**
<!-- more -->

## 问题描述

初始给你长度为$n$的一个序列$a$，有两种操作，一是在序列后在添加一个数$x$，二是让你在$[l,r]$区间选一个$p$，使得$a[p]\oplus a[p+1]\oplus ...a[r]\oplus x$的值最大.

## 前置技能

- trie树
- 可持久化数据结构思想

## 可持久化trie树

因为之前在练主席树，所以学习了一下发现，核心思想都差不多，就是**「每次操作只增加需要增加的节点，不变的节点仍然和上一次状态保持一致」**

然后...然后好像就没有然后了...

## 题目分析

这道题分三个部分，我们一点一点看

### 1. 问题转化

首先，先不要考虑关于区间的问题

利用异或的性质，我们可以将它转化为类似前缀和处理。我们先得到前缀异或和$s$数组。如果要求$a[p]\oplus a[p+1]\oplus ... a[n]\oplus x$，其实就是求$s[p-1]\oplus s[n]\oplus x$，其中$s[n]\oplus x$是固定值，这里用$val$代替。也就是我们要在数组$s$中找一个$s[p-1]$，使得$s[p-1]\oplus val$最大

如果没有区间的限制，只需要对$s$中每个数01比特建一课异或trie树，然后拿着val尽可能往反方向走就行。

### 2. 右边界限制

其实右区间限制也好说，我们只需要用第$r$个历史版本即可，因为此时右边界右边的数还没有插到trie树中。历史版本用可持久化trie树做就行。

### 3. 左边界限制

左边界限制，我们需要同时在维护一个T[x].latest，表示到达这个节点最近的一次是通过数组中的第几个节点，其实就是维护一下数组位置的$max$，然后递归查询的时候，准备递归的节点的latest，必须$\ge limit$（其中$limit$是左边界限制），否则就递归另一个孩子

## 代码

注意，转换成前缀和数组的问题后，我们的区间$[l,r]$就成了新的问题中的$[l-1,r-1]$

```c++
#include <iostream>
#include <stdio.h>
#include <stdio.h>
using namespace std;

const int maxn = 600020;
struct node{
	int lc,rc,latest;
}T[maxn*40];
int n,m,tot,root[maxn],arr[maxn],s[maxn];
void update(int y,int &x,int i,int k)
{
	x = ++tot;T[x] = T[y];
	if(k < 0){
		T[x].latest = i;
		return;
	}
	int c = (s[i] >> k) & 1;
	if(!c)update(T[y].lc,T[x].lc,i,k-1);
	else update(T[y].rc,T[x].rc,i,k-1);
	T[x].latest = max(T[T[x].lc].latest,T[T[x].rc].latest);
} 

int query(int x,int k,int val,int limit)
{
	if(k < 0)return s[T[x].latest] ^ val;
	int c = (val >> k) & 1;
	if(c){
		if(T[T[x].lc].latest >= limit)return query(T[x].lc,k-1,val,limit);
		else return query(T[x].rc,k-1,val,limit);
	}else{
		if(T[T[x].rc].latest >= limit)return query(T[x].rc,k-1,val,limit);
		else return query(T[x].lc,k-1,val,limit);
	}
}
char sss[5];
int main()
{
    scanf("%d%d",&n,&m);
    T[0].latest=-1;
    for(int i = 1;i<=n;i++){
    	scanf("%d",&arr[i]);
    	s[i] = s[i-1] ^ arr[i];
    	update(root[i-1],root[i],i,23);
	}
	int now_len = n;
	char cc;int x,ql,qr;
	for(int i = 1;i<=m;i++){
		scanf("%s",sss);
		if(sss[0] == 'A'){
			now_len ++;
			scanf("%d",&x);
			s[now_len] = s[now_len - 1] ^ x;
			update(root[now_len-1],root[now_len],now_len,23);
		}else{
			scanf("%d%d%d",&ql,&qr,&x);
			int val = s[now_len] ^ x;
			printf("%d\n",query(root[qr-1],23,val,ql-1));
		}
	}
    return 0;
}
```

当然，还有一种写法，是利用了前缀和相减的性质，通过计算个数，判断是否能往相反跳，过程中统计答案，下面是Oi-Wiki的代码

```c++
#include <algorithm>
#include <cstdio>
#include <cstring>
using namespace std;
const int maxn = 600010;
int n, q, a[maxn], s[maxn], l, r, x;
char op;
struct Trie {
  int cnt, rt[maxn], ch[maxn * 33][2], val[maxn * 33];
  void insert(int o, int lst, int v) {
    for (int i = 28; i >= 0; i--) {
      val[o] = val[lst] + 1;  //在原版本的基础上更新
      if ((v & (1 << i)) == 0) {
        if (!ch[o][0]) ch[o][0] = ++cnt;
        ch[o][1] = ch[lst][1];
        o = ch[o][0];
        lst = ch[lst][0];
      } else {
        if (!ch[o][1]) ch[o][1] = ++cnt;
        ch[o][0] = ch[lst][0];
        o = ch[o][1];
        lst = ch[lst][1];
      }
    }
    val[o] = val[lst] + 1;
    // printf("%d\n",o);
  }
  int query(int o1, int o2, int v) {
    int ret = 0;
    for (int i = 28; i >= 0; i--) {
      // printf("%d %d %d\n",o1,o2,val[o1]-val[o2]);
      int t = ((v & (1 << i)) ? 1 : 0);
      if (val[ch[o1][!t]] - val[ch[o2][!t]])
        ret += (1 << i), o1 = ch[o1][!t], o2 = ch[o2][!t];  //尽量向不同的地方跳
      else
        o1 = ch[o1][t], o2 = ch[o2][t];
    }
    return ret;
  }
} st;
int main() {
  scanf("%d%d", &n, &q);
  for (int i = 1; i <= n; i++) scanf("%d", a + i), s[i] = s[i - 1] ^ a[i];
  for (int i = 1; i <= n; i++)
    st.rt[i] = ++st.cnt, st.insert(st.rt[i], st.rt[i - 1], s[i]);
  while (q--) {
    scanf(" %c", &op);
    if (op == 'A') {
      n++;
      scanf("%d", a + n);
      s[n] = s[n - 1] ^ a[n];
      st.rt[n] = ++st.cnt;
      st.insert(st.rt[n], st.rt[n - 1], s[n]);
    }
    if (op == 'Q') {
      scanf("%d%d%d", &l, &r, &x);
      l--;
      r--;
      if (l == r && l == 0)
        printf("%d\n", s[n] ^ x);  //记得处理 l=r=1 的情况
      else
        printf("%d\n", st.query(st.rt[r], st.rt[max(l - 1, 0)], x ^ s[n]));
    }
  }
  return 0;
}
```

## P4592 [TJOI2018]

还有一个题是树上的查询，有两种查询方式，一种是在子树中选一个数与$x$异或最大，这个可以通过dfs序转成区间问题，还有一种路径查询的，思想和树上主席树一样

[原题链接](https://www.luogu.org/problem/P4592)

```c++
#include <iostream>
#include <stdio.h>
#include <stdio.h>
using namespace std;

const int maxn = 600020;
const int K = 31;
int fa[maxn][20],deep[maxn],vis[maxn];
int root_arr[maxn],root_tree[maxn];
struct node{
    int lc,rc,cnt,val;
}T[maxn * 32];
struct _edge{
    int to,next;
}edge[maxn];
int root[maxn],head[maxn],tot,cnt,m,n;
void addedge(int u,int v){
   edge[cnt].to = v;
   edge[cnt].next = head[u];
   head[u] = cnt ++; 
}
void init(int n){
    cnt = 0;
    for(int i = 0;i<=n;i++){
        head[i] = -1;
    }
}
int dsz;
int in[maxn],out[maxn],id[maxn],v[maxn];
int xxor[maxn];
int lca(int u,int v)
{
    if(deep[u] < deep[v])swap(u,v);
    int dis = deep[u] - deep[v];
    for(int i = 0;i<20;i++){
        if((1<<i)&dis)u = fa[u][i];
    }
    if(u == v)return u;
    for(int i = 19;i>=0;i--){
        if(fa[u][i] != fa[v][i]){
            u = fa[u][i];
            v = fa[v][i];            
        }
    }
    return fa[u][0];
}
void update(int y,int &x,int val,int k)
{
    // cout << y << endl;
    x = ++tot;T[x] = T[y];T[x].cnt ++;
    if(k < 0){
        T[x].val = val;
        return;
    }
    int c = (val >> k) & 1;
    if(!c){
        update(T[y].lc,T[x].lc,val,k-1);
    }else{
        update(T[y].rc,T[x].rc,val,k-1);
    }
}
void dfs(int now)
{
    vis[now] = true;
    in[now] = ++dsz; 
    id[dsz] = now;
    for(int i = 1;i<20;i++){
        if(deep[now] < (1<<i))break;
        fa[now][i] = fa[fa[now][i-1]][i-1];
    }
    for(int i = head[now];~i;i=edge[i].next){
        int to = edge[i].to;
        if(!vis[to]){
            fa[to][0] = now;
            deep[to] = deep[now] + 1;
            update(root_tree[now],root_tree[to],v[to],K);
            dfs(to);
        }
    }
    out[now] = dsz;
}
int query_arr(int y,int x,int val,int k,int ret=0)
{
    //cout << y << " " << x << " " << val << " " << k << endl;
    if(k < 0)return T[x].val ^ val;
    int c = (val >> k) & 1;
    if(c){
        int cnt = T[T[x].lc].cnt - T[T[y].lc].cnt;
        if(cnt > 0)return query_arr(T[y].lc,T[x].lc,val,k-1,ret+(1<<k));
        else return query_arr(T[y].rc,T[x].rc,val,k-1,ret);
    }else{
        int cnt = T[T[x].rc].cnt - T[T[y].rc].cnt;
        if(cnt > 0)return query_arr(T[y].rc,T[x].rc,val,k-1,ret+(1<<k));
        else return query_arr(T[y].lc,T[x].lc,val,k-1,ret);
    }
}
int query_path(int y,int x,int f,int ff,int val,int k,int ret=0)
{
    if(k < 0){return ret;}
    int c = (val >> k) & 1;
    if(c){
        int cnt = T[T[x].lc].cnt + T[T[y].lc].cnt - T[T[f].lc].cnt - T[T[ff].lc].cnt;
        //cout << k << " left " << cnt<<endl;
        if(cnt > 0)return query_path(T[y].lc,T[x].lc,T[f].lc,T[ff].lc,val,k-1,ret+(1<<k));
        else return query_path(T[y].rc,T[x].rc,T[f].rc,T[ff].rc,val,k-1,ret);
    }else{
        int cnt = T[T[x].rc].cnt + T[T[y].rc].cnt - T[T[f].rc].cnt - T[T[ff].rc].cnt;
        //cout << k << " right " << cnt << endl;
        if(cnt > 0)return query_path(T[y].rc,T[x].rc,T[f].rc,T[ff].rc,val,k-1,ret+(1<<k));
        else return query_path(T[y].lc,T[x].lc,T[f].lc,T[ff].lc,val,k-1,ret);
    }
}
void build(int &x,int k)
{
    cout << k << endl;
    x = ++tot;T[x].cnt = 0;
    if(k<0)return;
    build(T[x].lc,k-1);
    build(T[x].rc,k-1);
}
int main()
{
   scanf("%d%d",&n,&m); 
   init(n);
   for(int i = 1;i<=n;i++)scanf("%d",&v[i]);
   for(int i = 0;i<n-1;i++){
       int xx,yy;scanf("%d%d",&xx,&yy);
       addedge(xx,yy);addedge(yy,xx);
   }
   update(root_tree[0],root_tree[1],v[1],K);
   dfs(1);
   for(int i = 1;i<=dsz;i++){
       xxor[i] =  v[id[i]];
       update(root_arr[i-1],root_arr[i],xxor[i],K);
   }
   int op,qx,qy,qz;
   while(m--){
       scanf("%d",&op);
       if(op == 1){
          scanf("%d%d",&qx,&qy);
          printf("%d\n",query_arr(root_arr[in[qx] - 1] , root_arr[out[qx]],qy,K)); 
       }else{
           scanf("%d%d%d",&qx,&qy,&qz);
           int fx = lca(qx,qy);
           int ffx = fa[fx][0];
        //    cout << fx << " " << ffx << endl;
           printf("%d\n",query_path(root_tree[qx],root_tree[qy],root_tree[fx],root_tree[ffx],qz,K));
       }
   }
}
```
