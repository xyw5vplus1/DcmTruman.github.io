---
title: cm_启发式分治学习笔记！
tags: 
  - cm
  - 启发式
  - 分治
categories:
  - cm
  - 分治
date: 2019-8-26
---

# cm_启发式分治学习笔记！



事情的起因是，之前一场杭电的网络赛，有道统计满足某种条件的区间个数的题，自闭了三个小时没有出来。后来看别人题解，说着很明显是一个启发式分治的题，我一脸懵逼（WTF?启发式分治是啥？），然后一怒之下，就决定好好学学这个知识点。
<!-- more -->
首先说一下分治的核心，大家应该都知道，是把一个大问题转换成解决若干个子问题。那么为什么分治能做到一个优秀的时间复杂度呢？关键在于**子问题之间可以快速合并**,你想，如果两个子问题之间也要$O(n^{2})$地去枚举，还有什么分治的必要呢？个人认为，能否快速计算贡献/快速合并 是判断一个题能不能用分治做的一个很重要的点。举个例子，$2^{8}$可以通过子问题$2^{4} * 2^{4}$快速得到，就节省了原来需要线性才能计算得到的时间。

既然要快速计算，就要提到启发式了。当你有两个子问题（比如两个区间），你需要计算横跨两个子问题的种类数，可能会需要枚举一个子问题中的情况，快去得到与之对应的另一个子问题中的情况数，这种时候，选择规模小的子问题去枚举，就叫启发式。**它是启发式合并的逆过程**。

有些时候分裂点可以预处理得到，然后枚举小区间计算横跨大区间的答案。有时候分裂点可能也需要自己找，这种时候从两端向中间找会防止极端情况（我也不太会证）

今天就挑了四个很经典的题，来感受一下分治在统计问题和判断可行性上的优秀思想。

## [BZOJ 4059](https://www.lydsy.com/JudgeOnline/problem.php?id=4059)（[Gym100624D](https://codeforces.com/gym/100624))（Cerc2012)

如果一个序列的所有子段都存在一个数在这个子段中只出现一次，那么这个序列是不无聊的。给你一个序列让你判定无不无聊。

首先预处理出每个数下一次出现的位置和上一次出现的位置，这样可以快速判断当前数在这个区间是否只出现一次。对于当前区间，从两头找只出现一次的数，如果找到，直接判断子区间的结果然后返回。这里有个比较关键的问题，为什么直接返回就行了，为什么不去看看下一个满足条件的数，在分裂递归去判断。大家可以思考一下为什么，搞清楚这一点也就能明白这里用分治为什么复杂度会很优秀了。

代码：

```c++
#include <iostream>
#include <stdio.h>
#include <string.h>
#include <string>
#include <stdlib.h>
#include <map>
using namespace std;
const int maxn = 200020;
int arr[maxn];
int pr[maxn],su[maxn];
map<int , int>pre;
map<int , int>suf;
bool solve(int nowl,int nowr){
	if(nowl >= nowr)return true;
	bool fl = 1;
	int l = nowl , r = nowr;
	while(l <= r){
		if(fl){
			if(su[l] > nowr && pr[l] < nowl){
				return (solve(nowl , l - 1) && solve(l + 1 , nowr ));
			}
			l ++ ;
		}else{
			if(su[r] > nowr && pr[r] < nowl){
				return (solve(nowl , r - 1) && solve(r + 1 , nowr));
			}
			r --;
		}
		fl ^= 1;
	}
	return false;
}
int main()
{
	int T;scanf("%d",&T);
	while(T--){
		int n;scanf("%d",&n);
		for(int i = 1;i<=n;i++){
			scanf("%d",&arr[i]);
		}
		pre.clear();suf.clear();
		for(int i = 1;i<=n;i++){
			pr[i] = pre[arr[i]];
			pre[arr[i]] = i;
		}
		for(int i = n;i>=1;i--){
			su[i] = (suf.count(arr[i]))?suf[arr[i]] : n + 1;
			suf[arr[i]] = i;
		}
		printf(solve(1 , n)?"non-boring\n" : "boring\n");
	}
}
```



## [Gym101623F](https://codeforces.com/gym/101623/my)

