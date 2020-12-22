title: 堆栈ADT实现及应用
author: 归零幻想
publishDate: 2019-05-13
editDate: 2019-05-13
tags: [C++, 编程, 数据结构, 栈, 算法, 测试]

<!--config-->

> 假设栈ADT的数据元素为整数，栈ADT的实现采用顺序存储结构。现要用栈来辅助完成任意非负十进制整数到Base(Base不大于35)进制的转换。部分代码已经给出，请补充完善栈溢出处理函数和主函数。  注意：只提交需要补充的函数，其他代码不允许自己重写和修改。

栈溢出处理函数`overflowProcess`：当栈满时，将栈的空间在原来基础上扩大1倍。

主函数： 输入一个非负十进制整数n及要转换的进制`Base`，输出其转换后的进制形式，以及长度。输出格式如下：
```
($...$)10=(#...#)Base 
Length=转换进制后数的位数
```
其中`$...$`是输入的十进制数n，`#...#`是转换得到的`Base`进制数，如果转换后位码多于1位，则用大写字母A,B,...等表示，`10-A`, `11-B`,......

例如，输入：
```
1024 2
```

输出：
```
(1024)10=(10000000000)2
Length=11
```

再如，输入： 
```
25 30
```
输出：
```
(25)10=(P)30
Length=1
```

<!--summary-->

预置代码如下：
```c++
#include <iostream>
using namespace std;
#include <stdio.h>
#include <stdlib.h>
typedef  int  ElemType;
class SeqStack  
{  //顺序栈类定义
    private:     
        ElemType *elements; //数组存放栈元素
        int top;             //栈顶指示器
        int maxSize;               //栈最大容量     
        void overflowProcess(); //栈的溢出处理
    public:
         SeqStack(int sz);                    //构造函数
         ~SeqStack() { delete []elements; };        //析构函数
         void Push(ElemType x);    //进栈
         int Pop(ElemType &x);     //出栈
         int IsEmpty() const { return top == -1; }
         int IsFull() const { return top == maxSize-1; }
         int GetSize() const {return top+1;}
};
SeqStack::SeqStack(int sz)
{  elements=new ElemType[sz];  //申请连续空间
    if(elements==NULL) {cout<<"空间申请错误！"<<endl;exit(1);}
    else { top=-1;       //栈顶指示器指向栈底
               maxSize=sz;     //栈的最大空间
               };
};
/*
**********************************************************
  补充overflowProcess() 函数
**********************************************************
*/
void SeqStack::Push(ElemType x) 
{   //若栈满,则溢出处理，将元素x插入该栈栈顶
    if (IsFull() == 1) overflowProcess();   //栈满
    elements[++top] = x;       //栈顶指针先加1, 再元素进栈
}; 
int SeqStack::Pop(ElemType & x) 
{//若栈不空，函数退出栈顶元素并将栈顶元素的值赋给x,
  //返回true，否则返回false
   if (IsEmpty() == 1) return 0;
    x = elements[top--];           //先取元素，栈顶指针退1
      return 1;    //退栈成功
};
/*
**************************************************************
  补充mian()函数
**************************************************************
*/
```

## 分析和错误
`OverflowProcess`，用于栈溢出处理，一般申请原长度二倍的存储空间。

进制转换，10进制转n进制，除n取余法。

不能通过隐藏样例，认为是有类似0100之类零开头的结果，写代码排除，还是不行。

多次尝试无法通过，与通过的同学对比，发现是溢出处理写错了，在申请完二倍的空间后没让`maxSize`乘二。其实他的也写错了，不过因为他开的初始空间足够大这个函数压根没执行到，而强迫症的我就被卡了。

> 不得不吐槽的一点是题目中竟然让我补充`mian`函数……

## Answer
```c++
/*

**********************************************************

  补充overflowProcess() 函数

**********************************************************

*/
void SeqStack::overflowProcess(){
    ElemType*tmp=new ElemType[maxSize*2];
    for(int i=0;i<maxSize;i++){
        tmp[i]=elements[i];
    }
    delete[] elements;
    elements=tmp;
    maxSize*=2;
}

/*

**************************************************************

  补充mian()函数

**************************************************************

*/
int main(){
    int n,base;
    SeqStack*stk=new SeqStack(1);
    cin>>n>>base;
    int tmp=n;
    while(tmp>=base){
        stk->Push(tmp%base);
        tmp=tmp/base;
    }
    stk->Push(tmp);
    int counter=0;
    cout<<"("<<n<<")10=";
    cout<<"(";
    int tmp1;
    bool flag=true;
    while(stk->Pop(tmp1)){
        if(flag&&tmp1==0){
            continue;
        }else{
            flag=false;
        }
        char tmp2;
        if(tmp1>9){
            tmp2=tmp1-10+'A';
        }else{
            tmp2=tmp1+'0';
        }
        cout<<tmp2;
        counter++;
    }
    if(counter==0){
        cout<<0;
        counter++;
    }
    cout<<")"<<base<<endl;
    cout<<"Length="<<counter<<endl;
    delete stk;
}

```
