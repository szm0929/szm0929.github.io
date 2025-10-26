# 分治
分治，就是分而治之，先把问题划分为更小规模的问题，然后再进行合并。
## 主定理
??? info
	上课时是从平面最近点对开始引入主定理的，这里为了方便先讲主定理。
一般的分治时间复杂度递推式为：

$$T(n)=aT(\frac{n}{b})+f(n)$$

这里认为 $f(n)=n^c$，则：

$$T(n)=\begin{cases}
O(n^c),&a<b^c\\
O(n^c\log n),&a=b^c\\
O(n^{\log_ba}),&a>b^c
\end{cases}$$

当然，若 $f(n)=n^c\log^kn(k\ge 0)$，则当 $a=b^c$ 时有 $T(n)=n^c\log^{k+1}n$。
## 平面最近点对
给出平面上的 $n$ 个点，求最近的两个点。

可以将这 $n$ 个点按照 $x$ 坐标排序，然后分成左右两边各 $n/2$ 个点。

记 $\delta_1$ 为左边点对的最近距离，$\delta_2$ 为右边点对的最近距离，并令 $\delta=\min(\delta_1,\delta_2)$。接下来仅需考虑跨分界线的点即可。且这些点距离分界线均小于 $\delta$

由于左右两边所有点之间的距离均不小于 $\delta$，因此可以把带划分成多个 $\frac{\delta}{2}\times \frac{\delta}{2}$ 的正方形，则每个正方形至多存在一个点。

因此我们再将中间的点按照 $y$ 坐标进行排序，则至多需要向后比较 $7$ 个点。

如果直接排序，则有 $T(n)=2T(\frac{n}{2})+O(n\log n)$，时间复杂度为 $O(n\log^2 n)$。

其实可以进行优化，注意到这一过程可以与归并排序同时进行，此时有 $T(n)=2T(\frac{n}{2})+O(n)$，时间复杂度为 $O(n\log n)$。

??? 参考代码
	```cpp
	#include<bits/stdc++.h>
	#define ll long long
	using namespace std;
	const int N=4e5+7;
	ll read(){
		ll x=0,f=1;
		char c=getchar();
		while(!isdigit(c)){
			if(c=='-')
				f=-1;
			c=getchar();
		}
		while(isdigit(c)){
			x=x*10+f*(c-48);
			c=getchar();
		}
		return x;
	}
	ll sqr(ll x){
		return x*x;
	}
	ll n,x[N],y[N],p[N],q[N];
	bool cmp(ll a,ll b){
		return x[a]<x[b];
	}
	ll dist(ll i,ll j){
		return sqr(x[i]-x[j])+sqr(y[i]-y[j]);
	}
	ll solve(ll l,ll r){
		if(r-l+1<=3){
			ll mn=1e18;
			for(ll i=l;i<=r;++i)
				for(ll j=i+1;j<=r;++j)
					mn=min(mn,dist(p[i],p[j]));
			sort(p+l,p+r+1,[](ll a,ll b){
				return y[a]<y[b];
			});
			return mn;
		}
		ll mid=(l+r)>>1;
		ll midx=x[p[mid]];
		ll d=min(solve(l,mid),solve(mid+1,r));
		ll a=l,b=mid+1,c=l;
		while(a<=mid&&b<=r){
			if(y[p[a]]<y[p[b]])
				q[c++]=p[a++];
			else
				q[c++]=p[b++];
		}
		while(a<=mid)
			q[c++]=p[a++];
		while(b<=r)
			q[c++]=p[b++];
		for(ll i=l;i<=r;++i)
			p[i]=q[i];
		vector<ll>vec;
		for(ll i=l;i<=r;++i){
			if(sqr(x[p[i]]-midx)<d)
				vec.push_back(p[i]);
		}
		ll sz=vec.size();
		for(ll i=0;i<sz;++i){
			for(ll j=i+1;j<sz&&sqr(y[vec[j]]-y[vec[i]])<d;++j)
				d=min(d,dist(vec[i],vec[j]));
		}
		return d;
	}
	int main(){
		#ifdef alarm5854
		freopen("Closest.in","r",stdin);
		freopen("Closest.out","w",stdout);
		#endif
		n=read();
		for(int i=1;i<=n;++i){
			x[i]=read();
			y[i]=read();
			p[i]=i;
		}
		sort(p+1,p+n+1,cmp);
		printf("%lld\n",solve(1,n));
		return 0;
	}
	```