这个题给你一个序列，问你是否存在一棵树，每个孩子都和它所有的祖先互质，且中序遍历是这个序列，存在的话输出任意一种情况下每个节点大父亲，不存在输出-1。

对于每个区间，我们挑出一个节点，这个节点和这个区间其他所有数互质，然后判断左右区间能否构成满足条件的子树。和上道题一样，我们只需要找到一个节点就可以直接返回递归之后的结果了，原因依然需要大家思考。

那么如何快速知道一个数是否和这个区间里所有数都互质呢？我们打表的时候找到每个数的素因子。遍历每个数的每个素因子，更新这个素因子最近一次出现的位置，然后就能更新这个数左边第一个和他不互质的数。右边对应也就能求出来。

剩下的就和上面的题很像了。

```c++
#include <iostream>
#include <stdlib.h>
#include <string>
#include <string.h>
#include <stdio.h>
#include <time.h>
#include <stack>
#include <algorithm>
#include <stdio.h>
#include <queue>
#include <math.h>
#include <random>
#include <set>
#include <map>
#include <vector>
using namespace std;
#define ll long long 
#define lson(x) T[(x)].lc
#define rson(x) T[(x)].rc
#define ls(x) x << 1
#define rs(x) x << 1 | 1
#define lowbit(x) x&(-x)
#define INF 0x3f3f3f3f
const int maxn = 1000020;
const int MAXN = 10000000;
int tot,n,m,arr[maxn];
int isnp[MAXN] , pre[MAXN] , pri[MAXN] , fis[MAXN] , las[MAXN];
int fa[maxn];
int A[maxn],B[maxn];
ll ans;
 
void init()
{
	for(int i = 2;i<MAXN;i++){
		if(!isnp[i]){
			pre[i] = i;
			pri[tot] = i;
			tot ++;
		}
		for(int j = 0;j<tot && i * pri[j] < MAXN;j++){
			isnp[i * pri[j]] = true;
			pre[i * pri[j]] = pri[j];
			if(i % pri[j] == 0)break;
		}
	}
}
 
bool solve(int nowl,int nowr,int type)
{
	//cout << nowl << "*" << nowr << endl;
	
	if(nowl == nowr){
		if(type)fa[nowl] = nowl - 1;
		else fa[nowr] = nowr + 1;
		return true;
	}
	if(nowl > nowr)return true;
	bool fl = 1;int l = nowl,r = nowr;
	while(l <= r){
		if(fl){
			//cout << l  << " " << arr[l]<< " " << A[l] << " " << B[l] << endl;
			if(A[l] < nowl && B[l] > nowr){
				if(solve(nowl , l - 1, 0) && solve(l + 1 , nowr , 1)){
					if(type)fa[l] = nowl - 1;
					else fa[l] = nowr + 1;
					return true;
				}else return false;
			}
			l ++;
		}else{
			//cout << r << " " << arr[r] << " "<< A[r] << " " << B[r] << endl;
			if(A[r] < nowl && B[r] > nowr){
				if(solve(nowl , r - 1, 0) && solve(r + 1 , nowr , 1)){
					if(type)fa[r] = nowl - 1;
					else fa[r] = nowr + 1;
					return true;
				}else return false;
			}
			r--;
		}
		fl ^= 1;
	}
	return false;
}
int main()
{
	scanf("%d",&n);
	init();
	B[0] = INF;
	for(int i = 1;i <= n;i++){
		scanf("%d",&arr[i]);
		B[i] = INF;
	}
	for(int i = 1;i<=n;i++){
		int x = arr[i];
		while(x != 1){
			int p = pre[x];
			//cout << x << " " << p << endl;
			A[i] = max(A[i] , (las[p])?las[p]  : 0);
			las[p] = i;
			while(x % p== 0)x /= p;
		}
	}
	for(int i = n;i>=1;i--){
		int x = arr[i];
		while(x != 1){
			int p = pre[x];
			B[i] = min(B[i] , (fis[p])?fis[p] : n + 1);
			fis[p] = i;
			while(x % p == 0)x /= p;
		}
	}
	bool flag = solve(1,n,0);
	if(!flag)printf("impossible\n");
	else{
		for(int i = 1;i<=n;i++){
			if(fa[i] == n+1)fa[i] = 0;
			if(i == 1)printf("%d",fa[i]);
			else printf(" %d",fa[i]);
		}
		printf("\n");
	}
	
}
```



