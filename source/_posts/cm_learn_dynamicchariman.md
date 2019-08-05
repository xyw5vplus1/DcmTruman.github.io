---
title: cm_动态主席树学习笔记 & ZOJ 2112题解
tags: 
  - acm
  - 主席树
  - 学习笔记
categories:
  - cm
  - 数据结构
  - 主席树
date: 2019-8-5
top: False
---
# cm_动态主席树学习笔记 & ZOJ 2112题解


## 问题描述

给你一个程度为$n$的序列$a_{1},a_{2}…a_{n}$，$m$个询问，有两种操作

1. 查询区间第k大
2. 把某个位置的值改成某个值

$0<n\leq 50000,0<m\leq 10000,a_{i}\leq 1000000000$
<!-- more -->
## 前置技能

- 静态主席树查区间第k大
- 树状数组



## 主席树

如果知道学过主席树，想必「区间查询第k大」大家并不陌生，对于静态区间第k大的问题，我们可以按照下标依次从小到大建立**可持久化的权值线段树**，线段树底层的点，表示的是这个数在区间[1-i]出现的次数，属于一种前缀。由于「次数」这种东西是可减的，所以如果我们想知道[l-r]的权值分布情况，只需要用**r**节点的线段树减去**l-1**节点的线段树即可

 ## 动态主席树

然后考虑，修个一个节点，会产生什么影响呢？

由于，主席树是统计前缀信息，每个新的树都要依赖前一颗，每当某个位置$i$的权值的数量被修改，那么$[i,n]$对应的所有树都要被修改，但时间上不允许我们这么搞，所以要想个更优雅的办法，单点修改，区间查询，有没有什么想法？

首先我们对于原数组建一个传统线段树，然后，为了维护修改信息我们可以再建一个树状数组，数组的每个值代表的一颗权值线段树，每当一个点被修改，我只新增$log_{2}n$颗树，每颗树值新增$log_{2}n$个节点。查询的时候，只需要先在原主席树查到当前节点的权值sum，然后在加上修改后的影响。那么这个影响怎么查呢，我根据我的代码来讲。



首先根据原数组建主席树

```c++
for(int i = 1;i<=n;i++){
  update(root[i-1],root[i],1,sz,getid(arr[i]),1);
} 
```



`c[i]`数组记录的是发生修改后的权值线段树的根节点，如果某个点$i$数值发生变化，我们**不会暴力的修改$i-n$的每颗权值线段树，而是利用类似树状数组的思想，只修改部分权值线段树**

```c++
void update_a(int i,int pos,int x){
    while(i <= n){
        update(c[i],c[i],1,sz,pos,x);
        i += lowbit(i);
    }
}
```

查询的时候，先把改变的值对应的一些线段树的根节点找到，每次递归计算sum的时候，在原数列的值上加上修改后的影响,具体操作看下方完整代码

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

const int maxn = 6e4+7;
const int maxm = 2500010;
vector<int> vv;
int getid(int x){return (int)(lower_bound(vv.begin(),vv.end(),x)-vv.begin())+1;}
struct node{
    int l,r,sum;
}T[maxm];
int m,tot,n,sz;
int root[maxn];
int arr[maxn];
int c[maxn],a[maxn];
void build(int &x,int l,int r)
{
    x=++tot;
    T[x].sum = 0;
    if(l == r)return;
    int mid=(l+r)>>1;
    build(T[x].l,l,mid);
    build(T[x].r,mid+1,r);
}
void update(int y,int &x,int l,int r,int pos,int vl)
{
    x=++tot;T[x]=T[y];
    if(l == r){
        T[x].sum += vl;
        return;
    }
    int mid = (l+r)>>1;
    if(mid >= pos)update(T[y].l,T[x].l,l,mid,pos,vl);
    else update(T[y].r,T[x].r,mid+1,r,pos,vl);
    T[x].sum = T[T[x].l].sum + T[T[x].r].sum;
}
void init_ta(){
    for(int i = 0;i<=n;i++)c[i] = root[0];
}
void update_a(int i,int pos,int x){
    while(i <= n){
        update(c[i],c[i],1,sz,pos,x);
        i += lowbit(i);
    }
}
int aa[maxn],cnt;
int bb[maxn],cnt2;
void getsum(int x,int y)
{
    cnt = 0;
    cnt2 = 0;
    while(x > 0){
        aa[cnt] = c[x];
        cnt ++;
        x -= lowbit(x);
    }
    while(y > 0){
        bb[cnt2] = c[y];
        cnt2 ++;
        y -= lowbit(y);
    }
}
int query(int y,int x,int l,int r,int k)
{
    if(l == r)return l;
    int mid = (l + r) >> 1;
    int tmp = T[T[x].l].sum - T[T[y].l].sum;
    for(int i = 0;i<cnt;i++){
        tmp -= T[T[aa[i]].l].sum;
    }
    for(int i = 0;i<cnt2;i++){
        tmp += T[T[bb[i]].l].sum;
    }
    if(tmp >= k){
        for(int i = 0;i<cnt;i++){
            aa[i] = T[aa[i]].l;
        }
        for(int i = 0;i<cnt2;i++){
            bb[i] = T[bb[i]].l;
        }
        return query(T[y].l,T[x].l,l,mid,k);
    }
    else{
        for(int i = 0;i<cnt;i++){
            aa[i] = T[aa[i]].r;
        }
        for(int i = 0;i<cnt2;i++){
            bb[i] = T[bb[i]].r;
        }
        return query(T[y].r,T[x].r,mid+1,r,k-tmp);
    }
}
struct qs{
    int type;
    int i,j,k;
}qs[maxn];
int main()
{
    int T;scanf("%d",&T);
    while(T--)
    {
        tot = 0;
        vv.clear();
        scanf("%d%d",&n,&m);
        for(int i = 1;i<=n;i++){
            scanf("%d",&arr[i]);
            vv.push_back(arr[i]);
        } 
        for(int i = 0;i<m;i++){
            char c;cin >> c;
            if(c == 'Q'){
                scanf("%d%d%d",&qs[i].i,&qs[i].j,&qs[i].k);
                qs[i].type = 0;
            }else{
                scanf("%d%d",&qs[i].i,&qs[i].k);
                vv.push_back(qs[i].k);
                qs[i].type = 1;
            }
        }
        sort(vv.begin(),vv.end());
        vv.erase(unique(vv.begin(),vv.end()),vv.end());
        sz = vv.size();
        build(root[0],1,sz);
        for(int i = 1;i<=n;i++){
            update(root[i-1],root[i],1,sz,getid(arr[i]),1);
        } 
        init_ta();
        for(int i = 0;i<m;i++){
            if(!qs[i].type){
                getsum(qs[i].i-1,qs[i].j);
                printf("%d\n",vv[query(root[qs[i].i-1],root[qs[i].j],1,sz,qs[i].k) - 1]);
            }
            else{
                update_a(qs[i].i,getid(arr[qs[i].i]),-1);
                arr[qs[i].i] = qs[i].k;
                update_a(qs[i].i,getid(arr[qs[i].i]),1);
            }
        }
    }
}
```

---

对了，这道题要注意一下范围，因为要提前存好所有询问在离散化，所以同时也要考虑询问时那些修改的值的数量。


