---
layout: post
title: AcWing 题目（13-88 是剑指 Offer）
categories: Prolems
description: AcWing 题目
keywords: AcWing 题目
---

目录

* TOC
{:toc}

## 1. A+B

+ 题目描述：

    输入两个整数，求这两个整数的和是多少。

    **事例：**

    样例输入：

    > 3 4

    样例输出：

    > 7

+ 解法：

    没啥好说的，注意一下 Scanner 和 BufferedReader 区别就行了，预热题。

+ 代码：

    ``` java
    import java.io.*;
    import java.util.*;

    public class Main{
        public static void main(String[] args){
            Scanner sc = new Scanner(System.in); // Scanner读取数据是按空格符（这其中包括空格键，Tab键，Enter键）来分割数据的。只要遇到其中之一，Scanner的方法就会返回下一个输入（当然nextLine()方法的结束符为换行符，它会返回换行符之前的数据）
            int a = sc.nextInt();
            int b = sc.nextInt();
            System.out.println(a+b);
        }
    }
    ```

## 2. 01 背包问题

+ 题目描述：

    有 N 件物品和一个容量是 V 的背包。每件物品只能使用一次。

    第 i 件物品的体积是 vi，价值是 wi。

    求解将哪些物品装入背包，可使这些物品的总体积不超过背包容量，且总价值最大。
    输出最大价值。

    输入格式

    第一行两个整数，N，V，用空格隔开，分别表示物品数量和背包容积。

    接下来有 N 行，每行两个整数 vi,wi，用空格隔开，分别表示第 i 件物品的体积和价值。

    输出格式

    输出一个整数，表示最大价值。

    数据范围

    0<N,V≤1000

    0<vi,wi≤1000

    **事例：**

    样例输入：

    >4 5  
    1 2  
    2 4  
    3 4  
    4 5  

    样例输出：

    > 8

+ 解法：

    如果拿回溯法解，时间复杂度是很高的，过不去案例的。

    用动态规划做，dp[i][j] 表示前 i 件物品，在体积 j 下的容量，这样 dp[i][j] = max(dp[i-1][j],dp[i-1][j-第 i 件的体积]+第 i 件的价值)。所以说外层是 i 件（因为要把全部的 i-1 的 j 算出来才能比大小），里层是从 1 开始的体积。

+ 代码：

    ``` java
    import java.io.*;
    import java.util.*;
    public class Main{
        static int max;
        public static void main2(String[] args){
            Scanner sc = new Scanner(System.in);
            int num = sc.nextInt();
            int maxV = sc.nextInt();
            int[] vs = new int[num];
            int[] values = new int[num];
            for(int i=0;i<num;i++){
                vs[i] = sc.nextInt();
                values[i] = sc.nextInt();
            }
            helper(vs,values,0,maxV,0);
            System.out.println(max);
        }
        public static void helper(int[] vs,int[] values,int index,int restV,int current){
            if(restV<0){
                return;
            }
            max = Math.max(max,current);
            for(int i=index;i<vs.length;i++){
                helper(vs,values,i+1,restV-vs[i],current+values[i]);
            }
        }

        public static void main(String[] args){
            Scanner sc = new Scanner(System.in);
            int num = sc.nextInt();
            int maxV = sc.nextInt();
            int[] vs = new int[num];
            int[] values = new int[num];
            for(int i=0;i<num;i++){
                vs[i] = sc.nextInt();
                values[i] = sc.nextInt();
            }
            int max = 0;
            int[][] dp = new int[vs.length+1][maxV+1];//前 i 个东西在体积 V 的情况下的最大值
            for(int i=1;i<=vs.length;i++){
                for(int j=1;j<=maxV;j++){
                    if(j>=vs[i-1])
                    dp[i][j] = Math.max(dp[i-1][j-vs[i-1]]+values[i-1],dp[i-1][j]);
                    else
                    dp[i][j] = dp[i-1][j];
                }
            }
            System.out.println(dp[vs.length][maxV]);
        }
    }
    ```

## 3. 完全背包问题

