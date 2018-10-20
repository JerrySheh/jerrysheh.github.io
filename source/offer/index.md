---
title: 剑指offer
comments: false
date: 2018-10-19 21:13:51
---

# 1.赋值运算符函数

略

# 2.实现 Singleton 模式

1. 线程安全
2. 构造函数私有

```java
public class Singleton{
  private static Holder{
    private static Singleton s = new Signleton();
  }

  private Singleton (){

  }

  public static get(){
    return Holder.s;
  }

}
```

# 3.数组中重复的数字

题目：长度为 n 的数组，数字在 0 ~ n-1 范围内，有些数字是重复的，但不知道几个和几次。找出任意一个重复的数字。
例如：输入长度为 7 的数组{2,3,1,0,2,5,3}，数字在 0 ~ 6 范围内，输出 2 或者 输出 3

## 思路一：排序

先排序，然后从头到尾扫描数组。排序一个长度为 n 的数组需要的时间为 O(nlogn)

## 思路二：哈希表

扫描数组，每扫描到一个数字，都用 O(1) 的时间来判断哈希表是否已经包含该数字。如果没包含，就加入哈希表。该算法时间复杂度为 O(n)。但哈希表需要占用O(n)空间复杂度。

```java
private static int isDuplicateEleByHashSet(int[] arr) {

    HashSet<Integer> set = new HashSet<>();

    for (int i = 0; i < arr.length; i++) {
        if (set.contains(arr[i])){
            return arr[i];
        } else {
            set.add(arr[i]);
        }
    }
    return -1;
}
```

## 思路三：arr[i] 交换到下标为 i 的地方

在数组中，从 index = 0 开始，将arr[i] 交换到下标为 i 的地方，如果要交换的数字跟当前数字一样，说明是重复的。

```java
public static int isDuplicateEle(int[] arr){
    for (int i = 0; i < arr.length; i++) {
        // 只要 arr[i] 跟 i 不同，就比较下去
        while (arr[i] != i){
            // arr[i]是数组当前值， arr[arr[i]] 是即将要比较的值
            if (arr[arr[i]] == arr[i]){
                return arr[i];
            }
            Swap.swap(arr, arr[i], i);
        }
    }
    return -1;
}
```

---

# 4. 二维数组的查找

在二维数组中，每一行从左到右递增，每一列从上到下递增。给定一个二维数组和一个整数，该整数是否在数组里

```
      →  递增
     1   2   8   9
↓    2   4   9  12
递   4   7  10  13
增   6   8  11  15
```

## 思路

二维数组` arr[row][column]` 中，第一个中括号控制行（row），第二个中括号控制列（column）。可以从右上角开始，右上角为 `arr[0][arr[0].length - 1]`，如果目标值大于右上角，跳到下一行（row++），如果目标值小于右上角，跳到上一列（column--）

```java
public static boolean find(int[][] arr, int target){

    // 右上角的行号为 0，列号为 arr[0].length - 1
    int row = 0;
    int column = arr[0].length - 1;

    // 行依次递增，但要保证小于arr[0].length
    // 列依次递减，但要保证大于0
    while (row < arr[0].length && column >= 0){

        // 如果命中，返回 true
        if (target == arr[row][column]){
            return true;
        }

        // 如果 target 大于当前值，在下一行找
        else if(target > arr[row][column] ) {
            row++;
        }

        // 如果 target 小于当前值，在上一列找
        else if (target < arr[row][column]){
            column--;
        }

    }
    return false;

}
```

---

# 5. 替换空格

实现一个函数，把字符串中的空格替换成 `%20`，例如输入 `We are happy.`，输出`We%20are%20happy`。

## 思路

先计算原字符串空格数，如果有 n 个空格，那么新字符串长度应该是 原长度 + 2*n。用两个下标，一个是原字符串下标，一个是新字符串下标，依次从后往前拷贝，遇到空格时，依次赋值`0`、`2`、`%`，非空格直接拷贝原字符串内容。

