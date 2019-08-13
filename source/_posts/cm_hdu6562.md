---
title: cm_HDU6562 题解
tags: 
  - 线段树
  - 题解
  - acm
categories:
  - cm
  - 数据结构
  - 线段树
date: 2019-08-13
top: False
---

# cm_HDU6562 题解

*其实这一篇我也想放在学习笔记那里，因为昨晚写这个题的时候我都不好意思说自己学过带lazy的线段树...*
<!-- more -->

## 题目描述

两种操作，

1. $w,l,r,d$，把区间里的每个数，前面加上一个d，后面加上一个d，比如原来是3，d是4，这个数之后就变成了434，注意，根据样例可以发现，前导0也要算的，比如前后加0，变成030，这时候在前后加4，就是40304，而不是4304
2. $ q,l,r$，询问区间和

## 分析

两头加一个d，我们先假设d一位数，（为什么要假设呢，因为我们延迟更新的话，向下更新的数可能是好几位），对sum的贡献就是

$$sum = sum \times 10 + (r - l + 1) \times d + 10 \times  d \times \sum_{i=l}^{r} 10^{len_{a_{i}}}$$

线段树的每个节点需要有$sum$，同时还需要维护一下$\sum_{i=l}^{r} 10^{len_{a_{i}}}$

当前节点的更新好说，问题是如果要带延迟，该怎么更新呢，我用来两个lazy标记，`lazyd`表示右边要拼的数，`lazym`表示左边要拼的数，那么我们延迟的思想就是，我们一次可以拼好几个数，也就没必要深入到底下一个一个修改，用到再push。

再加上上面的题目描述，所以我们还需要知道，当前拼的数是几位数，注意，这道题里，0是1位，00是2位，更新的时候也就更新多位数，`vald`表示后面拼的数，`valm`表示前面拼的数，`wei`表示要拼的数的位数

```c++
sum = sum * qpow(10,wei,mod) % mod;
sum = (sum + (r-l+1) * vald % mod) % mod;
//cout << "len10=" << len10 << " ";
sum = (sum + valm * len10 % mod * qpow(10,wei,mod) % mod)%mod;
//cout << "sum="<< sum << endl;
len10 = len10 * qpow(10,2ll * wei,mod)%mod;
lazyd = (lazyd * qpow(10,wei,mod) % mod + vald) % mod;
lazym = (valm * qpow(10,lazywei,mod) % mod + lazym) % mod;
lazywei = (lazywei + wei) % mod;
```



## 代码

```c++
#include <iostream>
#include <stdio.h>
#include <math.h>
using namespace std;

#define lson(x) x << 1
#define rson(x) x << 1 | 1
#define ll long long
const ll mod = 1e9+7;
const int maxn = 1e6+2;

ll qpow(ll a,ll b,ll m)
{
    ll res = 1;
    while(b){
       if(b & (ll)1)res = res * a % m;
       a = a * a % m;
       b >>= 1;
    }
    return res;
}
ll inv = qpow(10,mod-2,mod);
struct node{
    int l,r;
    ll len10,sum;
    ll lazyd,lazym,lazywei;
    void update(ll vald,ll valm,ll wei)
    {
        sum = sum * qpow(10,wei,mod) % mod;
        sum = (sum + (r-l+1) * vald % mod) % mod;
        //cout << "len10=" << len10 << " ";
        sum = (sum + valm * len10 % mod * qpow(10,wei,mod) % mod)%mod;
        //cout << "sum="<< sum << endl;
        len10 = len10 * qpow(10,2ll * wei,mod)%mod;
        lazyd = (lazyd * qpow(10,wei,mod) % mod + vald) % mod;
        lazym = (valm * qpow(10,lazywei,mod) % mod + lazym) % mod;
        lazywei = (lazywei + wei) % mod;
    }
}tree[maxn * 4];

void pushup(int x)
{
    tree[x].sum = (tree[lson(x)].sum + tree[rson(x)].sum) % mod;
    tree[x].len10 = (tree[lson(x)].len10 + tree[rson(x)].len10 ) %mod;
}

void pushdown(int x)
{
    ll la = tree[x].lazywei;
    if(la){
        tree[lson(x)].update(tree[x].lazyd,tree[x].lazym,tree[x].lazywei);
        tree[rson(x)].update(tree[x].lazyd,tree[x].lazym,tree[x].lazywei);
        tree[x].lazyd = tree[x].lazym = 0;
        tree[x].lazywei = 0;
    }
    
}

void build(int x,int l,int r)
{
    tree[x].l = l,tree[x].r = r;
    tree[x].sum = tree[x].lazyd = tree[x].lazym = tree[x].lazywei = 0;
    if(l == r){
        tree[x].len10 = 1;
        return;
    }
    int mid = (l + r)>>1;
    build(lson(x),l,mid);
    build(rson(x),mid+1,r);
    pushup(x);
}

void update(int x,int l,int r,ll val)
{
    //cout << l <<  " " << r<<endl;
    int L = tree[x].l,R = tree[x].r;
    if(l <= L && R <= r){
        tree[x].update(val,val,1);
        return;
    }
    pushdown(x);
    int mid = (L + R) >> 1;
    if(l <= mid)update(lson(x),l,r,val);
    if(mid <r)update(rson(x),l,r,val);
    pushup(x);
}

ll query(int x,int l,int r)
{
    int L = tree[x].l,R = tree[x].r;
    if(l <= L && R <= r){
        return tree[x]. sum;
    }
    pushdown(x);
    int mid = (L + R) >> 1;
    ll res = 0;
    if(l <= mid)res = (res + query(lson(x),l,r)) % mod;
    if(mid <r)res = (res + query(rson(x),l,r)) % mod;
    pushup(x);
    return res;
}
int n,m,ql,qr;
ll qk;
char ss[7];
int main()
{
    int T;scanf("%d",&T);
    for(int kase=1;kase<=T;kase++)
    {
        printf("Case %d:\n",kase);
        scanf("%d%d",&n,&m);
        build(1,1,n);
        while(m--){
            scanf("%s",ss);
            if(ss[0] == 'w'){
                scanf("%d%d%lld",&ql,&qr,&qk);
                update(1,ql,qr,qk);
            }else{
                scanf("%d%d",&ql,&qr);
                printf("%lld\n",query(1,ql,qr));
            }
        }
    }
}
```