+ 题目描述：

    有 N 件物品和一个容量是 V 的背包。每种物品都有无限件可用。

    第 i 件物品的体积是 vi，价值是 wi。

    求解将哪些物品装入背包，可使这些物品的总体积不超过背包容量，且总价值最大。
    输出最大价值。

    输入格式

    第一行两个整数，N，V，用空格隔开，分别表示物品数量和背包容积。

    接下来有 N 行，每行两个整数 vi,wi，用空格隔开，分别表示第 i 件物品的体积和价值。

    输出格式

    输出一个整数，表示最大价值。

    数据范围

    0<N,V≤1000

    0<vi,wi≤1000

    **事例：**

    样例输入：

    >4 5  
    1 2  
    2 4  
    3 4  
    4 5  

    样例输出：

    > 10

+ 解法：

    同样用动态规划做，与之前不能放回相比，增加考虑一步除去第 i 个体积，增加第 i 个的重量。

+ 代码：

    ``` java
    import java.io.*;
    import java.util.*;
    public class Main{
        static int max;
        public static void main(String[] args){
            // 与之前的不同在于可以放回，所以最大值中还要包含一个本身自己
            Scanner sc = new Scanner(System.in);
            int num = sc.nextInt();
            int maxV = sc.nextInt();
            int[] vs = new int[num];
            int[] values = new int[num];
            for(int i=0;i<num;i++){
                vs[i] = sc.nextInt();
                values[i] = sc.nextInt();
            }
            int[][] dp = new int[vs.length+1][maxV+1];
            for(int i=1;i<vs.length+1;i++){
                for(int j=1;j<maxV+1;j++){
                    if(j>=vs[i-1])
                    dp[i][j] = Math.max(Math.max(dp[i-1][j-vs[i-1]]+values[i-1],dp[i][j-vs[i-1]]+values[i-1]),dp[i-1][j]);
                    else
                    dp[i][j] = dp[i-1][j];
                }
            }
            System.out.println(dp[vs.length][maxV]);
        }
    }
    ```

## 13. 找出数组中重复的数字

+ 题目描述：

    给定一个长度为 n 的整数数组 nums，数组中所有的数字都在 0∼n−1 的范围内。

    数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。

    请找出数组中任意一个重复的数字。

    注意：如果某些数字不在 0∼n−1 的范围内，或数组中不包含重复数字，则返回 -1；

    **样例：**

    给定 nums = [2, 3, 5, 4, 3, 2, 6, 7]。

    返回 2 或 3。

+ 解法：

    采用交换策略，即数组都在 0 - n-1 内，从头开始把对应的值和对应的位置做交换，当发现值一样的时候就不交换也就证明此时冲突了。

+ 代码：

    ``` java
    class Solution {
        public int duplicateInArray(int[] nums) {
            int res = -1;
            int mid = 0;
            for(int i=0;i<nums.length;i++){
                if(nums[i]<0||nums[i]>=nums.length) return -1;
                while(nums[i]!=i && nums[nums[i]]!=nums[i]){
                    swap(nums,i,nums[i]);
                }
                if(nums[i]!=i){
                    res = nums[i];
                    mid = i;
                    break;
                }
            }
            for(int i = mid;i<nums.length;i++){
                if(nums[i]<0||nums[i]>=nums.length) return -1;
            }
            return res;
        }
        public void swap(int[] nums, int a, int b){
            int temp = nums[a];
            nums[a]=nums[b];
            nums[b] = temp;
        }
    }
    ```

## 14. 不修改数组找出重复的数字

+ 题目描述：

    给定一个长度为 n+1 的数组nums，数组中所有的数均在 1∼n 的范围内，其中 n≥1。

    请找出数组中任意一个重复的数，但不能修改输入的数组。

    **样例：**

    给定 nums = [2, 3, 5, 4, 3, 2, 6, 7]。

    返回 2 或 3。

+ 解法：

    把数组想象成链表，这样找有无重复数字就转换成了找链表有没有环。这样就是一快一慢。但需要注意的是，如果数组中没有重复数字，需要记下个数，当走的步数超过了数组的长度就证明没有重复数字了。

    第二种是分治法，对自然数 1-n 分治，如果有重复数字，也就是分治那一半的数字数量会大于一半。

