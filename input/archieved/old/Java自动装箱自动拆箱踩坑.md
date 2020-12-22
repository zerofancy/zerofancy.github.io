title: Java自动装箱自动拆箱踩坑
author: 归零幻想
publishDate: 2020-03-17
editDate: 2020-03-17
tags: [Java, leetcode]

<!--config-->

int和Integer有什么区别？前者是基础数据类型，后者是封装的Java对象。但在有`Autoboxing`和`Unboxing`的情况下我们常常就把两者等同看待，无非后者能放`null`。

事情要首先从一道力扣题目说起：

## 删除排序数组中的重复项 II
给定一个排序数组，你需要在原地删除重复出现的元素，使得每个元素最多出现两次，返回移除后数组的新长度。

不要使用额外的数组空间，你必须在原地修改输入数组并在使用 O(1) 额外空间的条件下完成。

<!--summary-->

### 示例 1:

```
给定 nums = [1,1,1,2,2,3],

函数应返回新长度 length = 5, 并且原数组的前五个元素被修改为 1, 1, 2, 2, 3 。

你不需要考虑数组中超出新长度后面的元素。
```

### 示例 2:

```
给定 nums = [0,0,1,1,1,1,2,3,3],

函数应返回新长度 length = 7, 并且原数组的前五个元素被修改为 0, 0, 1, 1, 2, 3, 3 。
```

你不需要考虑数组中超出新长度后面的元素。

### Answer

针对这个题目，我写出了这样的代码：

```java
/*
 * @lc app=leetcode.cn id=80 lang=java
 *
 * [80] 删除排序数组中的重复项 II
 */

// @lc code=start
class Solution {
    public int removeDuplicates(int[] nums) {
        int l = 0, r = 0;
        int ans1 = Integer.MIN_VALUE, ans2 = Integer.MIN_VALUE;
        while (r < nums.length) {
            if (ans1==ans2&& ans2 == nums[r]) {
                ;
            } else {
                nums[l] = nums[r];
                ans1 = ans2;
                ans2 = nums[r];
                l++;
            }
            r++;
        }
        return l;
    }
}
// @lc code=end
```

### 掉坑
这代码在一般情况是没有问题的，但题目有个测试样例是`[-2147483648,-2147483648,-2147483648,1,1,1,2]`，WA。

懒得处理开头特殊情况，但貌似运气不好，出题人是想让我处理一下的。

但作为懒癌患者我立马想到了新的偷懒方案，即用Integer代替int，这样用`null`表示没有就正好了。

于是改变如上语句，

```java
        Integer ans1 = null, ans2 = null;
```

和，

```java
            if (ans1!=null&&ans1== ans2&& ans2 == nums[r]) {
```

然而又WA了。那组样例输出是`[-2147483648,-2147483648,-2147483648,1,1,2]`。

## 问题原因
打开`jshell`，

jshell&gt; Integer a=-2147483648
a ==&gt; -2147483648

jshell&gt; Integer b=-2147483648
b ==&gt; -2147483648

jshell&gt; a==b
$6 ==&gt; false

jshell&gt; a==-2147483648
$7 ==&gt; true

jshell&gt; 

成功复现问题。

a与b直接比较，`a==b`，两者不是同一对象。与基本数据类型比较，`a==-2147483648`，发生自动拆箱，两者值相等。

所以，上述代码只需要修改

```java
            if (ans1 != null && ans1.intValue() == ans2.intValue() & ans2 == nums[r]) {
```

AC。

## 另一个处坑
这不是*自动装箱/自动拆箱*第一次坑我，曾经有这样一段代码：


<pre>jshell&gt; List&lt;Integer&gt; l=new LinkedList&lt;&gt;()
l ==&gt; []

jshell&gt; l.add(3)
$9 ==&gt; true

jshell&gt; l.add(5)
$10 ==&gt; true

jshell&gt; l.remove(3)
|  异常错误 java.lang.IndexOutOfBoundsException：Index: 3, Size: 2
|        at LinkedList.checkElementIndex (LinkedList.java:559)
|        at LinkedList.remove (LinkedList.java:529)
|        at (#11:1)

jshell&gt; l.remove((Object)3)
$12 ==&gt; true
</pre>


自动装箱自动拆箱配上函数重载，简直法力无边。你以为你删除了元素`3`，实际上你的代码是删除第三个元素……

## 参考
- [Java 的自动装箱(autoboxing)与拆箱(unboxing) ](http://chanthuang.github.io/2016/09/07/java-autoboxing-and-unboxing/)
