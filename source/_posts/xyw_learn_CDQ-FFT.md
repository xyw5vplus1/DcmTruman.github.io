---
title: 分治FFT学习笔记
tags: 
  - CDQ分治
  - xyw
  - FFT
categories:
  - xyw
  - 学习笔记
date: 2019-8-16
---

# 分治FFT #

算是因为兴趣学了一个黑科技吧，主要是突然最近对**CDQ分治**特别感兴趣，刚好分治FFT是CDQ分治的思想与FFT/NTT 结合的例子，FFT和NTT相对来说自己已经明白原理了并运用了，感觉学习分治FFT能更快地运用CDQ分治的思想，也能更快加深自己对CDQ分治的理解。

<!-- more -->

不知道有没有机会遇到非模板题，感觉思想还是蛮有趣的，感觉如果针对性地出题的话就很卡没有学过的人。



对ACM和OI中的“时间”一词越来越着迷，就像主席树把**选择一个区间转化为了选择两个历史版本之间的信息**，

把**对区间的操作转化为了时间维度上的操作**，CDQ分治也有类似的感觉，**相对靠后的位置的值会受相对靠前的位置的值的影响，而相对靠前的则不会受相对靠后的位置的影响**，那么我们可以先计算出左半段的值，然后为右半段的加上左半段的值对右半段产生的贡献，最后再计算出右半段的最终值。

那么这样看来，CDQ分治的运用条件在于需要有一个**类似时间维度**的东西，先发生的会影响后发生的，后发生的不会对先发生的值产生影响，那么在允许离线的情况下，我们可以考虑运用CDQ分治。

分治FFT作为一个工具，一般都是用来加速计算递推式的，那么显然是一个离线问题，

那么分治FFT的运用场景主要是什么呢？

下面进入正题

分治FFT主要是用来解决形如$f(i)=\sum_{j=1}^{j≤i} f(i-j)\times g(j)=\sum_{j=0}^{j<i} f(i)\times g(i-j)$的式子的，

暴力的做法复杂度是O(n^2)，如果做N次FFT的复杂度甚至达到了O(n^2logn)，还不如暴力，

所以我们这里考虑使用CDQ分治的思想，转化为时间维度上的分治，假设我们现在需要计算出整个[L,R]区间上的值，那么我们把[L,R]拆分成两个区间[L,mid]和[mid+1,R]，在计算完[L,mid]之后，我们去考虑**“贡献”**——左半段区间对右半段区间产生的贡献，最后在计算[mid+1,R]的最终值。

那么问题的核心就在于如何**计算出[L,mid]对[mid+1,R]的贡献**，

那么我们可以这么想，对于[mid+1,R]中的每个x，都有$f(x)=\sum_{j=L}^{j<=mid} f(j)\times g(x-j)$成立，

那么因为我们先计算左半边，在考虑左半边的贡献的时候，右半边的值还没有被计算，所以此时我们增大求和符号的上限。即$f(x)=\sum_{j=L}^{j<=x} f(j)\times g(x-j)$成立，此时f的下标和g的下标之和已经是定值了，我们只需要减小一定的大小就可以构造出用来卷积的多项式了，这里我们减小f的下标，我们令$j-L=t$，则有$f(x)=\sum_{t=0}^{t<=x-L} f(t+L)\times g(x-L-t)$成立，其中$mid+1<=x<=R$，这就是一个标准的卷积形式，我们令$a[i]=f[i+L],b[i]=g[i]$，那么式子就变成了$f(x)=C(x-L)=\sum_{i=0}^{i<=x-L} a(i)\times b(x-L-i)$，于是我们就可以愉快地FFT/NTT了，最后对每一项$x$把$C[x-L]$加上再递归调用右半边就OK了。

分治的层数是O(logn)的，而每次计算的FFT/NTT的复杂度是O(nlogn)的，总的复杂度是O(nlog^2n)的。

下面附上主题代码

```c++
void get_rev(int n)  // 蝴蝶变换
{
    int t=0;  while ((1<<t)<n) t++;  // 找到比n不小的最小的2的幂次
    rev[0]=0;
    for(int i=1;i<n;i++) rev[i]=(rev[i>>1]>>1)|((i&1)<<(t-1));
}
void NTT(int a[],int n,int k)
{
    for(int i=0;i<=n-1;i++)
        if(i<rev[i]) swap(a[i],a[rev[i]]);
    for(int m=2;m<=n;m<<=1)
    {
        int wn=quick_pow(G,(mod-1)/m);
        if(k==-1) wn=quick_pow(wn,mod-2);
        for(int i=0;i<=n-1;i+=m){
            int w=1,t0,t1;
            for(int j=0;j<=m/2-1;j++,w=1LL*w*wn%mod) {
                t0=a[i+j];t1=1LL*w*a[i+j+m/2]%mod;
                a[i+j]=(t0+t1)%mod;
                a[i+j+m/2]=((t0-t1)%mod+mod)%mod;
            }
        }
    }
    if(k==-1){
        int t=quick_pow(n,mod-2);
        for(int i=0;i<=n-1;i++) a[i]=1LL*a[i]*t%mod;
    }
}
void cdq(int L,int R)
{
    if (L==R) return ;
    int mid=(L+R)>>1;
    cdq(L,mid);
    int n=R-L+1; int len=1;
    while (len<2*n) len<<=1;
    get_rev(len);
    memset(a,0,sizeof(int)*len); memset(b,0,sizeof(int)*len);
    for (int i=L;i<=mid;i++) a[i-L]=f[i];
    for (int i=0;i<n;i++) b[i]=g[i];
    NTT(a,len,1); NTT(b,len,1);
    for (int i=0;i<len;i++) a[i]=1LL*a[i]*b[i]%mod;
    NTT(a,len,-1);
    for (int i=mid+1;i<=R;i++) f[i]=(f[i]+a[i-L])%mod;
    cdq(mid+1,R);
}
```