## 大整数乘法
??? info
	接下来的内容为拓展内容，ads的PPT止步于此。

给定两个大整数 $A,B$，求 $A\times B$。

### 引入：小学算法
先处理 A+B problem 与 A-B problem。

计算加减法的时候，都是列竖式解决的。对于计算机而言，也是如此。设 $A$ 与 $B$ 的较大的数的位数为 $n$，则时间复杂度为 $O(n)$。

然而计算乘法的时候，如果列竖式解决，时间复杂度为 $O(n^2)$，因此需要进行优化。

### 分治乘法
将整数按 $k(k\ge 2)$ 进制进行划分，拆成高位和低位两部分：

$$A=a\times k^m+b,B=c\times 2^m+d$$

则：

$$A\times B=ab\times k^{2m}+(ad+bc)\times k^m+cd$$

若直接进行运算，则 $T(n)=4T(\frac{n}{2})+O(n)$，即时间复杂度为 $O(n^2)$，并没有改进。

Karatsuba 改进：

$$ad+bc=(a+b)(c+d)-ac-bd$$

因此只需要计算 $ac,bd,(a+b)(c+d)$ 这三次乘法，$T(n)=3T(\frac{n}{2})+O(n)$，即时间复杂度为 $O(n^{\log_23})$，即 $O(n^{1.585})$。

然而，此时需要注意空间开销，否则空间复杂度也会达到这个量级！