## [2019牛客多校第三场 Removing Stones](https://ac.nowcoder.com/acm/contest/883/G)

给了一个游戏，balabala说了一堆规则，大家对于游戏规则部分，可以去看一下原题，仔细思考一下。但抽象出来之后，让你统计有多少个区间，最大值大于区间总和的一半。

首先通过st表预处理，能$O(1)$快速知道一个区间中最大值的位置，就以这个位置为分裂点。枚举短的区间的每一个值，二分找另一个端点，把这段区间里满足条件的区间数加到贡献里，然后分裂递归。

```c++
#include <iostream>
#include <stdlib.h>
#include <string>
#include <string.h>
#include <stdio.h>
#include <time.h>
#include <stack>
#include <algorithm>
#include <stdio.h>
#include <queue>
#include <math.h>
#include <random>
#include <set>
#include <map>
#include <vector>
using namespace std;
#define ll long long 
#define lson(x) T[(x)].lc
#define rson(x) T[(x)].rc
#define ls(x) x << 1
#define rs(x) x << 1 | 1
#define lowbit(x) x&(-x)
#define INF 0x3f3f3f3f
const int maxn = 300020;
int tot,n,m,arr[maxn];
int root[maxn];
int st[maxn][32];
ll pre[maxn];
ll ans;

int getpos(int nowl,int nowr){
	int ln = log2(nowr-nowl + 1);
	return arr[st[nowl][ln]] > arr[st[nowr - (1 << ln) + 1][ln]]?st[nowl][ln] : st[nowr - (1 << ln) + 1][ln];
}
void solve(int nowl,int nowr){
	if(nowl >= nowr)return;
	int pos = getpos(nowl,nowr);
	//printf("nowl = %d , nowr = %d , nowmx = %d\n" ,nowl,nowr,arr[pos] );
	int mx = arr[pos];
	if(pos - nowl < nowr - pos){
		for(int i = nowl;i<=pos;i++){
			if(pre[pos] - pre[i - 1] >= 2ll * mx){
				ans += (ll)nowr - pos + 1;	
				continue;
			}
			int l = pos,r = nowr,anss = -1;
			while(l <= r){
				int mid = (l + r) >> 1;
				if(pre[mid] - pre[i - 1] >= 2ll * mx){
					anss = mid;
					r = mid - 1;
				}else{
					l = mid + 1;
				}
			}
			if(anss == -1)continue;
			//printf("i = %d , anss = %d\n",i,anss);	
			ans += (ll)nowr - anss + 1;
		}
	}else{
		for(int i = pos;i<=nowr;i++){
			if(pre[i] - pre[pos-1] >= 2ll * mx){
				ans += (ll)pos - nowl + 1;
				continue;
			}
			int l = nowl,r = pos,anss = -1;
			while(l <= r){
				int mid = (l + r) >> 1;
				if(pre[i] - pre[mid - 1] >= 2ll * mx){
					anss = mid;
					l = mid + 1;
				}else{
					r = mid - 1;
				}
			}
			if(anss == -1)continue;
			ans += (ll)anss - nowl + 1;
			//printf("i = %d , anss = %d\n",i,anss);	
		}
	}
	solve(nowl,pos-1);solve(pos+1,nowr);
}
int main()
{
	int T;scanf("%d",&T);
	while(T--){
		int n;scanf("%d",&n);
		pre[0] = 0;
		for(int i = 1;i<=n;i++){
			scanf("%d",&arr[i]);
			pre[i] = pre[i-1] + (ll)arr[i];	
			st[i][0] = i;
		}
		for(int j = 1;(1 << j) <= n;j++){
			for(int i = 1;i+((1<<j) - 1) <= n;i++){
				st[i][j] = arr[st[i][j-1]] > arr[st[i + (1 << (j - 1))][j - 1]]?st[i][j - 1]:st[i + (1 << (j - 1))][ j -1 ];
			}
		}
		ans = 0;
		solve(1,n);
		printf("%lld\n",ans);
	}
}

```



