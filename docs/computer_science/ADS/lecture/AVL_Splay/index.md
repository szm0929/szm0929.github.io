# AVL 与 Splay
这是本课程的第一章内容，主要介绍了 AVL 树与 Splay 树，以及由 Splay 的复杂度分析引出的摊还分析。下面是这三部分内容的介绍。
## 前置知识：旋转
旋转，是一种改变树的形态但不改变二叉搜索树的性质的一种旋转方式，基础旋转操作可分为左旋与右旋。
### 左旋
左旋，就是把一个节点往左儿子方向进行旋转、让右儿子成为新的父节点的操作，原理图为：  
![](rotL.jpg)  
??? 参考代码
    ```cpp
    tree rotL(tree p){
        tree x=p->rs;
        p->rs=x->ls;
        x->ls=p;
        upd(p);//更新节点信息的函数
        upd(x);
        return x;
    }
    ```
### 右旋
右旋，就是把一个节点往右儿子方向进行旋转、让左儿子成为新的父节点的操作，原理图为：  
![](rotR.jpg)  
??? 参考代码
    ```cpp
    tree rotR(tree p){
        tree x=p->ls;
        p->ls=x->rs;
        x->rs=p;
        upd(p);
        upd(x);
        return x;
    }
    ```
## AVL 树
AVL 树是一种基于高度平衡的一种平衡树，其左右儿子的高度差的绝对值不会超过 $1$（下面记作平衡因子 $\text{HB}(p)$）。基于这一性质，我们可以求出一个高度为 $h$ 的 AVL 树至少包含了多少个节点，以下是计算过程。
??? 计算过程
    设高度为 $h$ 的 AVL 树至少包含 $a_h$ 个节点，则显然有 $a_0=0,a_1=1$。  
    对于更高的 AVL 树，设其高度为 $h$，则其两个儿子的高度最坏情况为一个 $h-1$ 和一个 $h-2$，即 $a_h=a_{h-1}+a_{h-2}+1$。  
    两边同时加一，有 $(a_h+1)=(a_{h-1}+1)+(a_{h-2}+1)$，且 $(a_0+1)=1=\text{Fib}(1),(a_1+1)=2=\text{Fib}(2)$。
    故 $a_h=\text{Fib}(h+1)-1$
由计算过程可知，一个含 $n$ 个节点的 AVL 树，其树高是 $O(\log n)$ 级别的。

接下来将介绍其插入、删除以及其他拓展操作（AVL 树的考点只有插入，介绍其他部分只是为了保持完整性）。先写下其结构体定义，以方便后续理解。  
??? 结构体定义代码
    ```cpp
    struct node;
    typedef node* tree;
    struct node{
        int val,h,sz,cnt;
        tree ls,rs;
        node(int val=0,int h=1,int sz=1,int cnt=1,tree ls=0,tree rs=0):val(val),h(h),sz(sz),cnt(cnt),ls(ls),rs(rs){}
    };
    int getH(tree p){
        if(!p)
            return 0;
        return p->h;
    }
    int getHB(tree p){
        if(!p)
            return 0;
        return getH(p->ls)-getH(p->rs);
    }
    int getSize(tree p){
        if(!p)
            return 0;
        return p->sz;
    }
    void upd(tree p){
        if(!p)
            return;
        p->h=max(getH(p->ls),getH(p->rs))+1;
        p->sz=getSize(p->ls)+getSize(p->rs)+p->cnt;
    }
    ```

### 插入
插入过程与常规二叉搜索树相同，然而当插入完成后，其性质可能会被破坏（不过尽管如此，$|\text{HB}(p)|\le 2$ 仍然满足），因此需要做出调整，这里存在四种情况。
#### 左左
左儿子的高度更高，且左儿子的左儿子的高度更高，即 $\text{HB}(p)=2,\text{HB}(l)\ge 0$，此时情况如图所示：  
![](LL1.jpg)  
此时只需要对节点 $p$ 进行右旋，即可再次平衡：  
![](LL2.jpg)
#### 左右
左儿子的高度更高，但左儿子的右儿子的高度更高，即 $\text{HB}(p)=2,\text{HB}(l)<0$，此时情况如图所示：  
![](LR1.jpg)  
此时需要对节点 $l$ 进行左旋，以转化为**左左**这一情况：
![](LR2.jpg)
#### 右右
右儿子的高度更高，且右儿子的右儿子的高度更高，即 $\text{HB}(p)=-2,\text{HB}(l)\le 0$，此时情况如图所示：  
![](RR1.jpg)  
此时只需要对节点 $p$ 进行左旋，即可再次平衡：  
![](RR2.jpg)
#### 右左
右儿子的高度更高，但右儿子的左儿子的高度更高，即 $\text{HB}(p)=-2,\text{HB}(l)>0$，此时情况如图所示：  
![](RL1.jpg)  
此时需要对节点 $r$ 进行右旋，以转化为**右右**这一情况：
![](RL2.jpg)

