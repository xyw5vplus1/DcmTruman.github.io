---
title: cm_字符串相关习题
tags: 
  - 字符串
  - 题目
  - 后缀数组
categories:
  - cm
  - 字符串
date: 2019-09-26
top: False
---

# cm_字符串相关习题

小灯菜鸡一气之下，开始刚字符串！

 ╰（‵□′）╯ 后缀数组！冲啊！

 ヽ(｀⌒´)ﾉ 后缀自动机/广义后缀自动机/后缀树！莽就完事！

 (*´ﾉ皿`) 回文自动机！遇事不决自动机！
 
 \（￣︶￣）/ kmp/扩展kmp 兄弟们上！兄弟们上！
 
 ┴┴︵╰（‵□′）╯︵┴┴  ac自动机 学好ac自动机，再也不用担心题目ac不了啦！！

<!-- more -->

## 后缀数组

- [x] HDU 6194 : 统计出现恰好k次的不同子串的个数 ，算贡献的时候减去前后height的max值
- [x] HDU 6704 : 在height上二分给定$L$的左右边界，对$sa$数组建主席树，求区间第k大
- [x] POJ 1743 ： 二分长度，按照长度对height分组，求每个组对应sa的最大最小值，如果大于长度，true
- [x] POJ 3415 : 给定两个串A和B ，求长度不小于k的公共子串的个数，先把连个字符串连接起来，求height，按照k对height数组，计算B对A的贡献，在计算A对B的贡献，问题在于，当遇到一个B的时候，如何快速求出和他前面每个A所产生的贡献？因为每次遇到一个B的后缀，都是与之前的所有A的后缀计算贡献，在求lcp的时候取最小值，之前每个位置到当前点的最小值肯定是费下降的，我们用单调栈维护，同时维护一下单调栈里应该计算的贡献之和，每遇到一个B，就把这个和加上。（改天应该练习一下单调栈的题）
- [x] 2019牛客第四场 I string。问你有子串，使得这些子串互不相同且也不等于其他子串的反串。正串反串拼起来，求本质不同的子串个数，然后加上原串中本质不同的回文串个数（回文自动机），然后除2就行了。
- [ ] 2018牛客多校第三场String，给你一个字符串，字符串每个位置的后缀+前缀可以得到一个新的字符串，让你统计这些字符串中有多少过不同的字符串。原字符串x2，然后hash一下放在map里即可，也可以x2以后统计多少个本质不同且长度为n的子串，按照n对height分一下组就可以了
- [x] HDU 5769 给定一个字符串S和一个字符X，统计包含X字符的本质不同的子串个数,处理出每个X的位置，预处理离每个后缀起始位置右边最近的X的位置，然后统计答案。
- [x] Codeforces123D ： 每个子串都和其他子串进行尽可能多的匹配，算总次数，在后缀数组上，每个位置都和前面的每一个进行lcp，单调栈维护即可。
- [x] [2018焦作网络赛](https://nanti.jisuanke.com/t/A201) ： 这道题的关键在于，要统计出现大于等于k次的子串，想象在height数组上有一个滑动窗口，每连续k个区间的lcp肯定至少出现过k次，我们每次计算完都要减去上一次计算出来的答案，如果大于0在相加，避免一些字符串算重复了。
- [x] 2017icpc 青岛 suffix  当时打铁很难受，字符串题最终还是没敢试暴力。**从后往前拼**，拼出来的串求一发后缀数组，找到字典序最小的在原字符串的后缀，然后和上一个拼，如此重复
- [x] POJ 3261 找到尽可能长的子串，该子串至少出现k次（可重叠）, 二分长度，按照长度分组，每组个数表示出现多少次，判断存不存在个数大于等于k的即可
- [x] SPOJ - REPEATS 重复次数最多的连续重复子串，论文里有，讲的比较清楚了。这里有个实现上的点，往后匹配尽可能多的字符，我们可以通过后缀字符得到，往前匹配尽可能多的字符如何做呢？一开始我的想法是反串在建一个后缀数组，就是多个常数的事，确实也可以。别人的题解里，比如当前枚举的长度是L，只需要看一下匹配长度%枚举长度多出来几个字符，就再往前查看（L - 多出来的字符）,判断能不能再凑一个循环节，以利用这些多出来的字符即可。可是为什么只用查看这些而不全匹配呢？因为如果往前会匹配到更多的，那么在之前的枚举中我们必然已经遇到过了。这部分如果不太理解那就反串再建一遍好了。
- [x] HDU 5008 找出本质不同的字符串中，第k小的，输出左右端点。如果有多个，输出左端点最小的那个。处理除了对于每个后缀，每次新增多少个，因为后缀数组本来就是字典序，所以二分就行了，但同时也要往后枚举一下，看看后面是否有左端点更小的。
- [x] 2018南京现场赛M ：把s串翻转，拼上#t串，然后相当于是求每个s'的位置和t位置的最长公共前缀，在乘上该位置为右端点的回文串的个数，上个回文自动机就行了

顺便贴一发我的板子,字符串下标从0开始，Rank数组、sa数组和height下标才1开始，sa数组的值（即原始字符串位置）从0开始。DA的参数n是原始字符串长度+1，r数组切记把`r[n]`置为一个未出现的数字

```c++
int cmp(int *r,int a,int b,int l){
    return (r[a]==r[b]) && (r[a+l]==r[b+l]);
}
int wa[N],wb[N],Ws[N],wv[N];
int Rank[N],height[N],root[N],n,m,st[maxn][25];
int k;
void DA(int *r,int *sa,int n,int m){ 
    int i,j,p,*x=wa,*y=wb,*t;
    for(i=0;i<m;i++) Ws[i]=0;
    for(i=0;i<n;i++) Ws[x[i]=r[i]]++;
    for(i=1;i<m;i++) Ws[i]+=Ws[i-1];
    for(i=n-1;i>=0;i--) sa[--Ws[x[i]]]=i; 
    for(j=1,p=1;p<n;j*=2,m=p) 
    {
        for(p=0,i=n-j;i<n;i++) y[p++]=i; 
        for(i=0;i<n;i++) if(sa[i]>=j) y[p++]=sa[i]-j; 
        for(i=0;i<n;i++) wv[i]=x[y[i]];
        for(i=0;i<m;i++) Ws[i]=0;
        for(i=0;i<n;i++) Ws[wv[i]]++;
        for(i=1;i<m;i++) Ws[i]+=Ws[i-1];
        for(i=n-1;i>=0;i--) sa[--Ws[wv[i]]]=y[i];  
        for(t=x,x=y,y=t,p=1,x[sa[0]]=0,i=1;i<n;i++)
            x[sa[i]]=cmp(y,sa[i-1],sa[i],j)?p-1:p++;  
    }
}

void calheight(int *r,int *sa,int n){ 
    int i,j,k=0;       
    for(i=1;i<=n;i++) Rank[sa[i]]=i;  
    for(i=0;i<n; height[Rank[i++]] = k ) 
    for(k?k--:0,j=sa[Rank[i]-1]; r[i+k]==r[j+k]; k++); 
} 

string sta[N];
char str[N];
int r[N];
ll r2[N];
ll pre[N];
int sa[N];
int getmin(int x,int y)
{
    //cout << x << " " << y << " ";
    if(x == y)return n - sa[x];
    if(x > y)swap(x,y);
	x ++;
    int ln = log2(y - x  + 1);
    int ret = min(st[x][ln] , st[y - (1 << ln) + 1][ln]);
	//cout << ret << endl;
	return ret;
}
```

DC3的板子

```c++
struct DA{
    #define F(x) ((x)/3+((x)%3==1?0:tb))
    #define G(x) ((x)<tb?(x)*3+1:((x)-tb)*3+2)
    int sa[maxn*6],rank[maxn*6],height[maxn*6],str[maxn*6];
    int wa[maxn*6],wb[maxn*6],wv[maxn*6],wss[maxn*6];
    int c0(int *r,int a,int b){
        return r[a]==r[b]&&r[a+1]==r[b+1]&&r[a+2]==r[b+2];
    }
    int c12(int k,int *r,int a,int b){
        if(k==2)
            return r[a]<r[b]||(r[a]==r[b]&&c12(1,r,a+1,b+1));
        else return r[a]<r[b]||(r[a]==r[b]&&wv[a+1]<wv[b+1]);
    }
    void sort(int *r,int *a,int *b,int n,int m){
        int i;
        for(i=0;i<n;i++)wv[i]=r[a[i]];
        for(i=0;i<m;i++)wss[i]=0;
        for(i=0;i<n;i++)wss[wv[i]]++;
        for(i=1;i<m;i++)wss[i]+=wss[i-1];
        for(i=n-1;i>=0;i--)
            b[--wss[wv[i]]]=a[i];
    }
    void dc3(int *r,int *sa,int n,int m){
        int i,j,*rn=r+n;
        int *san=sa+n,ta=0,tb=(n+1)/3,tbc=0,p;
        r[n]=r[n+1]=0;
        for(i=0;i<n;i++)if(i%3!=0)wa[tbc++]=i;
        sort(r+2,wa,wb,tbc,m);
        sort(r+1,wb,wa,tbc,m);
        sort(r,wa,wb,tbc,m);
        for(p=1,rn[F(wb[0])]=0,i=1;i<tbc;i++)
            rn[F(wb[i])]=c0(r,wb[i-1],wb[i])?p-1:p++;
        if(p<tbc)dc3(rn,san,tbc,p);
        else for(i=0;i<tbc;i++)san[rn[i]]=i;
        for(i=0;i<tbc;i++)if(san[i]<tb)wb[ta++]=san[i]*3;
        if(n%3==1)wb[ta++]=n-1;
        sort(r,wb,wa,ta,m);
        for(i=0;i<tbc;i++)wv[wb[i]=G(san[i])]=i;
        for(i=0,j=0,p=0;i<ta&&j<tbc;p++)
            sa[p]=c12(wb[j]%3,r,wa[i],wb[j])?wa[i++]:wb[j++];
        for(;i<ta;p++)sa[p]=wa[i++];
        for(;j<tbc;p++)sa[p]=wb[j++];
    }
    void da(int n,int m){
        for(int i=n;i<n*3;i++)str[i]=0;
        dc3(str,sa,n+1,m);
        int i,j,k=0;
        for(i=0;i<=n;i++)rank[sa[i]]=i;
        for(i=0;i<n;i++){
            if(k)k--;
            j=sa[rank[i]-1];
            while(str[i+k]==str[j+k])k++;
            height[rank[i]]=k;
        }
    }
    void print(int n){
        cout<<"sa[] ";
        for(int i=0;i<=n;i++)cout<<sa[i]<<" ";cout<<endl;
        cout<<"rank[] ";
        for(int i=0;i<=n;i++)cout<<rank[i]<<" ";cout<<endl;
        cout<<"height[] ";
        for(int i=0;i<=n;i++)cout<<height[i]<<" ";cout<<endl;
    }
}DA;
```

## 后缀自动机

其实后缀自动机的题，上上周就已经看了不少了，但一直没有怎么整理，今天特此在列一些题

- [x] SPOJ-LCS ： 模板题，求两个串的最长公共子串，建好S串的后缀自动机，在T串上跑就行了，失配就沿着father跳知道0
- [x] [后缀自动机二](http://hihocoder.com/problemset/problem/1445) ： 求本质不同的子串，建好后缀自动机，每个位置的$max(i)-min(i)$就是当前点不的不同字串，每个点相加就可以了。因为任何一条路径都能唯一的代表这个串的一个子串，所以拓扑排序之后dp计算路径数也可。
- [x] [后缀自动机三](http://hihocoder.com/problemset/problem/1449) ： 对于任何长度为k的子串，输出出现次数最多的子串的出现次数。每个点的right集合的大小就是当前节点所表示的字符串的出现次数，right集合大小在link树上累加即可（拓扑排序，儿子传递给父亲）。用right集合更新ans[max(i)]的大小，在用max(i)去更新ans[max(i)]...ans[min(i)]的值。
- [x] [后缀自动机四](http://hihocoder.com/problemset/problem/1457) ： 给你个0~9组成的串，当成数字看。求本质不同的串的数字和（这道题让我想去以前网络赛一个“求本质不同回文子串的数字和”）。如果是单个串。每个节点记录当前节点所表示的串的和，转移的时候就是当前节点*10+转移边-'0'。如果是多个传，就用‘：’连接（因为‘：’是‘0’的ascii码+10）
- [x] SPOJ LCS2 ： 多个串的最长公共子串。先用第一个串建自动机。剩下串去匹配。每个节点记录一下当前串匹配到的最大值。在记录一些每个串匹配到的最大值的最小值。注意，这个最小值是可以向父亲传递的，拓扑排序一下再儿子向父亲更新一遍最大值。然后输出每个节点记录的最小值的最大值即可。
- [x] SPOJ-SUBLEX ： 本质不同的子串第字典序第k大子串。dp一下表示从自动机这个点出发有多少种路径可以走。在自动机上贪心地边减k边走就完事儿。
- [x] HDU 4416 : 原串S建后缀自动机，然后不断拿着模式串T，去更新节点上的deep值。deep值表示，这些模式串中，到当前节点能匹配到的最大的长度。然后拓扑排序之后，每个节点都向father更新deep值的max。最后对于每个节点，如果deep为0，答案就加上当前节点所代表的子串，不过不为0，答案就加上当前节点所代表的子串到所匹配到的最大子串之前的这些子串，也就是`len[i] - deep[i]`，不过我怕算重，更新的是`len[i] - max(len[fa[i]] , deep[i])`,两种都能AC，后来想了想也确实如此。
- [x] [APIO2014](https://www.luogu.org/problem/P3649) 统计每个本质不同的回文串的出现次数和长度，然后输出乘积的最大值。注意儿子会对父亲有贡献，所以还要再把贡献累加上


顺便贴一发后缀自动机的板子

```c++
const int maxn = 1e6 + 20;
const ll MOD = 1e9+7;
struct SAM{
	#define CHAR_NUM 26
	int fa[maxn << 1] , ch[maxn << 1][CHAR_NUM] , len[maxn << 1];
	int siz[maxn << 1];//right num
	int sum[maxn << 1];//path num
	int t[maxn << 1] , A[maxn << 1];
	int lst , node;
	void init(){
		lst = node = 1;
		memset(fa , 0 , sizeof(fa));
		memset(ch , 0 , sizeof(ch));
		memset(len , 0 , sizeof(len));
		memset(siz , 0 , sizeof(siz));
	}
	void insert(int c){
		//------
		if(ch[lst][c]&&len[ch[lst][c]] == len[lst] + 1){
			lst = ch[lst][c];siz[lst] ++;return;
		}
		//------
		int f = lst , p = ++node;lst = p;
		len[p] = len[f] + 1;siz[p] = 1;
		//------
		while(f && !ch[f][c]){ch[f][c] = p;f = fa[f];}
		if(!f){fa[p]=1;return;}
		//------
		int x = ch[f][c] , y = ++node;
		if(len[f] + 1 == len[x]){fa[p] = x;node -- ;return;}
		//------
		len[y] = len[f] + 1;fa[y] = fa[x];fa[x] = fa[p] = y;
		memcpy(ch[y] , ch[x] , sizeof(ch[y]));
		while(f && ch[f][c] == x){ch[f][c] = y ; f = fa[f];}
	}
	void ssort(){
		//small --- > big
		for(int i=1;i<=node;i++) t[len[i]]++;
		for(int i=1;i<=node;i++) t[i]+=t[i-1];
		for(int i=1;i<=node;i++) A[t[len[i]]--]=i;
	}
	void getsiz(){
		for(int i = node;i>=1;i--){
			siz[fa[A[i]]] += siz[A[i]];
		}
	}
	void getsum(){
		sum[1] = 1;
		for(int i = 1;i <= node;i++){
			for(int j = 0 ; j < CHAR_NUM  ; j++){
				if(ch[A[i]][j]){
					sum[ch[A[i]][j]] += sum[A[i]];
				}
			}
		}
	}
	
}sam;
```

## 回文自动机

- [x] [URAL-1960](https://vjudge.net/problem/URAL-1960) : 裸题，不解释了，模板中p代表当前自动机的节点个数，也就是本质不同的回文串的个数
- [x] URAL 2040 : 裸题，每次添加字符，至多只会新增一个本质不同的回文子串，被卡时间卡空间，我要恶心死了
- [x] [P4555](https://www.luogu.org/problem/P4555) 国家集训队 最长双回文串 ： 正串跑回文树，求出s[i],si[i]表示当前点为右端点的最长的回文子串，反串跑回文树，求ti[i]，表示左端点的最长回文子串，然后遍历一遍求个mx即可`mx = max(mx , si[i] + ti[i + 1])`
- [x] [P1659 国家集训队 拉拉队排练](https://www.luogu.org/problem/P1659) : 通过回文自动机求出每个长度的回文串有多少个，然后算答案快速幂一下就行
- [x] Palisection CodeForces - 17E : 统计有多少对回文串是相交的。我们只需要求出来不相交的对就行了。用`sum[i]`表示，0~i的字符串中有多少回文串，`pos[i]` 表示以`i`为**左端点**的回文串有多少个，`sum[i] * pos[i+1]`的总和就是不相交的回文串对数，用总对数交掉就可以了。因为内存问题，所以这里用了链式前向星，等下也会贴出来这部分，复杂度比传统的略高。

顺便贴一发自己回文自动机的板子

```c++

struct Palindromic_Tree {
    int next[MAXN][CHARN] ;//next指针，next指针和字典树类似，指向的串为当前串两端加上同一个字符构成
    int fail[MAXN] ;//fail指针，失配后跳转到fail指针指向的节点
    int cnt[MAXN] ;
    int num[MAXN] ;
    int len[MAXN] ;//len[i]表示节点i表示的回文串的长度
    int S[MAXN] ;//存放添加的字符
    int last ;//指向上一个字符所在的节点，方便下一次add
    int n ;//字符数组指针
    int p ;//节点指针

    int newnode ( int l ) {//新建节点
        for ( int i = 0 ; i < CHARN ; ++ i ) next[p][i] = 0 ;
        cnt[p] = 0 ;
        num[p] = 0 ;
        len[p] = l ;
        return p ++ ;
    }

    void init () {//初始化
        p = 0 ;
        newnode (  0 ) ;
        newnode ( -1 ) ;
        last = 0 ;
        n = 0 ;
        S[n] = -1 ;//开头放一个字符集中没有的字符，减少特判
        fail[0] = 1 ;
    }

    int get_fail ( int x ) {//和KMP一样，失配后找一个尽量最长的
        while ( S[n - len[x] - 1] != S[n] ) x = fail[x] ;
        return x ;
    }

    int add ( int c ) {
        //c -= 'a' ;
        S[++ n] = c ;
        int cur = get_fail ( last ) ;//通过上一个回文串找这个回文串的匹配位置
        if ( !next[cur][c] ) {//如果这个回文串没有出现过，说明出现了一个新的本质不同的回文串
            int now = newnode ( len[cur] + 2 ) ;//新建节点
            fail[now] = next[get_fail ( fail[cur] )][c] ;//和AC自动机一样建立fail指针，以便失配后跳转
            next[cur][c] = now ;
            num[now] = num[fail[now]] + 1 ;
        }
        last = next[cur][c] ;
        cnt[last] ++ ;
		return num[last];
    }

    void count () {
        for ( int i = p - 1 ; i >= 0 ; -- i ) cnt[fail[i]] += cnt[i] ;
        //父亲累加儿子的cnt，因为如果fail[v]=u，则u一定是v的子回文串！
    }
} pat;

```

邻接表版本

```
struct link_list{
    int u[MAXN], v[MAXN], nxt[MAXN], head[MAXN], tot, i;
    void _clear(){
        memset(head, -1, sizeof head);
        tot = 0;
    }
    void _clear(int x){head[x] = -1; }
    int get(int x, int y){
        for(i = head[x]; i != -1; i = nxt[i]){
            if(u[i] == y) return v[i];
        }
        return 0;
    }
    void  _insert(int x, int y, int z){
        u[tot] = y, v[tot] = z;
        nxt[tot] = head[x];
        head[x] = tot++;
    }
};
struct Palindromic_Tree {
	link_list nxt;
    int fail[MAXN] ;//fail指针，失配后跳转到fail指针指向的节点
    int cnt[MAXN] ;
    int num[MAXN] ;
    int len[MAXN] ;//len[i]表示节点i表示的回文串的长度
    int S[MAXN] ;//存放添加的字符
    int last ;//指向上一个字符所在的节点，方便下一次add
    int n ;//字符数组指针
    int p ;//节点指针

    int newnode ( int l ) {//新建节点
		nxt._clear(p);
        cnt[p] = 0 ;
        num[p] = 0 ;
        len[p] = l ;
        return p ++ ;
    }

    void init () {//初始化
		nxt._clear();
        p = 0 ;
        newnode (  0 ) ;
        newnode ( -1 ) ;
        last = 0 ;
        n = 0 ;
        S[n] = -1 ;//开头放一个字符集中没有的字符，减少特判
        fail[0] = 1 ;
    }

    int get_fail ( int x ) {//和KMP一样，失配后找一个尽量最长的
        while ( S[n - len[x] - 1] != S[n] ) x = fail[x] ;
        return x ;
    }

    int add ( int c ) {
        //c -= 'a' ;
        S[++ n] = c ;
        int cur = get_fail ( last ) ;//通过上一个回文串找这个回文串的匹配位置
		if(!nxt.get(cur,c)){
        //if ( !next[cur][c] ) {//如果这个回文串没有出现过，说明出现了一个新的本质不同的回文串
            int now = newnode ( len[cur] + 2 ) ;//新建节点
			int tmp = get_fail(fail[cur]);
			fail[now] = nxt.get(tmp,c);
            //fail[now] = next[get_fail ( fail[cur] )][c] ;//和AC自动机一样建立fail指针，以便失配后跳转
			nxt._insert(cur,c,now);
            //next[cur][c] = now ;
            num[now] = num[fail[now]] + 1 ;
        }
        //last = next[cur][c] ;
		last = nxt.get(cur,c);
        cnt[last] ++ ;
		return last;
    }

    void count () {
        for ( int i = p - 1 ; i >= 0 ; -- i ) cnt[fail[i]] += cnt[i] ;
        //父亲累加儿子的cnt，因为如果fail[v]=u，则u一定是v的子回文串！
    }
} pat;

```