## [HDU 6701Make Rounddog Happy(2019 hdu多校) ](http://acm.hdu.edu.cn/showproblem.php?pid=6701)

给你一个k，然后统计有多少个区间满足，区间内每个数字都不重复，且区间最大值减区间长度小于等于k。

首先，预处理出每个数端点向左/向右延伸的最远距离。

以右端点为例，其想做延伸的最远距离一定是递增的，比较一下上次一这个数出现的位置和上一个数向左延伸的位置，取最大值即可。左端点类似的思想。同时用st表预处理最大值位置。

如何计算贡献？加入当前分裂的左区间短，枚举每个点，我们要计算以当前点为左端点，且横跨最大值，以最大值为区间最大值的区间，且满足题目条件的区间有多少个。根据题目条件，一直最大值和左端点，我们可以计算出右端点至少在哪，根据预处理的数组，可以知道，向右延伸不重复数组区间最远到哪，判断一下大小关系，然后计算贡献即可。

```c++
#include <iostream>
#include <stdlib.h>
#include <string>
#include <string.h>
#include <stdio.h>
#include <time.h>
#include <stack>
#include <algorithm>
#include <stdio.h>
#include <queue>
#include <math.h>
#include <random>
#include <set>
#include <map>
#include <vector>
using namespace std;
#define ll long long 
#define lson(x) T[(x)].lc
#define rson(x) T[(x)].rc
#define ls(x) x << 1
#define rs(x) x << 1 | 1
#define lowbit(x) x&(-x)
#define INF 0x3f3f3f3f
const int maxn = 1000020;
const int nb = 1000010;
int suf[maxn] , dp[maxn],dp1[maxn] , arr[maxn],pre[maxn];
int Max[maxn][30];
int ans;
int n,k,tot;
int query(int nowl,int nowr)
{
    int ln = log2(nowr - nowl + 1);
    return arr[Max[nowl][ln]] >= arr[Max[nowr - (1 << ln) + 1][ln]]?Max[nowl][ln] : Max[nowr - (1 << ln) + 1][ln];

}
void solve(int nowl,int nowr){
    if(nowl > nowr)return;
    int pos = query(nowl,nowr);
    if(pos - nowl < nowr - pos){
        for(int i = nowl;i<=pos;i++){
            int lr = arr[pos] - k + i - 1;
            lr = max(lr,pos);
            int sufnow = min(dp1[i] , nowr);
            if(lr > sufnow)continue;
            ans += sufnow - lr + 1;
        }
    }
    else{
        for(int i = pos;i<=nowr;i++){
            int lr = -(arr[pos] - k - i  - 1);
            lr = min(lr , pos);
            int prenow = max(dp[i] , nowl);
            if(lr < prenow)continue;
            ans += lr - prenow + 1;
        }
    }
    solve(nowl , pos - 1);solve(pos + 1,nowr);
}
int main()
{
    int TT;scanf("%d",&TT);
    while(TT--){
        scanf("%d%d",&n,&k);
        for(int i = 0;i<=n+1;i++){
            pre[i] = 0;
            suf[i] = n + 1;
            dp[i] = 0;dp1[i] = INF;
        }
        for(int i = 1;i<=n;i++){
            scanf("%d",&arr[i]);
            Max[i][0] = i;
        }
        for(int j = 1;j <= 22;j++){
            for(int i = 1;i + (1 << j) - 1 <= n;i++){
                Max[i][j] = arr[Max[i][j-1]] > arr[Max[i + (1 << (j -1))][j - 1]]?Max[i][j - 1] : Max[i + (1<<(j - 1))][j - 1];
            }
        }
        for(int i = 1;i<=n;i++){
            dp[i] = max(pre[arr[i]] + 1 , dp[i - 1]);
            pre[arr[i]] = i;
        }
        for(int i = n;i>=1;i--){
            dp1[i] = min(suf[arr[i]] - 1 , dp1[i + 1]);
            suf[arr[i]] = i;
        }
        ans = 0;
        solve(1,n);
        printf("%d\n",ans);
        
    }
}
```

