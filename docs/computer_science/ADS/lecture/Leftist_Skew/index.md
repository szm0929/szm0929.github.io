# 左偏堆与斜堆
普通的二叉堆虽然实现简单，但是在合并的时候复杂度很高，为此，我们需要维护一种能够快速合并的堆。
??? warning
	为了方便介绍两种堆，在介绍部分采用指针写法。然而在一般使用中，指针写法很容易出现问题，故参考代码为数组版本。

	~~本质上是因为本人用指针无法通过P2713~~
## 左偏堆
先引入一个概念：$dist/Npl$。对于任意一个节点，它的 $dist$ 为这个节点到没有两个孩子的节点的最短距离。对叶节点、只有一个孩子的节点，$dist$ 都为 $0$。

那么，左偏堆的定义如下：

- 满足二叉堆的性质（与左右儿子之间的大小关系）
- 对任意节点，其左孩子的 $dist$ 一定大于等于右孩子的 $dist$

根据该定义，左偏堆具有以下性质：

- 任意节点的 $dist$ = 其右孩子的 $dist + 1$（如果没有右孩子，则为 $0$）。
- 若根节点的 $dist$ 为 $N$，则前 $N+1$ 层为满二叉树，整个左偏堆至少有 $2^{N+1}-1$ 个节点。
- 若整个左偏堆共 $N$ 个节点，则右侧路径最多有 $\lfloor \log(N+1) \rfloor$ 个节点。

左偏堆最右侧的路径尽可能短，合并时只沿右侧路径合并，时间复杂度低。

其节点的定义如下：

??? 参考代码
	```cpp
	struct node;
	typedef node* tree;
	struct node{
		int val,dist;
		tree ls,rs;
		node(int val=0,int dist=0,tree ls=0,tree rs=0):val(val),dist(dist),ls(ls),rs(rs){}
	};
	int getDist(tree p){
		if(!p)
			return -1;
		return p->dist;
	}
	```
### 合并
#### 递归合并
其步骤为：

- 将已经合并的顶点（记为 $o$）从根较小的堆开始，沿最右侧不断下移。
- 每次有两个待合并的堆，分别为 $o$ 的右儿子和另一个左偏堆。将这两者中根较小的作为 $o$ 的右儿子。
- 检查**放在 $o$ 的右儿子**这一步是否违反左偏性质，调整并更新 $o$ 的 $dist$。
- 将 $o$ 下移，递归进行。

具体实现如下：

??? 参考代码
	```cpp
	tree merge(tree a,tree b){
		if(!a)
			return b;
		if(!b)
			return a;
		if(a->val>b->val)
			swap(a,b);
		a->rs=merge(a->rs,b);
		if(getDist(a->ls)<getDist(a->rs))
			swap(a->ls,a->rs);
		a->dist=getDist(a->rs)+1;
		return a;
	}
	```
#### 迭代合并
迭代合并中，用栈存储合并路径上的所有父节点。合并完后，回溯存储的父节点，依次调整左右儿子，使其保持左偏。其步骤为：

- 将已经合并的顶点（记为 $o$）从根较小的堆开始，沿最右侧不断下移。
- 若另一个堆小于 $o$ 的右儿子，则交换到 $o$ 右儿子的位置。
- 在栈中存储合并的父节点 $o$。
- 将 $o$ 下移，递归进行。
- 依次弹栈，维护左偏性质。

??? 参考代码
	```cpp
	tree merge(tree a,tree b){
		if(!a)
			return b;
		if(!b)
			return a;
		if(a->val>b->val)
			swap(a,b);
		tree root=a;
		stack<tree>stk;
		while(a->rs&&b) {
			if (a->rs->val>b->val)
				swap(a->rs,b);
			stk.push(a);
			a=a->rs;
		}
		if(!a->rs){
			a->rs=b;
			stk.push(a);
		}
		while(stk.size()){
			a=stk.top();
			stk.pop();
			if(getDist(a->ls)<getDist(a->rs))
				swap(a->ls,a->rs);
			a->dist=getDist(a->rs)+1;
		}
		return root;
	}
	```
### 插入
单点插入可看作原先的左偏堆和只有一个节点的新堆的合并。
### 删除最小值
直接合并左右子树即可。

至此，左偏堆介绍完成，这是可以通过洛谷P2713的代码：
??? 参考代码
	```cpp
	#include<bits/stdc++.h>
	using namespace std;
	const int N=1e6+7;
	int read(){
		int x=0,f=1;
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
	int n,m,u,v,rt[N],dis[N],ls[N],rs[N],a[N],dead[N];
	int find(int x){
		return rt[x]==x?x:rt[x]=find(rt[x]);
	}
	int merge(int x,int y){
		if(!x||!y)
			return x|y;
		if(a[x]>a[y])
			swap(x,y);
		rs[x]=merge(rs[x],y);
		if(dis[ls[x]]<dis[rs[x]])
			swap(ls[x],rs[x]);
		dis[x]=dis[rs[x]]+1;
		return x;
	}
	void pop(int x){
		dead[x]=1;
		rt[x]=rt[ls[x]]=rt[rs[x]]=merge(ls[x],rs[x]);
	}
	int main(){
		#ifdef alarm5854
		freopen("LeftistHeap.in","r",stdin);
		freopen("LeftistHeap.out","w",stdout);
		#endif
		n=read();
		for(int i=1;i<=n;++i){
			a[i]=read();
			rt[i]=i;
		}
		m=read();
		for(int i=1;i<=m;++i){
			char c;
			while(!isalpha(c=getchar()));
			if(c=='M'){
				u=read();
				v=read();
				if(dead[u]||dead[v])
					continue;
				u=find(u);
				v=find(v);
				if(u==v)
					continue;
				rt[u]=rt[v]=merge(rt[u],rt[v]);
			}
			else{
				u=read();
				if(dead[u]){
					puts("0");
					continue;
				}
				u=find(u);
				printf("%d\n",a[u]);
				pop(u);
			}
		}
		return 0;
	}
	```