```java
private static String replace(String str){

    char[] cs = str.toCharArray();

    // 计算空格数
    int blankNum = 0;
    for (int i = 0; i < cs.length; i++) {
        if (cs[i] == ' ') blankNum++;
    }

    // 创建新数组
    char[] resultCs = new char[cs.length + blankNum*2];

    // 两个下标，一个指向原数组，一个指向新数组
    int index_ori = cs.length - 1;
    int index_result = resultCs.length - 1;

    // 从后往前遍历原数组
    while (index_ori >= 0){
        // 遇到空格，新数组添加 %20
        if (cs[index_ori] == ' '){
             resultCs[index_result--] = '0';
             resultCs[index_result--] = '2';
             resultCs[index_result--] = '%';
             index_ori--;
        } else {
            // 不是空格，直接拷贝原数组内容
            resultCs[index_result--] = cs[index_ori--];
        }
    }

    return new String(resultCs);
}
```

如果允许我们用 java 内置方法，直接 replaceAll 搞定。

```java
private static String replaceByReplaceMethod(String s){
    return s.replaceAll(" ", "%20");
}
```

如果要求是 StringBuffer，没有 replaceAll 方法，但可以用 replace 方法

```java
// 如果要求是 StringBuffer，没有 replaceAll
private static String replaceByBuffer(StringBuffer sb){

    for (int i = 0; i < sb.length(); i++) {
        int index = sb.charAt(i);
        if (index == ' '){
            sb.replace(i, i+1, "%20");
        }
    }

    return sb.toString();

}
```

---

# 6. 从尾到头打印链表

输入一个链表的头节点，从尾到头打印出每个节点的值

```java
// 链表定义如下
static class ListNode{
    int value;
    ListNode next = null;

    ListNode(int val){
        this.value = val;
    }
}

// 测试用例如下
public static void main(String[] args) {

    ListNode n1 = new ListNode(1);
    n1.next = new ListNode(44);
    n1.next.next = new ListNode(80);
    n1.next.next.next = new ListNode(17);

    reversePrint(n1);
}
```

# 思路

逆向输出链表，自然会想到用后进先出的栈，可以遍历链表，把遍历的值依次写入栈，然后再出栈即可。

```java
private static void reversePrintByStack(ListNode node){

    Stack<Integer> stack = new Stack<>();

    do{
        stack.add(node.value);
    } while ((node = node.next) != null);

    while (!stack.isEmpty()){
        System.out.println(stack.pop());
    }

}
```

事实上，递归本身就是栈结构，这道题可以直接用递归代替栈

```java
// 递归实现
private static void reversePrint(ListNode node){

    if (node.next != null) {
        reversePrint(node.next);
    }

    System.out.println(node.value);

}
```

---

# 7. 重建二叉树

待完成

---

# 8. 二叉树的下一个节点

待完成

---

# 9. 用两个栈实现队列

思路：分为 stackA 和 stackB ，队列进时，往 stackA 进，队列出时先判断 StackB是否为空，不为空直接出，为空先把 stackA 全部倒进 stackB，再出。

```java
public class twoStack {

    Stack<Integer> StackA = new Stack<>();
    Stack<Integer> StackB = new Stack<>();

    // 插入队列
    public void insert(int a){
        StackA.push(a);
    }

    // 出队列
    public void delete(){

        // 如果 B 栈不空，直接出
        if (!StackB.isEmpty()){
            StackB.pop();
            return;
        }

        // 如果 B 栈空
        // 1.先将 A 栈全部倒入 B 栈
        while (!StackA.isEmpty()){
            int a = StackA.pop();
            StackB.push(a);
        }
        // 2.然后正常出
        StackB.pop();

    }

}
```

---

# 10. 斐波那契数列（Fibonacci）

写一个函数，输入n，输出斐波那契数列的第 n 项。

递归写法

```java
private static int getFib(int n){
    if (n < 1) return -1;
    if (n == 1 || n == 2) return 1;
    return getFib(n-2) + getFib(n-1);
}
```