对于洛谷 P1919，取 $k=10^9$，可以通过。
??? 参考代码
	```cpp
	#include<bits/stdc++.h>
	#define ll long long
	using namespace std;
	const int N=1e6+7;
	const int lim=4096;
	const int base=1e9;
	ll* add(ll* a,ll* b);
	ll* sub(ll* a,ll* b);
	ll* nor(ll* a){
		if(a[0]==-1&&a[1]==0)
			a[0]=1;
		return a;
	}
	ll* add(ll* a,ll* b){
		if(a[0]*b[0]<0){
			if(a[0]>0){
				b[0]=-b[0];
				ll* res=sub(a,b);
				b[0]=-b[0];
				return res;
			}
			a[0]=-a[0];
			ll* res=sub(b,a);
			a[0]=-a[0];
			return res;
		}
		ll lena=abs(a[0]),lenb=abs(b[0]);
		ll len=max(lena,lenb);
		ll flg=a[0]>0?1:-1;
		ll *res=new ll[len+2];
		ll *ta=new ll[len+2],*tb=new ll[len+2];
		memset(res,0,sizeof(ll)*(len+2));
		memset(ta,0,sizeof(ll)*(len+2));
		memset(tb,0,sizeof(ll)*(len+2));
		for(ll i=1;i<=lena;++i)
			ta[i]=a[i];
		for(ll i=1;i<=lenb;++i)
			tb[i]=b[i];
		for(ll i=1;i<=len;++i){
			res[i]+=ta[i]+tb[i];
			if(res[i]>=base){
				res[i]-=base;
				++res[i+1];
			}
		}
		if(res[len+1]>0)
			++len;
		res[0]=flg*len;
		delete[] ta;
		delete[] tb;
		return nor(res);
	}
	bool cmp(ll* a,ll* b){
		if(a[0]^b[0])
			return a[0]<b[0];
		if(a[0]>0){
			for(int i=a[0];i;--i){
				if(a[i]^b[i])
					return a[i]<b[i];
			}
			return 0;
		}
		for(int i=-a[0];i;--i){
			if(a[i]^b[i])
				return a[i]>b[i];
		}
		return 0;
	}
	ll* sub(ll* a,ll* b){
		if(a[0]*b[0]<0){
			if(a[0]>0){
				b[0]=-b[0];
				ll* res=add(a,b);
				b[0]=-b[0];
				return res;
			}
			a[0]=-a[0];
			ll* res=add(a,b);
			a[0]=-a[0];
			res[0]=-res[0];
			return res;
		}
		if(a[0]<0){
			a[0]=-a[0];
			b[0]=-b[0];
			ll* res=sub(b,a);
			a[0]=-a[0];
			b[0]=-b[0];
			return res;
		}
		if(cmp(a,b)){
			ll* res=sub(b,a);
			res[0]=-res[0];
			return res;
		}
		ll len=a[0];
		ll *res=new ll[len+1];
		ll *ta=new ll[len+1],*tb=new ll[len+1];
		memset(res,0,sizeof(ll)*(len+1));
		memset(ta,0,sizeof(ll)*(len+1));
		memset(tb,0,sizeof(ll)*(len+1));
		for(ll i=1;i<=a[0];++i)
			ta[i]=a[i];
		for(ll i=1;i<=b[0];++i)
			tb[i]=b[i];
		for(ll i=1;i<=len;++i){
			res[i]+=ta[i]-tb[i];
			if(res[i]<0){
				res[i]+=base;
				--res[i+1];
			}
		}
		while(len>1&&!res[len])
			--len;
		res[0]=len;
		delete[] ta;
		delete[] tb;
		return nor(res);
	}
	ll* mul(ll* a,ll* b){
		ll lena=abs(a[0]),lenb=abs(b[0]);
		if(lena*lenb<=lim){//当位数较小时，使用小学算法更加合适。
			ll len=lena+lenb;
			ll* res=new ll[len+1];
			memset(res,0,sizeof(ll)*(len+1));
			for(ll i=1;i<=lena;++i){
				for(ll j=1;j<=lenb;++j){
					res[i+j-1]+=a[i]*b[j];
					if(res[i+j-1]>=8ll*base*base){
						res[i+j]+=res[i+j-1]/base;
						res[i+j-1]%=base;
					}
				}
			}
			for(ll i=1;i<=len;++i){
				if(res[i]>=base){
					res[i+1]+=res[i]/base;
					res[i]%=base;
				}
			}
			while(len>1&&!res[len])
				--len;
			res[0]=(a[0]*b[0]>0?1:-1)*len;
			return nor(res);
		}
		ll len=max(lena,lenb);
		ll mid=(len+1)>>1;
		ll *lowa=new ll[(len>>1)+2],*higha=new ll[(len>>1)+2];
		ll *lowb=new ll[(len>>1)+2],*highb=new ll[(len>>1)+2];
		memset(lowa,0,sizeof(ll)*((len>>1)+2));
		memset(higha,0,sizeof(ll)*((len>>1)+2));
		memset(lowb,0,sizeof(ll)*((len>>1)+2));
		memset(highb,0,sizeof(ll)*((len>>1)+2));
		for(ll i=1;i<=lena;++i){
			if(i<=mid)
				lowa[i]=a[i];
			else
				higha[i-mid]=a[i];
		}
		for(ll i=1;i<=lenb;++i){
			if(i<=mid)
				lowb[i]=b[i];
			else
				highb[i-mid]=b[i];
		}
		lowa[0]=higha[0]=mid;
		lowb[0]=highb[0]=mid;
		while(lowa[0]>1&&!lowa[lowa[0]])
			--lowa[0];
		while(higha[0]>1&&!higha[higha[0]])
			--higha[0];
		while(lowb[0]>1&&!lowb[lowb[0]])
			--lowb[0];
		while(highb[0]>1&&!highb[highb[0]])
			--highb[0];
		ll* z0=mul(lowa,lowb);
		ll* z1=mul(higha,highb);
		ll* sa=add(lowa,higha);
		ll* sb=add(lowb,highb);
		ll* tot=mul(sa,sb);
		ll* sum=add(z0,z1);
		ll* z2=sub(tot,sum);
		ll* res=new ll[2*len+2];
		memset(res,0,sizeof(ll)*(2*len+2));
		for(ll i=1;i<=z0[0];++i)
			res[i]+=z0[i];
		for(ll i=1;i<=z2[0];++i)
			res[i+mid]+=z2[i];
		for(ll i=1;i<=z1[0];++i)
			res[i+2*mid]+=z1[i];
		res[0]=2*len;
		for(ll i=1;i<=res[0];++i){
			if(res[i]>=base){
				res[i+1]+=res[i]/base;
				res[i]%=base;
			}
		}
		while(res[0]>1&&!res[res[0]])
			--res[0];
		res[0]=(a[0]*b[0]>0?1:-1)*res[0];
		delete[] lowa;
		delete[] higha;
		delete[] lowb;
		delete[] highb;
		delete[] z0;
		delete[] z1;
		delete[] z2;
		delete[] sa;
		delete[] sb;
		delete[] tot;
		delete[] sum;
		return nor(res);
	}
	struct bigint{
		ll* v;
		bigint(){
			v=new ll[2];
			v[0]=1;
			v[1]=0;
		}
		bigint(ll x){
			if(x==0){
				v=new ll[2];
				v[0]=1;
				v[1]=0;
				return;
			}
			ll flg=1;
			if(x<0){
				flg=-1;
				x=-x;
			}
			ll len=0;
			ll t=x;
			while(t){
				++len;
				t/=base;
			}
			v=new ll[len+1];
			v[0]=flg*len;
			for(ll i=1;i<=len;++i){
				v[i]=x%base;
				x/=base;
			}
		}
		bigint(string s){
			ll flg=1;
			ll len=s.length();
			ll start=0;
			if(s[0]=='-'){
				flg=-1;
				--len;
				start=1;
			}
			ll cnt=(len+8)/9;
			v=new ll[cnt+1];
			memset(v,0,sizeof(ll)*(cnt+1));
			v[0]=flg*cnt;
			for(ll i=len-1+start,j=1;i>=start;){
				ll t=1;
				for(ll k=0;k<9&&i>=start;++k,--i){
					v[j]+=t*(s[i]-48);
					t*=10;
				}
				++j;
			}
			v=nor(v);
		}
		void print(){
			if(v[0]<0)
				putchar('-');
			printf("%lld",v[abs(v[0])]);
			for(ll i=abs(v[0])-1;i>=1;--i)
				printf("%09lld",v[i]);
			puts("");
		}
		friend bigint operator+(const bigint& a,const bigint& b){
			bigint res;
			res.v=add(a.v,b.v);
			return res;
		}
		friend bigint operator-(const bigint& a,const bigint& b){
			bigint res;
			res.v=sub(a.v,b.v);
			return res;
		}
		friend bigint operator*(const bigint& a,const bigint& b){
			bigint res;
			res.v=mul(a.v,b.v);
			return res;
		}
	};
	char sa[N],sb[N];
	bigint a,b;
	int main(){
		#ifdef alarm5854
		freopen("mul.in","r",stdin);
		freopen("mul.out","w",stdout);
		#endif
		scanf(" %s",sa);
		scanf(" %s",sb);
		a=bigint(sa);
		b=bigint(sb);
		bigint c=a*b;
		c.print();
		return 0;
	}
	```