+ 代码：

    ``` java
    class Solution {
        public int duplicateInArray(int[] nums) {
            int res = -1;
            int mid = 0;
            for(int i=0;i<nums.length;i++){
                if(nums[i]<0||nums[i]>=nums.length) return -1;
                while(nums[i]!=i && nums[nums[i]]!=nums[i]){
                    swap(nums,i,nums[i]);
                }
                if(nums[i]!=i){
                    res = nums[i];
                    mid = i;
                    break;
                }
            }
            for(int i = mid;i<nums.length;i++){
                if(nums[i]<0||nums[i]>=nums.length) return -1;
            }
            return res;
        }
        public void swap(int[] nums, int a, int b){
            int temp = nums[a];
            nums[a]=nums[b];
            nums[b] = temp;
        }
        //分治法，因为所有的数都在 1-n 之间，把数组长度从 1-n 分治，每次确定重复的数字是在 1-mid 还是 mid-n
        public int duplicateInArray(int[] nums) {
            int left = 1;
            int right = nums.length;
            for(int i:nums){
                if(i<=0||i>nums.length) return -1;
            }
            while(left<right){
                int mid = (left+right)/2;
                int leftCount = fenzhi(nums,left,mid);
                int rightCount = fenzhi(nums,mid+1,right);
                if(leftCount>mid-left+1) right = mid;
                else left = mid+1;
            }
            return left;
        }
        public int fenzhi(int[] nums,int begin,int end){
            int count = 0;
            for(int i:nums){
                if(i>=begin && i<=end){
                    count++;
                }
            }
            return count;
        }
    }
    ```

## 15. 二维数组中的查找

+ 题目描述：

    在一个二维数组中（每个一维数组的长度相同），每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

+ 解法：

    从右上角/左下角开始找，因为这样一个方向是比这个数小，另一个方向比这个数大，每次可以删掉一行/列。

+ 代码：

    ``` java
    public class Solution {
        public boolean Find(int target, int [][] array) {
            //二维数组的右上角的左边都比他小，下边的都比他大，利用这个特点做
            int i = 0;//行
            int j = array[0].length-1;//列
            while(i<array.length && j>=0){
                if(array[i][j]==target) return true;
                else if(array[i][j]>target){ //应该去比他小的即左边着
                    j--;
                }else{ //找比他大的下边
                    i++;
                }
            }
            return false;
        }
    }
    ```

## 16. 替换空格

+ 题目描述：

    请实现一个函数，把字符串中的每个空格替换成"%20"。

    你可以假定输入字符串的长度最大是1000。
    注意输出字符串的长度可能大于1000。

+ 解法：

    提前申请一个替换后长度的 StringBuffer，这样可以避免重复申请空间。

+ 代码：

    ``` java
    class Solution {
        public String replaceSpaces(StringBuffer str) {
            int appendLength = 0;
            for(int i = 0;i<str.length();i++){
                if(str.charAt(i)==' '){
                    appendLength+=2;
                }
            }
            StringBuffer res = new StringBuffer(str.length()+appendLength);
            for(int i = 0;i<str.length();i++){
                char c = str.charAt(i);
                if(c==' '){
                    res.append("%20");
                }else{
                    res.append(c);
                }
            }
            return res.toString();
        }
    }
    ```

## 17. 从尾到头打印链表

+ 题目描述：

    输入一个链表的头结点，按照 从尾到头 的顺序返回节点的值。

    返回的结果用数组存储。

+ 解法：

    用一个栈保存从每个节点，再反遍历这个栈。当然也可以递归，但这里是返回数组，递归不方便。

+ 代码：

    ``` java
    class Solution {
        public int[] printListReversingly(ListNode head) {
            Stack<ListNode> stack = new Stack<>();
            int i = 0;
            while(head!=null){
                stack.add(head);
                head = head.next;
                i++;
            }
            int[] res = new int[i];
            i=0;
            while(!stack.isEmpty()){
                res[i] = stack.pop().val;
                i++;
            }
            return res;
        }
    }
    ```

## 16. 替换空格

+ 题目描述：

    请实现一个函数，把字符串中的每个空格替换成"%20"。

    你可以假定输入字符串的长度最大是1000。
    注意输出字符串的长度可能大于1000。

+ 解法：

    提前申请一个替换后长度的 StringBuffer，这样可以避免重复申请空间。

