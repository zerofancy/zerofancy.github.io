title: 数据结构与算法 C++编程能力评估
author: 归零幻想
publishDate: 2019-03-28
editDate: 2019-03-28
tags: [文章, 关键词]

<!--config-->

## 题目1

向类SeqList中插入数据，请根据main函数中的调用，完成Insert和output函数。

<!--summary-->

- Input

```
    5
    1 2 3 4 5
    2 9
```
- Output

```
1 2 3 4 5
1 2 9 3 4 5
```

- Answer

```cpp
#include<iostream>
#include<stdlib.h>
using namespace std;
class SeqList
{
private:
    int * data;
    int last; // index of the last element
public:
    SeqList ( int sz );
    ~SeqList ( )
    {
        delete [ ] data;
    }
    void input ();
    void output() ;
    void Insert   ( const int &x, int i);
}  ;
SeqList::SeqList ( int sz )
{
    if ( sz > 0 )
    {
        data = new int[sz];
        last = -1;
    }
}
void SeqList:: input()
{
    cin >>last;
    for (int i=0; i<last; i++)
        cin>>data [i];
    last--;
}

void SeqList::output()
{
    bool fi=false;
    for(int i=0; i<=last; i++)
    {
        if(!fi)
        {
            fi=!fi;
            cout<<data[i];
        }else{
            cout<<' '<<data[i];
        }
    }
    cout<<endl;
}

void SeqList::Insert(const int& x, int i)
{
    for(int i1=last;i1>=i;i1--){
        data[i1+1]=data[i1];
    }
    data[i]=x;
    last++;
}

int main()
{
    SeqList myList(50);
    myList.input();
    myList.output();
    int where,value;
    cin >>where;
    cin >>value;
    myList.Insert(value,where);
    myList.output ();
    return 1;
}
```

## 题目2

已知一块靶场大小为m*n（m行n列），每个点上都有一面旗子，我们的炮兵发射炮弹，炮弹一次只能炸掉一面旗子，每次炮弹的落点坐标都有记录，问你几炮过后，靶场还剩下几面旗子呢？

- Input

有若干行，第一行为两个整数m,n，以空格分隔，都是整数，且1<=m<=n<=100；

接下来有若干行，每行有两个整数x,y，代表炮弹落点的坐标（行、列坐标，编号从0开始）。

```
3 3
1 1
1 1
99 99
```

- Output

一个整数，代表最后靶场上剩余的旗子数。

提示：炮弹可能打在同一个位置，也可能脱靶，但保证在100*100的范围内。

```
8
```

- Answer

```cpp
#include <iostream>
#define N 105

using namespace std;

int main()
{
    bool data[N][N];
    for(int i=0;i<N;i++){
        for(int i2=0;i2<N;i2++){
            data[i][i2]=false;
        }
    }
    int m,n;
    cin>>m>>n;
    int res=m*n;
    int x,y;
    while(cin>>x>>y){
        data[x][y]=true;
    }
    for(int i=0;i<m;i++){
        for(int i2=0;i2<n;i2++){
            if(data[i][i2]){
                res--;
            }
        }
    }
    cout<<res<<endl;
    return 0;
}

```

## 题目3

买房面积不要求太大，环境不要求太好，只要单价够便宜就已经很满足啦~现在挑选了一些房源，请你帮忙选一选，哪个最适合呢？小本本上记录了房子的名称（字母和数字组成，无空格）、面积和总价，你来帮编个程序自动计算一下吧。

- Input

第一行为一个整数N(1<=N<=100)，表示接下来有N套房源信息；

接下来有N行，每行包括房源名称(不超过100个字符)、面积和总价（double类型，小数点后保留两位），以空格分隔。

```
3
Tangdaowan0101 50 50
Jiangshan1314 49 50
Jinshatan1111 51 50
```

- Output

```
Jinshatan1111 51.00 50.00
```

仅一行，为最适合的房源信息，以空格分隔，末尾换行。

测试用例保证没有单价重复的情况。

- Answer

```cpp
#include <stdio.h>
#include <string.h>
#include<limits.h>
typedef struct
{
    char name[101];
    double area;
    double sum;
    double avg;
} House;

int Gethouse(House*h,int n){
    double res=INT_MAX;
    int res_num=0;
    for(int i=0;i<n;i++){
        h[i].avg=h[i].sum/h[i].area;
        if(h[i].avg<res){
            res=h[i].avg;
            res_num=i;
       }
    }
    return res_num;
}

int main()
{
    int n,i,flag = 0;//n有几行
    scanf("%d",&n);
    House h[n];
    for (i = 0; i < n; i ++)
      scanf("%s%lf%lf",h[i].name,&h[i].area,&h[i].sum);
    flag= Gethouse(h,n);
    printf("%s %.2f %.2f\n",h[flag].name, h[flag].area, h[flag].sum);
    return 0;
}

```

## 题目4

实现链表的输入（已实现）、输出和删除成员函数。

输入时，根据endtag确定是否结束输入；

删除时，根据下标（从0开始）。

注意，输入的顺序跟存放的顺序以及输出的顺序是相反的。

- Input

```
1 2 3 4 5 0
2
```

- Output

```
5 4 3 2 1
5 4 2 1
```

- Answer

```cpp
#include "stdlib.h"
#include "iostream"
#include<cstdlib>
using namespace std ;

class List; //前视定义,否则友元无法定义
class LinkNode
{
    friend  List; //链表结点类的定义
private:
    LinkNode *link;
    int data;
public:
    LinkNode(const int & item, LinkNode *ptr = NULL)
    {
        data=item;
        link=ptr;
    }
    LinkNode (LinkNode *ptr = NULL)
    {
        link=ptr;
    }
    ~LinkNode() {};

};

class List
{
    //单链表类的定义
private:
    LinkNode *first; //指向首结点的指针
public:
    List ()
    {
        first = new LinkNode ();   // 带头结点
    }
    ~List ()
    {
        MakeEmpty();   //析构函数
    }
    void MakeEmpty ( );      //链表置空
    int Remove ( int i );
    void input(int  endTag);
    void output();
};
void List:: MakeEmpty ( )
{
    LinkNode *q;
    while (  first->link != NULL )
    {
        q = first->link;
        first->link = q->link;
        delete q;
    }
};

void List :: input (int endTag)
{
    LinkNode  *newnode;
    int val;
    cin>>val;
    while(val!=endTag)
    {
        newnode=new LinkNode (val);
        newnode->link=first->link;
        first->link=newnode;
        cin>>val;
    }
}

int List::Remove(int i)
{
    LinkNode*tmp=first;
    for(int i2=0;i2<i;i2++){
        tmp=tmp->link;
    }
    LinkNode*tmp1=tmp->link;
    tmp->link=tmp->link->link;
    delete(tmp1);
}

void List::output()
{
    int fi=0;
    LinkNode*tmp=first->link;
    while(tmp!=NULL)
    {
        if(fi++)
        {
            cout<<' '<<tmp->data;
        }
        else
        {
            cout<<tmp->data;
        }
        tmp=tmp->link;
    }
    cout<<endl;
}

int main()
{
    List l;
    l.input(0); //0为输入的结束数字
    l.output ();
    int index;
    cin>>index; //要删除的元素的下标，下标从0 开始
    l.Remove(index);
    l.output (); //删除后输出
    return 0;
}

```