## 矩阵乘法
对于两个 $n$ 阶方阵 $A,B$，令 $C=A\times B$，则：

$$C_{i,j}=\sum_{k=1}^n A_{i,k}B_{k,j}$$

经典方法的时间复杂度时 $O(n^3)$ 的。

我们可以将一个 $n$ 阶方阵拆成四个 $\frac{n}{2}$ 阶方阵：

$$A=\begin{pmatrix}
A_{00} & A_{01}\\
A_{10} & A_{11}
\end{pmatrix},
B=\begin{pmatrix}
B_{00} & B_{01}\\
B_{10} & B_{11}
\end{pmatrix},
C=\begin{pmatrix}
C_{00} & C_{01}\\
C_{10} & C_{11}
\end{pmatrix}$$

则：

$$\begin{cases}
C_{00}=A_{00}B_{00}+A_{01}B_{10}\\
C_{01}=A_{00}B_{01}+A_{01}B_{11}\\
C_{10}=A_{10}B_{00}+A_{11}B_{10}\\
C_{11}=A_{10}B_{01}+A_{11}B_{11}
\end{cases}$$

此时 $T(n)=8T(\frac{n}{2})+O(n^2)$，时间复杂度仍然为 $O(n^3)$，没有优化。

Strassen 注意到：通过巧妙的线性组合只用 7 次矩阵乘法（外加 18 次加减法），减少一次乘法：

$$\begin{cases}
P_1=A_{00}(B_{01}-B_{11})\\
P_2=(A_{00}+A_{01})B_{11}\\
P_3=(A_{10}+A_{11})B_{00}\\
P_4=A_{11}(B_{10}-B_{00})\\
P_5=(A_{00}+A_{11})(B_{00}+B_{11})\\
P_6=(A_{01}-A_{11})(B_{10}+B_{11})\\
P_7=(A_{00}-A_{10})(B_{00}+B_{01})
\end{cases}$$

然后组合成：

$$\begin{cases}
C_{00}=P_5+P_4-P_2+P_6\\
C_{01}=P_1+P_2\\
C_{10}=P_3+P_4\\
C_{11}=P_1+P_5-P_3-P_7
\end{cases}$$