+ 代码：

    ``` java
    class Solution {
        public String replaceSpaces(StringBuffer str) {
            int appendLength = 0;
            for(int i = 0;i<str.length();i++){
                if(str.charAt(i)==' '){
                    appendLength+=2;
                }
            }
            StringBuffer res = new StringBuffer(str.length()+appendLength);
            for(int i = 0;i<str.length();i++){
                char c = str.charAt(i);
                if(c==' '){
                    res.append("%20");
                }else{
                    res.append(c);
                }
            }
            return res.toString();
        }
    }
    ```

## 18. 重建二叉树

+ 题目描述：

    输入一棵二叉树前序遍历和中序遍历的结果，请重建该二叉树。二叉树中每个节点的值都互不相同；输入的前序遍历和中序遍历一定合法；

+ 解法：

    前序遍历的第一个节点是中间节点，拿中间节点去数组中定位可以定位到左节点长度和右节点长度。可以用一个 map 记录中序遍历节点方便查找。手边可以画一个二叉树对这算一算 index。

+ 代码：

    ``` java
    class Solution {
        public TreeNode buildTree(int[] preorder, int[] inorder) {
            Map<Integer,Integer> map = new HashMap<>(); // 中序遍历的 hash 表，这样可以实现找中节点的速度降到 O(1)
            for(int i=0;i<inorder.length;i++){
                map.put(inorder[i],i);
            }
            return helper(preorder, inorder, 0, 0, preorder.length, map);
        }
        public TreeNode helper(int[] preorder, int[] inorder, int preIndex, int inIndex, int length, Map<Integer,Integer> map){
            if(preIndex>=preorder.length || inIndex>=inorder.length||length<=0) return null;//递归出来的条件是 index 有没有超界和是不是没有长度了
            int midVal = preorder[preIndex];
            TreeNode node = new TreeNode(midVal);
            int newinindex = map.get(midVal);
            int newLength = newinindex - inIndex;
            node.left = helper(preorder, inorder, preIndex+1, inIndex, newLength, map);
            node.right = helper(preorder, inorder, preIndex+newLength+1, newinindex+1, inIndex+length-newinindex-1, map);
            return node;
        }
    }
    ```

## 19. 二叉树的下一个节点

+ 题目描述：

    给定一棵二叉树的其中一个节点，请找出中序遍历序列的下一个节点。

    + 如果给定的节点是中序遍历序列的最后一个，则返回空节点;
    + 二叉树一定不为空，且给定的节点一定不是空节点；

+ 解法：

    需要分两种情况，第一种是二叉树右边不为空，这样下一个节点就是右边的最深的左节点。如果右边为空，就要判断该节点跟他父节点位置，要一直找到该节点是父节点的左节点时，父节点才是下一个点。如果没有那就是最后一个节点了。

+ 代码：

    ``` java
    /**
    * Definition for a binary tree node.
    * public class TreeNode {
    *     int val;
    *     TreeNode left;
    *     TreeNode right;
    *     TreeNode father;
    *     TreeNode(int x) { val = x; }
    * }
    */
    class Solution {
        //只要是右节点非空，就是右节点下的最左节点（不是直接右，第一次做错了）。
        //如果右节点空，一直往上找，直到他是父亲的左节点
        public TreeNode inorderSuccessor(TreeNode p) {
            if(p.right!=null){
                TreeNode next = p.right;
                while(next.left!=null){
                    next = next.left;
                }
                return next;
            }
            while(p.father!=null){
                TreeNode temp = p;
                p = p.father;
                if(p.left==temp) return p; //只有当老节点是父节点左节点时，才是下一个节点，否则就是继续往上找
            }
            return null;
        }
    }
    ```

## 20. 用两个栈实现队列

+ 题目描述：

    请用栈实现一个队列，支持如下四种操作：

    + push(x) – 将元素x插到队尾；
    + pop() – 将队首的元素弹出，并返回该元素；
    + peek() – 返回队首元素；
    + empty() – 返回队列是否为空；

    注意：

    + 你只能使用栈的标准操作：push to top，peek/pop from top, size 和 is empty；
    + 如果你选择的编程语言没有栈的标准库，你可以使用list或者deque等模拟栈的操作；
    + 输入数据保证合法，例如，在队列为空时，不会进行pop或者peek等操作；

    样例：

    ``` java
    MyQueue queue = new MyQueue();

    queue.push(1);
    queue.push(2);
    queue.peek();  // returns 1
    queue.pop();   // returns 1
    queue.empty(); // returns false
    ```

