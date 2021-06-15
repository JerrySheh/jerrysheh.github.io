---
title: 剑指offer
comments: false
date: 2018-10-19 21:13:51
---


# 快速索引

## 链表

- [#6-从尾到头打印链表](./#6-从尾到头打印链表)
- [#18-删除链表的节点](./#18-删除链表的节点)
- [#22-链表中倒数第k个节点](./#22-链表中倒数第 k 个节点)
- [#23-链表中环的入口节点](./#23-链表中环的入口节点)
- [#24-反转链表](./#24-反转链表)
- [#25-合并两个排序的链表](./#25-合并两个排序的链表)
- #36-二叉搜索树与双向链表
- #35-复杂链表的复制
- #52-两个链表的第一个公共节点
- #62-圆圈中最后剩下的数字


# 1.赋值运算符函数

略

# 2.实现 Singleton 模式

1. 构造函数私有
2. 静态内部类保证线程安全（静态内部类在类初始化时只会被加载一次，因此是线程安全的）

```java
public class Singleton{
  private static class Holder{
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

在数组中，从 index = 0 开始，将arr[i] 交换到下标为 i 的地方，如果要交换的数字跟当前数字一样，说明是重复的。尽管代码中有两个循环，但每个数字最多只要交换两次就能找到自己的位置，因此总的时间复杂度也是 O(n)。

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

第一步：先计算原字符串空格数，如果有 n 个空格，那么新字符串长度应该是 原长度 + 2*n。
第二步：用两个下标，一个是原字符串下标，一个是新字符串下标，依次从后往前拷贝，遇到空格时，依次赋值`0`、`2`、`%`，非空格直接拷贝原字符串内容。

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

逆向输出链表，自然会想到用后进先出的栈。先遍历链表，把遍历的值依次写入栈，然后再出栈即可。

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

输入一个数组，写一个函数调整该数组的顺序，使得奇数在前，偶数在后。

思路：两个指针，一个在前（A），一个在后（B）。A往后遍历，直到遇到偶数，B往前遍历，直到遇到奇数，交换AB。

```java
private static void adjust(int[] arr){

    // left - 1 是因为稍后 ++left 会把 left 变大 1 ， right + 1 同理
    int left =  0 - 1;
    int right = arr.length - 1 + 1 ;

    while (left < right){

        //A往后遍历，直到遇到偶数
        while ((arr[++left] % 2) != 0){
            if (left > arr.length - 1) break;
        }

        //B往前遍历，直到遇到奇数
        while ((arr[--right] % 2) == 0){
            if (right <= 0) break;
        }

        // 交换
        if (left < right){
            Swap.swap(arr, left, right);
        }

    }

    System.out.println(Arrays.toString(arr));
}
```

---

# 22. 链表中倒数第 k 个节点


思路：两个指针，如求倒数第 3 个节点，A指针先走3步，然后B指针开始走，当A指针到尾的时候，B指针所指就是倒数第 k 个节点

链表定义如下
```java
static class ListNode{
    int value;
    ListNode next = null;

    ListNode(int val){
        this.value = val;
    }
}
```

```
2 → 88 → 60 → 21 → 6 → 13 → 78
↑（B）    ↑（A）

2 → 88 → 60 → 21 → 6 → 13 → 78
                   ↑（B）    ↑（A）
```

实现：

```java
private static int printReciNode(ListNode head, int k){

    if (k <= 0) return -1;

    ListNode nodeA = head;
    ListNode nodeB = head;

    //A指针先走k步
    for (int i = 0; i < k; i++) {
        // 求倒数第3个节点，但万一链表只有 2 个节点呢？
        if (nodeA.next == null){
            return -1;
        }
        nodeA = nodeA.next;
    }

    // A、B都走，直到 A 到末尾
    do {
        nodeB = nodeB.next;
    } while ((nodeA = nodeA.next) != null);

    return nodeB.value;
}
```

---

# 23. 链表中环的入口节点

如果一个链表中包含环，如何找出环的入口节点？

两个指针，一个一次走一步，另一个一次走两步，如果快的指针追上了慢的指针，那链表就包含环。否则如果走得快的到了末尾（node.next = null）都没追上慢的，就不包括环。

相遇节点就是入口节点。

```java
// 只实现是否包含环
private static boolean isCircle(ListNode head){

    ListNode pointA = head;
    ListNode pointB = head;

    while (true){
        pointA = pointA.next.next;
        pointB = pointB.next;

        if (pointA==null){
            break;
        }

        if (pointA.value == pointB.value && pointA.next.equals(pointB.next)) {
            return true;
        }
    }

    return false;

}
```

---

# 24. 反转链表

输入一个链表的头节点，反转该链表并输出反转后链表的头节点。

```java
private static ListNode reverse(ListNode head){

    // 反转后的头节点
    ListNode reverseHead = null;

    ListNode pre = null;
    ListNode current = head;

    while (current != null){
        ListNode next = current.next;

        // 如果 next 是空，说明到末尾了，是反转后的第一个元素
        if (next == null){
            reverseHead = current;
        }

        // 掉头，反指
        current.next = pre;

        pre = current;
        current = next;
    }

    return reverseHead;
}
```

---

# 25. 合并两个排序的链表

合并两个递增排序的链表，新链表的节点依然是排序的

```
List1: 1 → 3 → 5 → 7

List2：2 → 4 → 6 → 8

合并后：1 → 2 → 3 → 4 → 5 → 6 → 7 → 8
```

思路：递归法，如果 A值 小于 B值， merge节点为A， 然后继续比较 A.next 和 B ，反之 如果 B值 小于 A， merge节点为B，然后继续比较 A 和 B.next

```java
private static ListNode merge(ListNode A, ListNode B){

    if( A == null) {
        return B;
    } else if( B == null) return A;

    ListNode mergeHead;

    if (A.value < B.value){
        mergeHead = A;
        mergeHead.next = merge(A.next, B);
    } else {
        mergeHead = B;
        mergeHead.next = merge(A, B.next);
    }

    return mergeHead;
}
```

---

# 30. 包含min函数的栈

设计一个栈，除了可以 pop 和 push 之外，还能 getMin 获取栈中的最小值

思路：用一个辅助栈，主栈入栈时，如果栈顶 < 要插入的值，辅助栈压入要插入的值，如果 栈顶 >= 要插入的值，辅助栈压入原来的值。

```java
class myStack{
    Stack<Integer> stackA;
    Stack<Integer> stackB;

    public Integer pop(){
        stackB.pop();
        return stackA.pop();
    }

    public Integer push(Integer v){

        if (stackA.isEmpty()){
            stackB.push(v);
        }

        if (stackA.peek() >= v ){
            stackB.push(stackB.peek());
        }

        if (stackA.peek() < v){
            stackB.push(v);
        }

        return stackA.push(v);
    }

    public Integer getMin(){
        return stackB.peek();
    }

}
```

---

# 38. 字符串的排列

输入一个字符串，打印该字符串的所有排列。如，输入 abc， 输出 abc, acb, bac, bca, cab, cba

## 思路

把字符串分为两部分，第一部分为首字符，第二部分为剩下的字符，首字符确定下来后，剩下的字符递归进行全排列。这样一趟完了之后。会发生：

```
a-bcd
a-bdc
a-cbd
a-cdb
a-dbc
a-dcb
```

然后把首字符和第二个字符交换，进行第二趟递归全排列

```
b-acd
b-adc
b-cad
b-cda
b-dac
b-dca
```

实现

```java
private static void permutation(String str, int start, int end){

    if (end <= 1) return;

    char[] cs = str.toCharArray();

    if (start == end){
        System.out.println(new String(cs));
    } else {
        for (int i = start; i <= end; i++) {
            Swap.swap(cs, i, start);
            permutation(String.valueOf(cs), start+1, end);
            Swap.swap(cs, start, i);
        }
    }
}
```

---

# 40. 最小的 k 个数

输入 n 个整数，找出其中最小的 k 个数。

## 思路一：用快排的 partition 函数

因为 partition 函数本质上是将比参考元素大的放左边，比参考元素小的放右边。如果 partition 后参考元素的下标正好是 k ，那说明 k 左边的都是比 k 小的。这种解法时间复杂度只有 O(n)

```java
public static int[] partitionAndFind(int[] arr, int k){

    if (k > arr.length){
        return null;
    }

    int start = 0;
    int end = arr.length - 1;
    int index = QuickSort.partition(arr, start, end);

    // 如果 partition 后下标大了，调小，否则调大
    while (index != k-1){
        if(index > k-1){
            end = index - 1;
        }else{
            start = index + 1;
        }
        index = QuickSort.partition(arr,start, end);
    }

    int[] output = new int[k];
    System.arraycopy(arr, 0, output, 0, k);

    return output;
}
```

# 思路二：优先队列（堆）或红黑树

用一个大小为 k 的最大堆（本质是二叉树）作为数据容器 （也可以用红黑树作为数据容器），当容器未满，每次输入的数字直接加到堆里面，当容器已满，需要完成，判断堆顶（最大值）和即将添加的元素谁大，保留更小的。

需要 O(logk)的时间完成删除及插入操作。对于大小为 n 的输入，找到最小的 k 个，时间复杂度是 O(nlogk)，但是这种解法适合海量数据。

```java
public static int[] heapAndFind(int[] arr, int k){
    // o2 - o1， 最大堆， o1 - o2（默认） 最小堆
    PriorityQueue<Integer> q = new PriorityQueue<>(k, (o1, o2) -> o2-o1);

    for (int i = 0; i < arr.length; i++) {

        // 如果容器未满，直接添加
        if (!isFull(q, k)){
            q.add(arr[i]);
        } else { // 如果容器已满，将待添加元素和堆顶（最大值）比较，更小的放进堆里
            if (arr[i] < q.peek()){
                q.poll();
                q.add(arr[i]);
            }
        }
    }

    // 一个新的数组用来存放结果
    int[] output = new int[k];
    for (int i = 0; i < k; i++) {
        output[i] = q.poll();
    }

    return output;
}

// 优先队列（堆）是否满了
private static boolean isFull(PriorityQueue q, int k){
    return q.size() >= k;
}
```

---

#
