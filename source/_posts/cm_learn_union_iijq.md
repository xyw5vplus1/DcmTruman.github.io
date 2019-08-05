---
title: cm_可持久化并查集学习笔记 && BZOJ 3657 题解
tags: 
  - acm
  - 并查集
  - 主席树
  - 可持久化线段树
  - 学习笔记
categories:
  - cm
  - 数据结构
  - 主席树
date: 2019-8-5
top: False
---
# cm_可持久化并查集学习笔记 && BZOJ 3657 题解

## 问题描述

给你$n$个集合$1,2,...n$，要求支持下面一些操作

- 合并$a$和$b$所在的集合
- 回到第$k$次操作之后的状态(查询算作操作）
- 询问$a$和$b$是否在同一集合
<!-- more -->
## 前置技能

- 并查集
- 主席树

## 可持久化并查集

简单来说，这道题就是，并查集的询问集合以及合并的操作，但是要求你能回退到k次操作状态之后。所以我们必须要记录历史版本信息。

首先，先考虑，并查集的`merge`和`find`操作，实际上是改变了`father`数组，但是如果每次改变，我们都重新拷贝一次数组的话，空间就很爆炸，那么我们可以用**可持久化线段树的思想**。数组中每次某个位置节点发生改变，就新生成一个根节点，增加 $log_{2}n$个节点。而在取得时候，我们也不会像原来那样直接取father数组里拿，而是指定一个表示当前状态的根节点，$log_{2}$ 的查询到叶子节点，这两步我写成了`getfa`和`setfa`函数，其中`update`和`query`基本就是**可持久化线段树**

```c++
int getfa(int x,int now)
{
    return query(x,1,n,now);
}

int setfa(int x,int pos,int val)
{
   update(x,tmp,1,n,pos,val);
   return tmp; 
}
```

当然，线段树中，除了叶子节点以外的节点，是不存储额外信息的

```c++
void update(int y,int &x,int l,int r,int pos,int val)
{
    x=++tot;T[x]=T[y];
    if(l == r){
        T[x].val = val;
        return ;
    }
    int mid = (l+r)>>1;
    if(pos<=mid)update(T[y].l,T[x].l,l,mid,pos,val);
    else update(T[y].r,T[x].r,mid+1,r,pos,val);
}
```

整体代码比较简单好理解，就是在`find`和`merge`中，将原来函数中**数组取值father**和**改变数组值**变成现在在线段树上操作`getfa`和`setfa`。基本可以当板子来用了。

代码如下

```c++
#include <iostream>
#include <math.h>
#include <string>
#include <stdio.h>
#include <string.h>
#include <algorithm>
#include <vector>
using namespace std;

#define ll long long
#define sc1(a) scanf("%lld",&a)
#define sc2(a,b) scanf("%lld %lld",&a,&b)
#define lson(x) x<<1
#define rson(x) x<<1|1
#define lowbit(x) (x&(-x))
#define fuck(x) cout<<"[Debug : "<<#x<<" "<<(x)<<']'<<endl

const int maxn = 2e5+7;
const int maxm = 2500010;
vector<int> vv;
int getid(int x){return (int)(lower_bound(vv.begin(),vv.end(),x)-vv.begin())+1;}

struct node{
    int l,r,val;
}T[maxn * 40];
int tot,m,n;
void build(int &x,int l,int r)
{
    x=++tot;T[x].val = 0;
    if(l == r){T[x].val = l;return;}
    int mid = (l+r)>>1;
    build(T[x].l,l,mid);
    build(T[x].r,mid+1,r);
}
void update(int y,int &x,int l,int r,int pos,int val)
{
    x=++tot;T[x]=T[y];
    if(l == r){
        T[x].val = val;
        return ;
    }
    int mid = (l+r)>>1;
    if(pos<=mid)update(T[y].l,T[x].l,l,mid,pos,val);
    else update(T[y].r,T[x].r,mid+1,r,pos,val);
}
int query(int x,int l,int r,int pos)
{
    if(l==r)return T[x].val;
    int mid = (l + r) >> 1;
    if(mid >= pos)return query(T[x].l,l,mid,pos);
    else return query(T[x].r,mid+1,r,pos);
}


int opt[maxn];
int aa,bb,kk;
int tmp;
int getfa(int x,int now)
{
    return query(x,1,n,now);
}

int setfa(int x,int pos,int val)
{
   update(x,tmp,1,n,pos,val);
   return tmp; 
}

int find(int &rt,int x)
{
    // fuck(x);
    int fx = getfa(rt,x);
    if(fx == x)return x;
    else{
        fx = find(rt,fx);
        // fuck(x);
        // fuck(fx);
        rt = setfa(rt,x,fx);
        return fx;
    }
}

void merge(int &rt,int x,int y)
{
    int fx = find(rt,x);
    int fy = find(rt,y);
    if(fx != fy){
        rt = setfa(rt,fx,fy);
    }
}
int main()
{
    scanf("%d%d",&n,&m);
    n ++;
    //for(int i = 1;i<=n;i++)scanf()
    build(opt[0],1,n);
    int lastans = 0;
    for(int i = 1;i<=m;i++)
    {
        opt[i] = opt[i-1];
        int oo;scanf("%d",&oo);
        if(oo == 1){
            scanf("%d%d",&aa,&bb);
            aa = aa ^ lastans;
            bb = bb ^ lastans;
            aa ++;bb ++;
            // fuck(aa);fuck(bb);
            merge(opt[i],aa,bb);
        }else if( oo == 2 ){
            scanf("%d",&kk);
            kk = kk ^ lastans;
            // fuck(kk);
            opt[i] = opt[kk];
        }else{
            scanf("%d%d",&aa,&bb);
            aa = aa ^ lastans;
            bb = bb ^ lastans;
            aa ++;
            bb ++;
            int fx = find(opt[i],aa);
            int fy = find(opt[i],bb);
            // fuck(aa);fuck(bb);
            // fuck(fx);fuck(fy);
            if(fx == fy)lastans = 1;
            else lastans = 0;
            printf("%d\n",lastans);
        }
    }

}
```
