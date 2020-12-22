title: 线性表ADT的实现
author: 归零幻想
publishDate: 2019-05-13
editDate: 2019-05-13
tags: [C++, 数据结构, 栈, 算法, 测试]

<!--config-->

# 线性表ADT的实现
[tag]:C++|编程|数据结构|栈|算法|测试

数据结构平时测试的题目开放了，这次平时测试睡过了，全宿舍都睡过了。现在回头做做题目，还是有坑点的。

## Question
假设线性表ADT的数据元素类型为正整数，采用带头结点的单链式存储结构。线性表ADT实现的大部分代码已经给出，请补充写出类的两个成员函数`insert`和`reverse`。  注意：**只需提交需要补充的函数代码，其他代码不能自己重写和修改。**

`insert`函数：在元素值从小到大有序的线性表中插入一个元素，仍然保持有序。

`reverse`函数：实现线性表元素的倒置，即将线性表中数据元素的顺序反转。

线性表元素输入时，以 `endTag` 作为结束标志。

<!--summary-->

例如输入： 
```
3 8 7 2 4 9 1 6 5 0
```

则输出：
```
9 8 7 6 5 4 3 2 1
```

预置代码如下： （其中/*   */ 部分是要补充的`insert`和`reverse`函数）
```cpp
#include<iostream>
#include<stdlib.h>
using namespace std;
typedef int ElemType;  //数据元素类型
class List; //前视定义,否则友元无法定义
//结点类定义
class LinkNode
{  friend  class List; 
   private: 
     LinkNode *link; 
     ElemType data;  
   public: 
     LinkNode (LinkNode *ptr = NULL)    {link=ptr;}
     LinkNode(const ElemType & item, LinkNode *ptr = NULL){  data=item;link=ptr;} 
     ~LinkNode(){}; 
}; 
//单链表类定义 
class List   
{  private:    
     LinkNode *first; //指向链表头结点的指针          
   public:
     List (ElemType x) { first = new LinkNode (x);}   // 带头结点
     ~List (){ MakeEmpty();}         //析构函数
     void MakeEmpty ( );      //线性表置空    
     void insert(ElemType val);   //在有序线性表中插入元素val
     void reverse();   //线性表的倒置
     void output();    //线性表的输出               
}; 
void List:: MakeEmpty ( )
 { LinkNode *q;
   while (  first->link != NULL ) 
	{ q = first->link;  //指向别摘下结点 
      first->link = q->link;//从链中摘下结点
      delete q;        //释放摘下的结点 
    }
};	
void List ::output ( )
{  LinkNode  *p=first->link; 
   while(p!=NULL)
   { if(p==first->link) cout<<p->data;
     else  cout<<" "<<p->data;
     p=p->link;
   }
   cout<<endl;
}
/*
**************************************************
请写出 insert 成员函数
**************************************************
**************************************************
请写出 reverse 成员函数
**************************************************
*/
int  main()
{   List list(0);
    ElemType endTag=0;
    ElemType val;
    //下面通过不断读入元素，插入到有序单链表中，建立从小到大的有序单链表
    cin>>val;
    while(val!=endTag) 
     {  list.insert(val);     //在有序表中插入一个元素
        cin>>val;  
      }
    list.reverse ();   //线性表倒置
    cout<<"The result is:"<<endl;
    list.output ();
    return 0;
}
```

## 分析
基础题，注意边界的判断和特例的处理。

## Answer
```cpp
/*

**************************************************

请写出 insert 成员函数

**************************************************
*/
void List::insert(ElemType val){
    LinkNode*pNew=new LinkNode(val);
    LinkNode*pPointer=first;
    while(pPointer->link!=NULL){
        if(val<pPointer->link->data){
            pNew->link=pPointer->link;
            pPointer->link=pNew;
            return;
        }
        pPointer=pPointer->link;
    }
    pPointer->link=pNew;
}
/*
**************************************************

请写出 reverse 成员函数

**************************************************

*/

void List::reverse(){
    if(first->link==NULL||first->link->link==NULL){
        return;
    }
    LinkNode*pParent=first->link;
    LinkNode*pChild=first->link->link;
    first->link->link=NULL;
    while(pChild!=NULL){
        LinkNode*pTemp=pChild->link;
        pChild->link=pParent;
        pParent=pChild;
        pChild=pTemp;
    }
    first->link=pParent;
}

```
