---
title: cm_Fail树学习笔记 && CodeForces 547E 题解
tags: 
  - acm
  - 主席树
  - ac自动机
  - fail树
  - 学习笔记
categories:
  - cm
  - 字符串
  - ac自动机
date: 2019-8-6
top: False
---

# cm_Fail树学习笔记 && CodeForces 547E  题解（ac自动机fail树dfs序建可持久化线段树）



*每天都能学到新的知识，这种感觉还是很棒的！*

*说起来，今天学习fail树的应用，还是因为在做主席树专题的时候碰到了一个字符串的题*

*听说这一套流程已经成梗了，希望有空能把下图右半边也学会*

<!-- more -->

![%%%%%](https://raw.githubusercontent.com/DcmTruman/DcmTruman.github.io/images/img1_2019-08-06.jpeg)



## 问题描述

给你$n$个字符串，长度之和小于$2\mul 10^{5}$，现在有$5\mul 10^{5}$个询问$i,j,k$,问你，在第$i$个字符串到第$j$个字符串中，第$k$个字符串出现了多少次。

[原题链接](https://codeforces.com/problemset/problem/547/E)


## 前置技能

- 可持久化线段树
- ac自动机

## 分析

因为是**学习笔记**，所以我们先复习和学习一下关于ac自动机和fail树的知识点。

### ac自动机

熟悉kmp的同学都知道，在kmp中，存在一个next数组，`next[i]`表示的是到当前匹配串`i`的位置的字符串中，最长的**前缀与后缀相等的串**，kmp中之所以这么做，是因为一旦失配，可以减少一定的匹配次数。

同理ac自动机，**fail指针所指向的节点，表示的是走到当前节点所代表的字符串中，与其某个后缀相同的最长的前缀在trie树上的位置。**

说到这里就要说一句我觉得一些子串问题的本质思想--**子串是某个后缀的前缀，或者某个前缀的后缀**。虽然听起来有点废话，但对我这种菜鸡来说，一旦用前后缀的思想去想，有些问题就会有些眉目了。

### fail树

现在有个问题，如何统计模式串x在模式串y中出现的次数呢？要统计次数，说明我们要找到x在哪些地方是y的子串。按照上面的思想，我们需要找到，在y的所有后缀中，哪些的前缀是x在trie对应的位置？

暴力做法是，我们沿着y在trie中往下走，每到一个节点，我们就沿着fail一直跳到根，如果过程中走到x所在的位置，说明存在一个y的前缀，其后缀等于x。但是很暴力啊，每个位置都要沿着fail树跳到根啊！不优雅啊！

那么我们可以想到这么一件事，fail指针在每个节点下，是唯一的。我们是不是可以反向建一棵树？通过y走到当前节点now，沿着fail往上跳，也就是在树中从孩子往父亲跳。如果最终跳到了字符串x所代表的的节点，那么说明now在fail树中的节点必然是以x在fail树的节点为根的子树中。所以我们计算孩子对祖先的贡献，只需要沿着y走的时候，把每个节点在fail树中对应的节点+1，然后计算x对应节点子树的权值之和即可！

### 可持久化线段树

是不是听起来很酷？但是，有询问啊，针对不同区间的询问，我们加的权值也不一样，回答完后还要把他们恢复，避免互相干扰，怎么办？一种做法，离线。但是为了练习可持久化线段树，我偏不离线！

子树这种可以dfs序拆成区间的东西和**权值和**这种可减东西，让然可以用可持久化线段树维护啊！

### 题解

那么，这道题的题解就出来了，ac自动机fail树dfs序建可持久化线段树

1. 先把所有串放在ac自动机中，build一下，我们得到了fail指针。
2. 根据fail指针反向建树，我们得到了一颗fail树。
3. 遍历n个串。对于每个串，我们会树中修改`strlan(n)`个点，使得树中的点权值+1，为了快速获得一颗子树的权值和，我们先将树dfs，拆成一个个的区间，然后就是类似于线段树单点修改、区间查和的操作了。
4. 在这个基础上，每次线段树一个点发生变化，我们都新开一个根节点，新增$logn$个节点，基本思想和主席树类似，`strlan(n)`个点修改，就不断更新`root[now]`。
5. 然后查区间和，也是`root[r]`的树对应的区间和减去`root[l-1]`树对应的区间和，即可，这样，我们就能快速求得在插入i~j个字符串后，fail树上k节点的子树的权值和，即子串出现的次数。

## 代码

有点丑。期间还把`r`和`R`搞错了，一直在T第十个样例，我好蠢。

```c++
#include <iostream>
#include <math.h>
#include <string>
#include <stdio.h>
#include <string.h>
#include <queue>
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
const int maxn_tot=200000+7;
const int maxq = 200020;
int head[maxn_tot],cnt;
char s[maxn_tot*2];
int ss[maxq];
int ans[maxq];
int in[maxn_tot],out[maxn_tot];
struct _edge{
    int v;
    int next;
}edge[maxn_tot];
int n,m,tot;
void addedge(int u,int v)
{
    edge[cnt].v=v;
    edge[cnt].next=head[u];
    head[u]=cnt++;
}
struct _ac{
    struct state{
        int nxt[26];
        int fail,cnt;
    }stable[maxn_tot];
    int size;
    queue<int>q;
    void init(){
        while(!q.empty())q.pop();
        for(int i = 0;i<maxn_tot;i++){
            for(int j = 0;j<26;j++)stable[i].nxt[j]=0;
            stable[i].fail = stable[i].cnt = 0;
            //head[i] = 
        }
        size = 1;
    }
    void ins(char *s,int kk){
        // printf("nows=%s\n",s);
        int n = strlen(s);
        int now = 0;
        for(int i = 0;i<n;i++){
            char c = s[i];
            //cout << c << endl;
            if(!stable[now].nxt[c-'a'])stable[now].nxt[c-'a']=size++;
            now = stable[now].nxt[c-'a'];
        }
        ans[kk] = now;
    }
    void build(){
        stable[0].fail = -1;
        q.push(0);
        while(!q.empty()){
            int u = q.front();
            q.pop();
            for(int i = 0;i<26;i++){
                if(stable[u].nxt[i]){
                    if(u == 0){
                        stable[stable[u].nxt[i]].fail=0;
                        addedge(0,stable[u].nxt[i]);
                    }
                    else{
                        int v = stable[u].fail;
                        while(v!=-1){
                            if(stable[v].nxt[i]){
                                stable[stable[u].nxt[i]].fail=stable[v].nxt[i];
                                addedge(stable[v].nxt[i],stable[u].nxt[i]);
                                break;
                            }else{
                                v = stable[v].fail;
                            }
                        }
                        if(v == -1){
                            stable[stable[u].nxt[i]].fail = 0;
                            addedge(0,stable[u].nxt[i]);
                        }
                    }
                    q.push(stable[u].nxt[i]);
                }
            }
        }
    }
}ac;
 
void init()
{
    ac.init();
    for(int i = 0;i<maxn_tot;i++)head[i] = -1;
    cnt = 0;
    tot = 0;
}
int dsz = 0;
void dfs(int now,int f)
{
    // fuck(now);
    // fuck(f);
    in[now]=++dsz;
    for(int i = head[now];~i;i=edge[i].next){
        int to = edge[i].v;
        if(to != f)dfs(to,now);
    }
    out[now]=dsz;
}
struct node{
    int lc,rc,sum;
}T[maxq*40];
int root[maxn];
void build(int &x,int l,int r)
{
    // cout << x << " " << tot << endl;
    x=++tot;T[x].sum = 0;
    if(l == r)return;
    int mid = (l + r) >> 1;
    build(T[x].lc,l,mid);
    build(T[x].rc,mid+1,r);
}
void update(int y,int &x,int l,int r,int pos,int val)
{
    x = ++tot;T[x] = T[y];
    if(l == r){
        T[x].sum += val;
        return;
    }
    int mid = (l+r) >> 1;
    if(mid >= pos)update(T[y].lc,T[x].lc,l,mid,pos,val);
    else update(T[y].rc,T[x].rc,mid+1,r,pos,val);
    T[x].sum = T[T[x].lc].sum + T[T[x].rc].sum;
}
int query(int y,int x,int l,int r,int L,int R)
{
    if(L<=l && r<=R)return T[x].sum - T[y].sum;
    int mid = (l + r) >> 1;
    int ret = 0;
    if(L <= mid )ret += query(T[y].lc,T[x].lc,l,mid,L,R);
    if(mid < R)ret += query(T[y].rc,T[x].rc,mid+1,r,L,R);
    return ret;
}
void upup(int y,int &x,char *s)
{
    // printf("%s\n",s);
    x=++tot;T[x]=T[y];
    int ln = strlen(s);
    int now = 0;
    for(int i = 0;i<ln;i++){
        char c = s[i];
        int nxt = ac.stable[now].nxt[c-'a'];
        // fuck(nxt);
        // fuck(in[nxt]);
        update(x,x,1,dsz,in[nxt],1);
        now = nxt;
    }
}
int solve(int nowl,int nowr,int nowk)
{
    int ac_id = ans[nowk];
    int l = in[ac_id];
    int r = out[ac_id];
    return query(root[nowl-1],root[nowr],1,dsz,l,r);
}
int main()
{
    scanf("%d%d",&n,&m);
    init();
    for(int i = 0;i<maxn;i++)s[i] = '\0';
    
    for(int i = 1;i<=n;i++){
        ss[i] = ss[i-1] + strlen(s + ss[i-1]) + 1;
        scanf("%s",s+ss[i]);
        ac.ins(s + ss[i],i);
    }
    ac.build();
    dfs(0,-1);
    for(int i = 1;i<=n;i++){
        upup(root[i-1],root[i],s+ss[i]);
    }
    for(int i = 0;i<m;i++){
        int aa,bb,kk;scanf("%d%d%d",&aa,&bb,&kk);
        printf("%d\n",solve(aa,bb,kk));
    }
}
```




