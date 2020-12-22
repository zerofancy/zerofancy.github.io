title: 二叉树ADT的实现
author: 归零幻想
publishDate: 2019-05-14
editDate: 2019-05-14
tags: [数据结构, 算法, C++, 树, 遍历]

<!--config-->

假设二叉数的数据元素为字符，采用二叉链式存储结构。请编码实现二叉树ADT，其中包括创建二叉树、遍历二叉树（深度、广度）、求二叉树的深度（高度）、计算二叉树的元素个数、计算二叉树的叶子数、二叉树的格式输出等。

## Question
假设二叉数的数据元素为字符，采用二叉链式存储结构。请编码实现二叉树ADT，其中包括创建二叉树、遍历二叉树（深度、广度）、求二叉树的深度（高度）、计算二叉树的元素个数、计算二叉树的叶子数、二叉树的格式输出等。

根据输入的符号，执行相应的操作。如下：

- C：根据完全前序序列创建二叉树，#表示空结点（空子树）；输入：二叉树的完全前序序列，创建成功输出 `Created success!`
- H：求二叉树的高度；   输出： `Height=高度`
- L：计算二叉树的叶子数；输出：`Leaves=叶子个数`
- N：计算二叉树中元素总个数；输出：`Nodes=结点个数`
- 1：先序遍历二叉树；输出：`Preorder is:序列 .`
- 2：中序遍历二叉树；输出：`Inorder is:序列 .`
- 3：后序遍历二叉树；输出：`Postorder is:序列 .`
- 4：广度遍历二叉树；输出：`BFSorder is:序列 .`
- F：查找值为x的结点个数；输出：`The count of x is 个数 .`
- P：以目录缩格文本形式输出所有节点。输出：`The tree is:`（换行，下面各行是输出的二叉树）
- X：退出

<!--summary-->

## Example
### Input
```
C
ABC##DE#G##F###
H
L
N
1
2
3
4
F
A
P
X
```

### Output
```
Created success!
Height=5
Leaves=3
Nodes=7
Preorder is:A B C D E G F .
Inorder is:C B E G D F A .
Postorder is:C G E F D B A .
BFSorder is:A B C D E F G .
The count of A is 1
The tree is:
A
  B
    C
    D
      E
        G
      F
```

## 分析
二叉树不多说，数据结构的基本内容。这题主要就是麻烦，要求的操作比较多，一看肯定不卡时间，递归走起。

> 递归一时爽，一直递归一直爽。

在思路清晰的情况下，递归大大降低了编码的复杂程度，于是创建递归，求叶子数递归，求高度递归，总个数递归，查找递归，目录形式输出递归，深度遍历递归，广度……咳咳，好吧，这个不用递归，要用个队列。

不让用`STL`，队列也只好手写。

