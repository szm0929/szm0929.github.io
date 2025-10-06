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