遍历写法

```java
private static int getFib2(int n){
    if (n < 1) return -1;

    int last = 0;
    int current = 1;
    int next = last + current;

    for (int i = 1; i < n; i++) {
        current = next;
        next = current + last;
        last = current;
    }
    return next;
}
```

扩展问题：

1. 一只青蛙依次可以跳上1级台阶，也可以跳上2级，求该青蛙跳上一个 n 级台阶总共由多少种跳法。
2. 有一对兔子，从出生后第3个月起每个月都生一对兔子，小兔子长到第三个月后每个月又生一对兔子，假如兔子都不死，问每个月的兔子对数为多少？

以青蛙为例：

| 台阶数 | 跳法    | 说明 |
| :------------- | :------------- | :-----|
| 1       | 1      | 只能跳一下|
| 2       | 1      | 先1，后1|
| 3       | 2      | 先1后2，或者先2后1|
| 4       | 3      | 111，12，21|
| 5       | 4      | 1111，112， 121， 211， 22 |

显然是斐波那契数列问题

---

# 11. 旋转数组最小数字

数组旋转：把一个数组最开始的几个元素搬到数组的末尾。

要求：输入一个递增数组的旋转，输出旋转数组的最小元素

如：输入{3，4，5，1，2} （它是{1, 2, 3, 4, 5}的旋转），最小值为 1

## 思路

直观解法：遍历，但是时间复杂度为 O(n)

两个指针解法：


思路：

```
前：
6  7  8  9  10  1  2
↑                  ↑
A                  B

后：
6  7  8  9  10  1  2
         ↑         ↑
         A         B
```

判断中间元素 9 是否大于第一个指针元素 6 ，如果是，说明最小元素在右边，把第一个指针指向中间缩小范围

```
前：
6  7  1  2  3  4  5
↑                 ↑
A                 B

后：
6  7  1  2  3  4  5
↑        ↑
A        B
```

反之，如果第一个指针元素大于中间元素，说明最小元素在左边，把第二个指针指向中间缩小范围。

即：
- 中间>左边，移动第一指针到中间
- 中间<左边，移动第二指针到中间

结束条件：第一个指针跟第二个指针相邻，第二个指针指向的元素就是最小元素

实现：

```java
private static int getMin(int[] arr){

    int left = 0;
    int right = arr.length - 1;

    // 如果数组本身是有序的（旋转0）直接返回最左元素
    if (arr[left] < arr[right]) return arr[left];

    while (left + 1 != right){
        int mid = (left + right) / 2;

        //中间>左边，移动第一指针到中间
        if (arr[left] < arr[mid]) left = mid;

        //中间<左边，移动第二指针到中间
        else if (arr[left] >= arr[mid]) right = mid;
    }

    return arr[right];

}
```

---

# 12. 矩阵中的路径

回溯法

---

# 13. 机器人的运动范围

回溯法

---

# 14. 剪绳子

动态规划 + 贪婪

---

# 15. 二进制中1的个数

求一个二进制数字中 1 的个数，如 110101001 ，返回 5

分析：把一个整数减去1，再和原整数【与】运算，会把该整数最右边的1变成0

```
原整数：
11011001010

减去1：
11011001001

与运算：
11011001000
```

利用这个特性，只要计算与运算的次数即可

```java
private static int numberOf1(int n){
  int count = 0;

  while(n != 0){
    ++count;
    n = (n-1) & n;
  }
}
```

---

# 16. 数值的整数次方

输入一个double数 base ，求它的 n 次方，n是整数。

```java

```

---

# 17. 打印从1到最大的n位数

略

---

# 18. 删除链表的节点

删除链表节点的两种方法(比如要删除 i 节点)：

- 方法一: 遍历到 i 的前一个节点 h，将 h 指向 i 的下一个节点 j，然后删除 i
- 方法二：把节点 j 的内容复制到 i，将 i.next 指向 j 的下一节点 k，然后删除 j

---

# 21. 调整数组顺序使奇数位于偶数前面