至此，插入产生的四种不平衡情况已介绍完成。  
??? 参考代码
    ```cpp
    tree ins(tree p,int x){
        if(!p)
            return new node(x);
        if(x<p->val){
            p->ls=ins(p->ls,x);
            if(getHB(p)>1){
                if(getHB(p->ls)<0)
                    p->ls=rotL(p->ls);
                p=rotR(p);
                return p;
            }
        }
        else if(x>p->val){
            p->rs=ins(p->rs,x);
            if(getHB(p)<-1){
                if(getHB(p->rs)>0)
                    p->rs=rotR(p->rs);
                p=rotL(p);
                return p;
            }
        }
        else
            ++p->cnt;
        upd(p);
        return p;
    }
    ```
### 删除
删除过程也与常规二叉搜索树相同，即当儿子数量不为 $2$ 时直接删除，否则找到后继再删除。且删除完成后遇到的情况仍然是上述的四种情况之一，因此这里直接给出参考代码了。!mask[不知道为什么 AVL 树的删除不考，反而考红黑树的删除。]  
??? 参考代码
    ```cpp
    tree del(tree p,int x){
        if(!p)
            return p;
        if(x<p->val){
            p->ls=del(p->ls,x);
            if(getHB(p)<-1){
                if(getHB(p->rs)>0)
                    p->rs=rotR(p->rs);
                p=rotL(p);
            }
        }
        else if(x>p->val){
            p->rs=del(p->rs,x);
            if(getHB(p)>1){
                if(getHB(p->ls)<0)
                    p->ls=rotL(p->ls);
                p=rotR(p);
            }
        }
        else{
            if(p->cnt>1){
                --p->cnt;
                upd(p);
                return p;
            }
            if(!p->ls||!p->rs){
                tree t=p;
                if(!p->ls)
                    p=p->rs;
                else
                    p=p->ls;
                delete t;
                return p;
            }
            else{
                tree t=p->rs;
                while(t->ls)
                    t=t->ls;
                p->val=t->val;
                p->cnt=t->cnt;
                t->cnt=1;
                p->rs=del(p->rs,t->val);
                if(getHB(p)>1){
                    if(getHB(p->ls)<0)
                        p->ls=rotL(p->ls);
                    p=rotR(p);
                }
            }
        }
        upd(p);
        return p;
    }
    ```
### 拓展操作
与常规二叉搜索树完全一致。