+ 解法：

    两个栈实现队列，一个是用来专门放进的，一个是专门出得。出得栈只有为空的时候，才会把进的栈全部倒入出的栈，这样可以保证顺序不会出现错乱。

+ 代码：

    ``` java
    class MyQueue {

        /** Initialize your data structure here. */
        Stack<Integer> stackPush; //倒序栈，用来往里面放
        Stack<Integer> stackPop; //正序栈，用来出，当没有的时候把倒叙里的全都扔到正序里，但一定注意只有空的时候才能扔，否则就不满足顺序了
        public MyQueue() {
            stackPush = new Stack<>();
            stackPop = new Stack<>();
        }
        
        /** Push element x to the back of queue. */
        public void push(int x) {
            stackPush.push(x);
        }
        
        /** Removes the element from in front of queue and returns that element. */
        public int pop() {
            if(stackPop.isEmpty()){
                while(stackPush.size()>0){
                    stackPop.push(stackPush.pop());
                }
            }
            return stackPop.pop();
        }
        
        /** Get the front element. */
        public int peek() {
            if(stackPop.isEmpty()){
                while(stackPush.size()>0){
                    stackPop.push(stackPush.pop());
                }
            }
            return stackPop.peek();
        }
        
        /** Returns whether the queue is empty. */
        public boolean empty() {
            return stackPush.isEmpty()&&stackPop.isEmpty();
        }
    }

    /**
    * Your MyQueue object will be instantiated and called as such:
    * MyQueue obj = new MyQueue();
    * obj.push(x);
    * int param_2 = obj.pop();
    * int param_3 = obj.peek();
    * boolean param_4 = obj.empty();
    */
    ```

## 21. 斐波那切数列

+ 题目描述：

    输入一个整数 n ，求斐波那契数列的第 n 项。

+ 解法：

    递归或循环。递归不说了，效率低，循环需要记住前两个数就行。

+ 代码：

    ``` java
    class Solution {
        // 1 1 2 3 5
        public int Fibonacci(int n) {
            if(n<=0) return 0;
            if(n<=2) return 1;
            return Fibonacci(n-1)+Fibonacci(n-2);
        }
        public int Fibonacci(int n) {
            if(n<=0) return 0;
            if(n<=2) return 1;
            int first = 1;
            int second = 1;
            for(int i=3;i<=n;i++){
                //前一个数等于后一个数，后一个数等于原来的前一个数加上后一个数
                int temp = first;
                first = second;
                second = temp + second;
            }
            return second;
        }
    }
    ```

## 22. 旋转最小的数字

+ 题目描述：

    把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。

    输入一个升序的数组的一个旋转，输出旋转数组的最小元素。

    例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。

    数组可能包含重复项。

    **注意：**数组内所含元素非负，若数组大小为0，请返回-1。

    **示例：**

    输入：nums=[2,2,2,0,1]

    输出：0

+ 解法：

    见到排序基本上就是二分法。拿中间数和两边数比，由于旋转后的规律，如果中间数小证明已经过了旋转点，如果中间数大，说明还没到旋转点。但如果相等则不好说了，需要一个一个去比。    

+ 代码：

    ``` java
    class Solution {
        public int findMin(int[] nums) {
            if(nums.length==0) return -1;
            //二分法，对于 7890123456，中间数 1 比 7 小，就在左边，如果比 7 大就在右边。
            int start = 0;
            int end = nums.length-1;
            if(nums[start]<nums[end]) return nums[start];//一直递增就是没转
            while(start<end-1){ //要考虑相等的情况，相等时说不好在左还是在右，比如 22201 就在右，但 1101111 就在左，同时因为 10 这样的，所以差 1 必须停下来
                int mid = end+(start-end)/2;
                if(nums[start]>nums[mid]){
                    end = mid;
                }else if(nums[start]<nums[mid]){
                    start = mid;
                }else{
                    break;
                }
            }
            int min = nums[start];
            for(int i=start+1;i<=end;i++){
                min = Math.min(nums[i],min);
            }
            return min;
        }
    }
    ```