我一开始写程序`Leaves=`写成了`Leaf=`，这鬼畜的错误半天没检查出来。顺便推荐个文本差异对比工具吧，这样找起来也容易：[在线文本差异对比](http://www.jq22.com/textDifference "在线文本差异对比")

算上空行都300行了，真是够麻烦的。

## Answer
```c++
#include <iostream>

using namespace std;
template<class T>
class QueueNode{
public:
    T data;
    QueueNode*link=NULL;

    QueueNode(){
        ;
    }

    QueueNode(T data){
        this->data=data;
    }
};

template<class T>
class Queue{
private:
    QueueNode<T>*tail=NULL;
    QueueNode<T>*head=NULL;
public:
    bool isEmpty(){
        return head==NULL;
    }

    ~Queue(){
        while(head!=NULL){
            QueueNode<T>*p=head->link;
            delete head;
            head=p;
        }
    }

    void enQueue(T data){
        if(tail==NULL){
            head=tail=new QueueNode<T>(data);
            return;
        }
        tail->link=new QueueNode<T>(data);
        tail=tail->link;
    }

    T deQueue(){
        if(isEmpty()){
            return NULL;
        }
        T res=head->data;
        QueueNode<T>*tmp=head;
        head=head->link;
        delete tmp;
        if(head==NULL){
            tail=NULL;
        }
        return res;
    }
};

class Tree;

class Node{
friend class Tree;
private:
    char data='\0';
    Node*lChild=NULL;
    Node*rChild=NULL;
public:
    Node(){
        ;
    }

    Node(char data){
        this->data=data;
    }
};

class Tree{
private:
    Node*root=NULL;
    void createTree(Node*&p,string&str,int&id){
        if(id>=str.size()){
            return;
        }
        if(str.at(id)=='#'){
            id++;
            return;
        }
        p=new Node();
        p->data=str.at(id);
        id++;
        createTree(p->lChild,str,id);
        createTree(p->rChild,str,id);
    }

    size_t getHeight(Node*p){
        size_t lHeight;
        size_t rHeight;
        if(p->lChild==NULL){
            lHeight=0;
        }else{
            lHeight=getHeight(p->lChild);
        }
        if(p->rChild==NULL){
            rHeight=0;
        }else{
            rHeight=getHeight(p->rChild);
        }
        return max(lHeight,rHeight)+1;
    }

    size_t getLeaves(Node*p){
        if(p==NULL){
            return 0;
        }
        size_t res=0;
        if(p->lChild==NULL&&p->rChild==NULL){
            res++;
        }
        res+=getLeaves(p->lChild);
        res+=getLeaves(p->rChild);
        return res;
    }

    size_t getNodes(Node*p){
        if(p==NULL){
            return 0;
        }
        return getNodes(p->lChild)+getNodes(p->rChild)+1;
    }

    void visit1(Node*p){
        if(p==NULL){
            return;
        }
        cout<<p->data<<' ';
        visit1(p->lChild);
        visit1(p->rChild);
    }

    void visit2(Node*p){
        if(p==NULL){
            return;
        }
        visit2(p->lChild);
        cout<<p->data<<' ';
        visit2(p->rChild);
    }

    void visit3(Node*p){
        if(p==NULL){
            return;
        }
        visit3(p->lChild);
        visit3(p->rChild);
        cout<<p->data<<' ';
    }

    void delNode(Node*p){
        if(p==NULL){
            return;
        }
        delNode(p->lChild);
        delNode(p->rChild);
        delete p;
    }

    int search(char c,Node*p){
        if(p==NULL){
            return 0;
        }
        int res=p->data==c?1:0;
        return search(c,p->lChild)+search(c,p->rChild)+res;
    }

    void indexTree(Node*p,int depth){
        if(p==NULL){
            return;
        }
        for(int i=0;i<depth;i++){
            cout<<' '<<' ';
        }
        cout<<p->data<<endl;
        indexTree(p->lChild,depth+1);
        indexTree(p->rChild,depth+1);
    }
public:
    Tree(string str){
        int id=0;
        createTree(root,str,id);
    }

    size_t getHeight(){
        return getHeight(root);
    }

    size_t getLeaves(){
        return getLeaves(root);
    }

    size_t getNodes(){
        return getNodes(root);
    }

    void visit1(){
        cout<<"Preorder is:";
        visit1(root);
        cout<<'.'<<endl;
    }

    void visit2(){
        cout<<"Inorder is:";
        visit2(root);
        cout<<'.'<<endl;
    }

    void visit3(){
        cout<<"Postorder is:";
        visit3(root);
        cout<<'.'<<endl;
    }

    void visitBFS(){
        Queue<Node*>*q=new Queue<Node*>();
        q->enQueue(root);
        while(!q->isEmpty()){
            Node*tmp=q->deQueue();
            cout<<tmp->data<<' ';
            if(tmp->lChild!=NULL){
                q->enQueue(tmp->lChild);
            }
            if(tmp->rChild!=NULL){
                q->enQueue(tmp->rChild);
            }
        }
        delete q;
    }

    int search(char c){
        return search(c,root);
    }

    void indexTree(){
        indexTree(root,0);
    }

    ~Tree(){
        delNode(root);
    }
};

int main()
{
    char Op;
    Tree*tree;
    while(cin>>Op){
        string str;
        switch(Op){
        case 'C':
            cin>>str;
            tree=new Tree(str);
            cout<<"Created success!"<<endl;
            break;
        case 'H':
            cout<<"Height="<<tree->getHeight()<<endl;
            break;
        case 'L':
            cout<<"Leaves="<<tree->getLeaves()<<endl;
            break;
        case 'N':
            cout<<"Nodes="<<tree->getNodes()<<endl;
            break;
        case '1':
            tree->visit1();
            break;
        case '2':
            tree->visit2();
            break;
        case '3':
            tree->visit3();
            break;
        case '4':
            cout<<"BFSorder is:";
            tree->visitBFS();
            cout<<'.'<<endl;
            break;
        case 'F':
            char x;
            cin>>x;
            cout<<"The count of "<<x<<" is "<<tree->search(x)<<endl;
            break;
        case 'P':
            cout<<"The tree is:"<<endl;
            tree->indexTree();
            break;
        }
    }
    delete tree;
    return 0;
}

```