至此，AVL 树已全部介绍完成，我将给出可以通过洛谷 P3369 的完整代码，有轻度封装。   
??? 参考代码
    ```cpp
    #include<bits/stdc++.h>
    using namespace std;
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
    struct node;
    typedef node* tree;
    struct node{
        int val,h,sz,cnt;
        tree ls,rs;
        node(int val=0,int h=1,int sz=1,int cnt=1,tree ls=0,tree rs=0):val(val),h(h),sz(sz),cnt(cnt),ls(ls),rs(rs){}
    };
    int getH(tree p){
        if(!p)
            return 0;
        return p->h;
    }
    int getHB(tree p){
        if(!p)
            return 0;
        return getH(p->ls)-getH(p->rs);
    }
    int getSize(tree p){
        if(!p)
            return 0;
        return p->sz;
    }
    void upd(tree p){
        if(!p)
            return;
        p->h=max(getH(p->ls),getH(p->rs))+1;
        p->sz=getSize(p->ls)+getSize(p->rs)+p->cnt;
    }
    tree rotL(tree p){
        tree x=p->rs;
        p->rs=x->ls;
        x->ls=p;
        upd(p);
        upd(x);
        return x;
    }
    tree rotR(tree p){
        tree x=p->ls;
        p->ls=x->rs;
        x->rs=p;
        upd(p);
        upd(x);
        return x;
    }
    tree ins(tree p,int x){
        if(!p)
            return new node(x);
        if(x<p->val){
            p->ls=ins(p->ls,x);
            if(getHB(p)>1){
                if(getHB(p->ls)<0)
                    p->ls=rotL(p->ls);
                p=rotR(p);
                return p;
            }
        }
        else if(x>p->val){
            p->rs=ins(p->rs,x);
            if(getHB(p)<-1){
                if(getHB(p->rs)>0)
                    p->rs=rotR(p->rs);
                p=rotL(p);
                return p;
            }
        }
        else
            ++p->cnt;
        upd(p);
        return p;
    }
    tree del(tree p,int x){
        if(!p)
            return p;
        if(x<p->val){
            p->ls=del(p->ls,x);
            if(getHB(p)<-1){
                if(getHB(p->rs)>0)
                    p->rs=rotR(p->rs);
                p=rotL(p);
            }
        }
        else if(x>p->val){
            p->rs=del(p->rs,x);
            if(getHB(p)>1){
                if(getHB(p->ls)<0)
                    p->ls=rotL(p->ls);
                p=rotR(p);
            }
        }
        else{
            if(p->cnt>1){
                --p->cnt;
                upd(p);
                return p;
            }
            if(!p->ls||!p->rs){
                tree t=p;
                if(!p->ls)
                    p=p->rs;
                else
                    p=p->ls;
                delete t;
                return p;
            }
            else{
                tree t=p->rs;
                while(t->ls)
                    t=t->ls;
                p->val=t->val;
                p->cnt=t->cnt;
                t->cnt=1;
                p->rs=del(p->rs,t->val);
                if(getHB(p)>1){
                    if(getHB(p->ls)<0)
                        p->ls=rotL(p->ls);
                    p=rotR(p);
                }
            }
        }
        upd(p);
        return p;
    }
    int rnk(tree p,int x){
        if(!p)
            return 1;
        if(x<p->val)
            return rnk(p->ls,x);
        if(x==p->val)
            return getSize(p->ls)+1;
        return getSize(p->ls)+p->cnt+rnk(p->rs,x);
    }
    int val(tree p,int k){
        if(!p||k<=0||k>getSize(p))
            return -1;
        if(k<=getSize(p->ls))
            return val(p->ls,k);
        if(k<=getSize(p->ls)+p->cnt)
            return p->val;
        return val(p->rs,k-getSize(p->ls)-p->cnt);
    }
    int pre(tree p,int x){
        if(!p)
            return -1;
        if(x<=p->val)
            return pre(p->ls,x);
        int t=pre(p->rs,x);
        if(t==-1)
            return p->val;
        return t;
    }
    int suf(tree p,int x){
        if(!p)
            return -1;
        if(x>=p->val)
            return suf(p->rs,x);
        int t=suf(p->ls,x);
        if(t==-1)
            return p->val;
        return t;
    }
    struct AVL{
        tree rt;
        int root(){
            if(!rt)
                return -1;
            return rt->val;
        }
        void insert(int x){
            rt=ins(rt,x);
        }
        void erase(int x){
            rt=del(rt,x);
        }
        int rank(int x){
            return rnk(rt,x);
        }
        int kth(int k){
            return val(rt,k);
        }
        int prev(int x){
            return pre(rt,x);
        }
        int suff(int x){
            return suf(rt,x);
        }
    };
    AVL T;
    int n,opt,x;
    int main(){
        #ifdef alarm5854
        freopen("AVL.in","r",stdin);
        freopen("AVL.out","w",stdout);
        #endif
        n=read();
        for(int i=1;i<=n;++i){
            opt=read();
            x=read();
            if(opt==1)
                T.insert(x);
            else if(opt==2)
                T.erase(x);
            else if(opt==3)
                printf("%d\n",T.rank(x));
            else if(opt==4)
                printf("%d\n",T.kth(x));
            else if(opt==5)
                printf("%d\n",T.prev(x));
            else
                printf("%d\n",T.suff(x));
        }
        return 0;
    }
    ```
!mask[这不比OIwiki给出的过度封装且长达904行的代码强多了]