此时 $T(n)=7T(\frac{n}{2})+O(n^2)$，时间复杂度为 $O(n^{\log_27})$，即 $O(n^{2.81})$。

然而该算法必须保证矩阵阶数为 $2^k$，若不足，需要补齐。且由于含有额外的加减法运算，只有当 $n=4096$ 时，该方法才更优（朴素 24s，分治13s）。

??? 参考代码
	```cpp
	#include<bits/stdc++.h>
	#define uint unsigned int
	using namespace std;
	const int n=4096;
	uint* add(uint* a,uint* b,uint pw){
		uint size=1<<pw;
		uint *res=new uint[size*size];
		for(uint i=0;i<size*size;++i)
			res[i]=a[i]+b[i];
		return res;
	}
	uint* sub(uint* a,uint* b,uint pw){
		uint size=1<<pw;
		uint *res=new uint[size*size];
		for(uint i=0;i<size*size;++i)
			res[i]=a[i]-b[i];
		return res;
	}
	uint* mul(uint* a,uint* b,uint pw){
		uint size=1<<pw;
		uint *res=new uint[size*size];
		memset(res,0,sizeof(uint)*size*size);
		if(pw<=5){
			for(uint i=0;i<size;++i)
			for(uint k=0;k<size;++k)
			for(uint j=0;j<size;++j)
				res[(i<<pw)|j]+=a[(i<<pw)|k]*b[(k<<pw)|j];
			return res;
		}
		uint half=1<<(pw-1);
		uint *A00=new uint[half*half];
		uint *A01=new uint[half*half];
		uint *A10=new uint[half*half];
		uint *A11=new uint[half*half];
		uint *B00=new uint[half*half];
		uint *B01=new uint[half*half];
		uint *B10=new uint[half*half];
		uint *B11=new uint[half*half];
		for(uint i=0;i<half;++i)
		for(uint j=0;j<half;++j){
			A00[(i<<(pw-1))|j]=a[(i<<pw)|j];
			B00[(i<<(pw-1))|j]=b[(i<<pw)|j];
			A01[(i<<(pw-1))|j]=a[(i<<pw)|(j+half)];
			B01[(i<<(pw-1))|j]=b[(i<<pw)|(j+half)];
			A10[(i<<(pw-1))|j]=a[((i+half)<<pw)|j];
			B10[(i<<(pw-1))|j]=b[((i+half)<<pw)|j];
			A11[(i<<(pw-1))|j]=a[((i+half)<<pw)|(j+half)];
			B11[(i<<(pw-1))|j]=b[((i+half)<<pw)|(j+half)];
		}
		uint *tmpa=0,*tmpb=0;
		tmpb=sub(B01,B11,pw-1);
		uint *P1=mul(A00,tmpb,pw-1);
		delete[] tmpb;
		tmpa=add(A00,A01,pw-1);
		uint *P2=mul(tmpa,B11,pw-1);
		delete[] tmpa;
		tmpa=add(A10,A11,pw-1);
		uint *P3=mul(tmpa,B00,pw-1);
		delete[] tmpa;
		tmpb=sub(B10,B00,pw-1);
		uint *P4=mul(A11,tmpb,pw-1);
		delete[] tmpb;
		tmpa=add(A00,A11,pw-1);
		tmpb=add(B00,B11,pw-1);
		uint *P5=mul(tmpa,tmpb,pw-1);
		delete[] tmpa;
		delete[] tmpb;
		tmpa=sub(A01,A11,pw-1);
		tmpb=add(B10,B11,pw-1);
		uint *P6=mul(tmpa,tmpb,pw-1);
		delete[] tmpa;
		delete[] tmpb;
		tmpa=sub(A00,A10,pw-1);
		tmpb=add(B00,B01,pw-1);
		uint *P7=mul(tmpa,tmpb,pw-1);
		delete[] tmpa;
		delete[] tmpb;
		delete[] A00;
		delete[] A01;
		delete[] A10;
		delete[] A11;
		delete[] B00;
		delete[] B01;
		delete[] B10;
		delete[] B11;
		tmpa=add(P5,P4,pw-1);
		tmpb=sub(tmpa,P2,pw-1);
		delete[] tmpa;
		tmpa=add(tmpb,P6,pw-1);
		for(uint i=0;i<half;++i)
		for(uint j=0;j<half;++j)
			res[(i<<pw)|j]=tmpa[(i<<(pw-1))|j];
		delete[] tmpa;
		delete[] tmpb;
		tmpa=add(P1,P2,pw-1);
		for(uint i=0;i<half;++i)
		for(uint j=0;j<half;++j)
			res[(i<<pw)|(j+half)]=tmpa[(i<<(pw-1))|j];
		delete[] tmpa;
		tmpa=add(P3,P4,pw-1);
		for(uint i=0;i<half;++i)
		for(uint j=0;j<half;++j)
			res[((i+half)<<pw)|j]=tmpa[(i<<(pw-1))|j];
		delete[] tmpa;
		tmpa=add(P1,P5,pw-1);
		tmpb=sub(tmpa,P3,pw-1);
		delete[] tmpa;
		tmpa=sub(tmpb,P7,pw-1);
		for(uint i=0;i<half;++i)
		for(uint j=0;j<half;++j)
			res[((i+half)<<pw)|(j+half)]=tmpa[(i<<(pw-1))|j];
		delete[] tmpa;
		delete[] tmpb;
		delete[] P1;
		delete[] P2;
		delete[] P3;
		delete[] P4;
		delete[] P5;
		delete[] P6;
		delete[] P7;
		return res;
	}
	template<uint size>
	struct mat{
		uint a[size][size];
		mat(){
			memset(a,0,sizeof(a));
		}
		uint* operator [](uint n){
			return a[n];
		}
		friend mat operator +(mat a,mat b){
			mat res;
			uint pw=0;
			while(1<<pw<size)
				++pw;
			uint sz=1<<pw;
			uint *A=new uint[sz*sz];
			uint *B=new uint[sz*sz];
			for(uint i=0;i<size;++i)
			for(uint j=0;j<size;++j){
				A[(i<<pw)|j]=a[i][j];
				B[(i<<pw)|j]=b[i][j];
			}
			uint *C=add(A,B,pw);
			for(uint i=0;i<size;++i)
			for(uint j=0;j<size;++j)
				res[i][j]=C[(i<<pw)|j];
			delete[] A;
			delete[] B;
			delete[] C;
			return res;
		}
		friend mat operator -(mat a,mat b){
			mat res;
			uint pw=0;
			while(1<<pw<size)
				++pw;
			uint sz=1<<pw;
			uint *A=new uint[sz*sz];
			uint *B=new uint[sz*sz];
			for(uint i=0;i<size;++i)
			for(uint j=0;j<size;++j){
				A[(i<<pw)|j]=a[i][j];
				B[(i<<pw)|j]=b[i][j];
			}
			uint *C=sub(A,B,pw);
			for(uint i=0;i<size;++i)
			for(uint j=0;j<size;++j)
				res[i][j]=C[(i<<pw)|j];
			delete[] A;
			delete[] B;
			delete[] C;
			return res;
		}
		friend mat operator *(mat a,mat b){
			mat res;
			uint pw=0;
			while(1u<<pw<size)
				++pw;
			uint sz=1<<pw;
			uint *A=new uint[sz*sz];
			uint *B=new uint[sz*sz];
			for(uint i=0;i<size;++i)
			for(uint j=0;j<size;++j){
				A[(i<<pw)|j]=a[i][j];
				B[(i<<pw)|j]=b[i][j];
			}
			uint *C=mul(A,B,pw);
			for(uint i=0;i<size;++i)
			for(uint j=0;j<size;++j)
				res[i][j]=C[(i<<pw)|j];
			delete[] A;
			delete[] B;
			delete[] C;
			return res;
		}
	};
	mat<n>a,b,c;
	int main(){
		#ifdef alarm5854
		freopen("matrix.in","r",stdin);
		freopen("matrix.out","w",stdout);
		#endif
		uint seedA,seedB;//种子用于生成矩阵，以方便比较效率
		scanf("%u%u",&seedA,&seedB);
		for(uint i=0;i<n;++i)
		for(uint j=0;j<n;++j)
			a[i][j]=((i|j)+j)^seedA;
		for(uint i=0;i<n;++i)
		for(uint j=0;j<n;++j)
			b[i][j]=((i&j)+i)^seedB;
		c=a*b;
		for(uint i=0;i<n;++i){
			uint sum=0;
			for(uint j=0;j<n;++j)
				sum^=c[i][j];
			printf("%u\n",sum);
		}
		return 0;
	}
	```
## 快速傅里叶变换(FFT)
还没讲。