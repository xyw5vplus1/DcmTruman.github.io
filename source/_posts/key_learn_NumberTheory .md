---
title: key_数论学习笔记
tags: 
  - 数论
  - 学习笔记
categories:
  - key
  - 数学
  - 数论
date: 2019-8-14
---

<!-- more -->

### 数论学习笔记

#### 快速乘

```c++
inline ll quick_mul(ll x,ll y ll mod)
{
    return (x*y-(ll)((long double)x/mod*y)*mod+mod)%mod;
}
```

------

#### 素数筛

**Eratosthenes筛**

```c++
int v[MAX_N];
void primes(int n)
{
    memset(v,0,sizeof(v));
    for(int i=2;i<=n;i++)
    {
        if(v[i]) continue;
        for(int j=i;j<=m/i;j++) v[j*i]=1;
    }
}
```



**线性筛**

```c++
int v[MAX_N],prime[MAX_N];
int cnt;//质数个数
void primes(int n)
{
    memset(v,0.sizeof(v));//最小质因子
    cnt=0;
    for(int i=2;i<=n;i++)
    {
        if(v[i]==0) v[i]=i,prime[++cnt]=i;
        for(int j=1;prime[j]<=v[i]&&j<=cnt;j++)
        {
            if(prime[j]>n/i) break;
            v[prime[j]*i]=prime[j];
        }
    }
}
```

------

#### 质因数分解

```c++
int p[MAX_N],c[MAX_N];//因子 每个因子个数
int cnt;//因子个数
void divide(int n)
{
    cnt=0;
    for(int i=2;i<=sqrt(n);i++)
        if(n%i==0)
        {
            p[++cnt]=i,c[cnt]=0;
            while(n%i==0) n=n/i,[cnt]++;
        }
    if(n>1) p[++cnt]=n,c[cnt]=1;
}
```

------

#### 约数

**求约数试除法**

```c++
int factor[MAX_N],cnt=0;
for(int i=1;i<=sqrt(n);i++)
{
    if(n%i==0)
    {
        factor[++cnt]=i;
        if(i!=n/i)factor[++cnt]=n/i;
    }
}
```

**求约数倍数法**

```c++
vector<int> factor[MAX_N];
for(int i=1;i<=n;i++)
    for(int j=1;j<=n/i;j++)
        factor[i*j].push_back(i);
```

**最小公倍数**

辗转相除法 欧几里得算法

```c++
int gcd(int a,int b)
{
    return b?gcd(b,a%b):a;
}
```

------

#### 欧拉函数

```c++
int phi(int n)
{
    int ans=n;
    for(int i=2;i<=sqrt(n);i++)
        if(n%i==0) 
        {
            ans=ans/i*(i-1);
    		while(n%i==0)n/=i;
        }
    if(n>1)ans=ans/n*(n-1);
    return ans;
}
```

**定理**

- $\forall n>1,1$ ~ $n$ 中与 $n$ 互质的数的和为 $n*\varphi(n)/2$
- $\sum_{d|n}\varphi(d)=n$

------

#### 欧拉定理

若正整数 $a,n$ 互质,则 $a^{\varphi(n)}\equiv1(mod\ n)$

**费马小定理：**若 $p$ 是质数，则对于任意整数 $a$ ，有 $a^p\equiv a(mod\ p)$

**推论：**

若正整数 $a,n$ 互质,则对于任意正整数 $b$，有 $a^b\equiv a^{b\ mod\ \varphi(n)}(mod\ n)$

若正整数 $a,n$ 不一定互质且 $b>\varphi(n)$，有 $a^b\equiv a^{b\ mod\ \varphi(n)+\varphi(n)}(mod\ n)$

> *原因：*
> *不管 $a,n$ 是否互质，$a^bmod\ n$ 均有* ***指数循环节* ** *，循环节长度为 $\varphi(n)$ 的约数。*
> *互质时循环从头开始*.
> *不互质时不循环的长度不超过 $\varphi(n)$.*

若正整数 $a,n$ 互质,则满足 $a^{x}\equiv1(mod\ n)$ 的 $x$ 的最小值有 $x|\varphi(n)$

------

#### 扩展欧几里得

$ax+by=gcd(a,b)$ ,求 $(x,y)$

```c++
int exgcd(int a,int b,int &x,int &y)
{
    if(b==0){x=1,y=0;return a;}
    int d=exgcd(b,a%b,x,y);
    int z=x;x=y;y=z-a/b*y;
    return d;
}
```

**求$ax+by=c$**

当 $gcd(a,b)|c$ 时有解.

**通解：** $x=\frac{c}{d}x_0+k\frac{b}{d},y=\frac{c}{d}y_0+k\frac{a}{d}$ , $k$ 取遍整数集合，$d=gcd(a,b)$ ,$x_0,y_0$ 是 $ax+by=gcd(a,b)$ 的一组特解

------

#### 乘法逆元

若整数 $b,m$ 互质，并且 $b|a$ ，则存在一个 $x$ ，使得 $a/b\equiv a*x\ (mod\ m)$.
$x$ 为 $b$ 的模 $m$ **乘法逆元**，记为 $b^{-1}(mod\ m)$ ,有 $b*x\equiv1\ (mod\ m)$

求 $x$ 时，若 $m$ 为质数且 $b<m$ ，$b^{m-2}$  则为乘法逆元.
若 $b,m$ 只保证互质，则要求解同余方程 $b*x\equiv1\ (mod\ m)$ 或 $b^{\varphi(m)-1}$

------

#### 线性同余方程

求满足 $a*x\equiv b\ (mod\ m)$ 的 $x$

$gcd(a,m)|b$ 时有解，解为 $a*x+m*y=b$ 的解

------

#### 高次同余方程

$a^x\equiv\ b\ (mod\ p)$ , 求 $x$

**Baby Step,Giant Step算法**

中途相遇

```c++
int bsgs(int a,int b,int p)
{
    map<int,int> hash;hash.claer();
    b%=p;
    int t=ceil(sqrt(p));
    for(int i=0;i<=t-1;i++)
    {
        int val=(long long)b*power(a,j,p)%p;
        hash[val]=j;
    }
    a=power(a,t,p);
    if(a==0) return b==0?1:-1;
    for(int i=0;i<=t;i++)
    {
        int val=power(a,i,p);
        if(hash.find(val)!=hash.end())
            if(i*t-hash[val]>=0) return i*t-hash[val];
    }
    return -1;
}
```

------