## 斜堆
与左偏堆相似，只是不需要像左偏堆那样去维护 $dist$。

此时，为了提高合并效率，合并时需要交换左右儿子，以此来保证在 $M$ 次操作后时间复杂度为 $O(M\log n)$。

其节点的定义如下：

??? 参考代码
	```cpp
	struct node;
	typedef node* tree;
	struct node{
		int val;
		tree ls,rs;
		node(int val=0,tree ls=0,tree rs=0):val(val),ls(ls),rs(rs){}
	};
	```
### 合并
合并过程大体上与左偏堆相同，只是因为不维护 $dist$，故每次都需要交换左右儿子（其中一个树为空除外）。
#### 递归合并
??? 参考代码
	```cpp
	tree merge(tree a,tree b){
		if(!a)
			return b;
		if(!b)
			return a;
		if(a->val>b->val)
			swap(a,b);
		a->rs=merge(a->rs,b);
		swap(a->ls,a->rs);
		return a;
	}
	```
#### 迭代合并
??? 参考代码
	```cpp
	tree merge(tree a,tree b){
		if(!a)
			return b;
		if(!b)
			return a;
		if(a->val>b->val)
			swap(a,b);
		tree root=a;
		stack<tree>stk;
		while(a->rs&&b) {
			if (a->rs->val>b->val)
				swap(a->rs,b);
			stk.push(a);
			a=a->rs;
		}
		if(!a->rs){
			a->rs=b;
			stk.push(a);
		}
		while(stk.size()){
			a=stk.top();
			stk.pop();
			swap(a->ls,a->rs);
		}
		return root;
	}
	```
### 插入与删除最小值
与左偏堆完全一致。
### 摊还分析
首先定义**重节点**：对于一个节点，在以这个节点为根（包含该节点）的子树中，如果右子树的节点数大于等于总节点数的一半，则该节点为重节点；反之为轻节点。

定义 $\Phi_1(x)$ 为完成第 $x$ 步后一共遍历节点的个数，$\Phi_2(x)$ 为完成第 $x$ 步后重节点的个数。

斜堆合并时，只有最右路径上节点的轻重情况会改变。其他节点的轻重情况一定不变（因为直接从左子树移动到右子树，局部结构不变）。所以考虑势能变化，只要考虑最右路径的轻重节点数。

最右路径上，重节点一定变为轻节点，而轻节点不一定变为重节点（本身交换后就不一定，左儿子还可能加其他节点）。

对于右路径上的轻节点，左子树的节点较多，类似左偏堆，轻节点最多有 $\log n$ 个。

记 $l_1,h_1,l_2,h_2$ 为第一、二个堆最右路径上轻、重节点数，$h$ 表示其他位置的重节点数。

则 $\Phi_2(x-1)=h_1+h_2+h,\Phi_2(x)\le l_1+l_2+h$

$$(\Phi_1(i)-\Phi_1(i-1))+(\Phi_2(i)-\Phi_2(i-1))\le ((l_1+l_2)+(h_1+h_2))+((l_1+l_2+h)-(h_1+h_2+h))=2(l_1+l_2)=O(\log n)$$

因此，$M$ 次操作后时间复杂度为 $O(M\log n)$。

至此，斜堆介绍完成，这是可以通过洛谷P2713的代码：
??? 参考代码
	```cpp
	#include<bits/stdc++.h>
	using namespace std;
	const int N=1e6+7;
	int read(){
		int x=0,f=1;
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
	int n,m,u,v,rt[N],ls[N],rs[N],a[N],dead[N];
	int find(int x){
		return rt[x]==x?x:rt[x]=find(rt[x]);
	}
	int merge(int x,int y){
		if(!x||!y)
			return x|y;
		if(a[x]>a[y])
			swap(x,y);
		rs[x]=merge(rs[x],y);
		swap(ls[x],rs[x]);
		return x;
	}
	void pop(int x){
		dead[x]=1;
		rt[x]=rt[ls[x]]=rt[rs[x]]=merge(ls[x],rs[x]);
	}
	int main(){
		#ifdef alarm5854
		freopen("SkewHeap.in","r",stdin);
		freopen("SkewHeap.out","w",stdout);
		#endif
		n=read();
		for(int i=1;i<=n;++i){
			a[i]=read();
			rt[i]=i;
		}
		m=read();
		for(int i=1;i<=m;++i){
			char c;
			while(!isalpha(c=getchar()));
			if(c=='M'){
				u=read();
				v=read();
				if(dead[u]||dead[v])
					continue;
				u=find(u);
				v=find(v);
				if(u==v)
					continue;
				rt[u]=rt[v]=merge(rt[u],rt[v]);
			}
			else{
				u=read();
				if(dead[u]){
					puts("0");
					continue;
				}
				u=find(u);
				printf("%d\n",a[u]);
				pop(u);
			}
		}
		return 0;
	}
	```