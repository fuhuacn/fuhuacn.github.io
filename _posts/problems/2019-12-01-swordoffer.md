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

## 23. 矩阵中的路径

+ 题目描述：

    请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。

    路径可以从矩阵中的任意一个格子开始，每一步可以在矩阵中向左，向右，向上，向下移动一个格子。

    如果一条路径经过了矩阵中的某一个格子，则之后不能再次进入这个格子。

    **注意：**数组内所含元素非负，若数组大小为0，请返回-1。

    **示例：**

    matrix=  
    [  
    ["A","B","C","E"],  
    ["S","F","C","S"],  
    ["A","D","E","E"]  
    ]

    str="BCCE" , return "true" 

    str="ASAE" , return "false"

+ 解法：

    回溯法，一个字符一个字符的去比，但要记录一个二维数组判定是否访问过这个节点。

+ 代码：

    ``` java
    class Solution {
        public boolean hasPath(char[][] matrix, String str) {
            if(matrix.length==0) return false;
            boolean[][] isVisted = new boolean[matrix.length][matrix[0].length];
            for(int i=0;i<matrix.length;i++){
                for(int j=0;j<matrix[0].length;j++){
                    if(helper(matrix,str,i,j,0,isVisted)) return true;
                }
            }
            return false;
        }
        public boolean helper(char[][] matrix, String str, int row, int col, int index,boolean[][] isVisited){
            if(index == str.length()) return true;
            int rowLength = matrix.length;
            int colLength = matrix[0].length;
            if(row<0||col<0||row==rowLength||col==colLength||isVisited[row][col]){
                return false;
            }
            if(matrix[row][col]==str.charAt(index)){
                isVisited[row][col] = true;
                boolean res = helper(matrix,str,row-1,col,index+1,isVisited)||helper(matrix,str,row+1,col,index+1,isVisited)||helper(matrix,str,row,col-1,index+1,isVisited)||helper(matrix,str,row,col+1,index+1,isVisited);
                if(res) return true;
                isVisited[row][col] = false;
                return false;
            }else{
                return false;
            }
        }
    }
    ```

## 24. 机器人的运动范围

+ 题目描述：

    地上有一个 m 行和 n 列的方格，横纵坐标范围分别是 0∼m−1 和 0∼n−1。

    一个机器人从坐标0,0的格子开始移动，每一次只能向左，右，上，下四个方向移动一格。

    但是不能进入行坐标和列坐标的数位之和大于 k 的格子。

    请问该机器人能够达到多少个格子？

    **示例：**

    输入：k=7, m=4, n=5

    输出：20

+ 解法：

    DFS：因为固定了从（0，0）走，所以其实路只会向下或向右走，左和上的一定是走过的，走到的地方判断符不符合。

    BFS：从头开始把能走到的加入队列挨个走。

+ 代码：

    ``` java
    class Solution {
        public int movingCount(int threshold, int rows, int cols)
        {
            boolean[][] visited = new boolean[rows][cols];
            int[] res = new int[1];
            helper(threshold, rows, cols, 0, 0, res, visited);
            return res[0];
        }
        public void helper(int threshold, int rows, int cols,int row,int col,int[] res,boolean[][] visited){
            // 从 0,0 出发只用顾及向右和向下就可以了
            if(row==rows||col==cols||visited[row][col]){
                return;
            }
            visited[row][col] = true;
            if(getValue(row,col)<=threshold){
                res[0]++;
                helper(threshold, rows, cols,row+1,col,res,visited);
                helper(threshold, rows, cols,row,col+1,res,visited);
            }
        }
        public int movingCount2(int threshold, int rows, int cols)
        {
            if (rows < 1 || cols < 1)
                return 0;
            boolean[][] visited = new boolean[rows][cols];
            Queue<int[]> queue = new LinkedList<>();//存的是一对分别是：行和列
            int[] begin = new int[2];
            queue.add(begin);
            int res = 0;
            visited[0][0] = true;
            while(queue.size()>0){
                int[] temp = queue.poll();
                res++;
                for(int i=0;i<2;i++){
                    temp[i]++;
                    int row = temp[0];
                    int col = temp[1];
                    if(row<rows&&col<cols&&!visited[row][col]&&getValue(row,col)<=threshold){
                        queue.add(new int[]{row,col}); //这块不能直接放数组，因为数组是地址，一会儿还要减回去
                        visited[row][col]=true;
                    }
                    temp[i]--;
                }
            }
            return res;
        }
        public int getValue(int row,int col){
            int sum=0;
            while(row>0){
                sum += row%10;
                row /= 10;
            }
            while(col>0){
                sum += col%10;
                col /= 10;
            }
            return sum;
        }
    }
    ```

## 25. 剪绳子

+ 题目描述：

    给你一根长度为 n 绳子，请把绳子剪成 m 段（m、n）都是整数，2≤n≤58 并且 m≥2）。

    每段的绳子的长度记为k[0]、k[1]、……、k[m]。k[0]k[1] … k[m] 可能的最大乘积是多少？

    例如当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到最大的乘积18。

    **示例：**

    输入：8

    输出：18

+ 解法：

    动态规划，他要注意长度为 1、2、3 时不切比切了还长。

+ 代码：

    ``` java
    class Solution {
        public int maxProductAfterCutting(int length)
        {
            if(length==0) return 0;
            if(length<=2) return 1;
            if(length==3) return 2;//这三个数不切比切了长
            int[] dp = new int[length+1];
            dp[1] = 1;
            for(int i=2;i<=length;i++){
                dp[i] = i;
                for(int j=1;j<=i/2;j++){
                    dp[i] = Math.max(dp[i],dp[j]*dp[i-j]);
                }
            }
            return dp[length];
        }
    }
    ```

## 26. 二进制中 1 的个数

+ 题目描述：

    输入一个32位整数，输出该数二进制表示中1的个数。

    **示例：**

    输入：-2

    输出：31
    解释：-2在计算机里会被表示成11111111111111111111111111111110，一共有31个1。

+ 解法：

    Java 有无符号右移（>>>）这题就比较好做，每次右移一位和 1 做与运算就好了。

+ 代码：

    ``` java
    class Solution {
        public int NumberOf1(int n)
        {
            //得用无符号右移(>>>)，>> 是有符号右移，这样不会补 1。
            int count = 0;
            while(n!=0){
                if((n&1)==1) count++;
                n>>>=1;
            }
            return count;
        }
    }
    ```

## 27. 数值的整数次方

+ 题目描述：

    实现函数double Power(double base, int exponent)，求base的 exponent次方。

    不得使用库函数，同时不需要考虑大数问题。

    **示例：**

    输入：10 ，-2  

    输出：0.01

+ 解法：

    需要考虑负数情况，有负数取个倒数。

+ 代码：

    ``` java
    class Solution {
        public double Power(double base, int exponent) {
            if(exponent==0) return 1;
            double res = base;
            int absExp = exponent>0?exponent:-exponent;
            for(int i=2;i<=absExp;i++){
                res *= base;
            }
            return exponent>0?res:1/res;
        }
    }
    ```

## 28. 在O(1)时间删除链表结点

+ 题目描述：

    给定单向链表的一个节点指针，定义一个函数在O(1)时间删除该结点。

    假设链表一定存在，并且该节点一定不是尾节点。

    **示例：**

    输入：链表 1->4->6->8

    删掉节点：第2个节点即6（头节点为第0个节点）

    输出：新链表 1->4->8

+ 解法：

    把当前节点变成他的下一个节点就好了。

+ 代码：

    ``` java
    /**
    * Definition for singly-linked list.
    * public class ListNode {
    *     int val;
    *     ListNode next;
    *     ListNode(int x) { val = x; }
    * }
    */
    class Solution {
        public void deleteNode(ListNode node) {
            node.val = node.next.val;
            node.next = node.next.next;
        }
    }
    ```

## 29. 删除链表中重复的节点

+ 题目描述：

    在一个**排序**的链表中，存在重复的结点，请删除该链表中重复的结点，重复的结点不保留。

    **示例：**

    输入：1->2->3->3->4->4->5

    输出：1->2->5

+ 解法：

    首先需要一个头节点，因为有可能第一个就是重复的就要被删除。

    需要一个节点来表示目前上一个已经确定的非重复节点。

    把每个节点和每个节点的下一个值去比。如果值相等，那他就是重复节点，就要用一个循环一直走到不再等的时候。如果不等，证明这个节点是唯一的。

+ 代码：

    ``` java
    /**
    * Definition for singly-linked list.
    * public class ListNode {
    *     int val;
    *     ListNode next;
    *     ListNode(int x) { val = x; }
    * }
    */
    class Solution {
        public ListNode deleteDuplication(ListNode head) {
            if(head==null) return null;
            ListNode first = new ListNode(-1);//以防第一个就是重复节点，增加一个头。
            first.next = head;
            ListNode lastNode = first;//上一个确定不重复的点，head 是目前到的点
            while(head!=null && head.next!=null){
                if(head.val == head.next.val){
                    while(head.next!=null&&head.val == head.next.val){
                        head = head.next;
                    }
                    head = head.next;
                    lastNode.next = head; // 把之前删除的删掉，但此时的 head 依然没有比较，所以 lastNode 不会变，无法确定当前这个head是否是唯一的。
                }else{
                    lastNode = head;
                    head = head.next;
                }
            }
            return first.next;
        }
    }
    ```

## 30. 正则表达式匹配

+ 题目描述：

    请实现一个函数用来匹配包括'.'和'*'的正则表达式。

    模式中的字符'.'表示任意一个字符，而'*'表示它前面的字符可以出现任意次（含0次）。

    在本题中，匹配是指字符串的所有字符匹配整个模式。

    例如，字符串"aaa"与模式"a.a"和"ab*ac*a"匹配，但是与"aa.a"和"ab*a"均不匹配。

+ 解法：

    首先明确什么情况下匹配失败：当正则表达式匹配完了，但字符还有的时候，就肯定匹配不上了。

    接下来考虑情况，首先是如果正则和字符串这位相同，或者正则这位是 . 双方都加一位就好了。

    如果正则的下一位是 *，这时情况比较复杂：
    
    当此位相同时：
    + 有可能是正则匹配无数个此位，即正则不变但字符串往后跳一位。
    + 有可能是正则这位算 0 个，由下面一位来和此位相等，即正则跳两位而字符串不变。
    + 有可能正好是结尾，也就是正则跳两位，字符串走一位。这一种情况是上两种的结合，所以代码里可以不体现，

    当此位不相同时：
    + 只可能是 * 算 0 个，所以正则跳两位而字符串不变。

+ 代码：

    ``` java
    class Solution {
        public boolean isMatch(String s, String p) {
            return helper(s,p,0,0);
        }
        public boolean helper(String s, String p, int sIndex, int pIndex){
            if(pIndex == p.length() && sIndex!=s.length()) return false; //p 完了，s 没完则后面一定匹配不上了
            if(p.length() == pIndex && s.length()==sIndex) return true;
            // 如果这一位的下一位是 * 且此位置的字符相同，则可以让比较 s 过一位，p 不变即无数个此字符。s 过一位，p 跳两位，即这是最后一个 *，s 不变，p 跳两位，即没有这个字符即 p 这次按 0 算，找下个字符
            // 如果不相同则可能是 s 不变，p 跳两位，即没有这个字符。
            // s 过一位，p 跳两位，即这是最后一个 * 这种情况是 s 过一位，p 不变 情况下再 s 不变，p 跳两位，所以不用计算这个了。
            if(pIndex+1<p.length() && p.charAt(pIndex+1)=='*'){
                // 一定要加 sIndex 的判断，因为很有可能 s 已经匹配完了，p 还有 * 要记着有 . 的情况
                if(sIndex<s.length()&&(s.charAt(sIndex) == p.charAt(pIndex) || p.charAt(pIndex)=='.')){
                    return helper(s,p,sIndex+1,pIndex)||helper(s,p,sIndex,pIndex+2);
                }else{
                    return helper(s,p,sIndex,pIndex+2);
                }
            }
            //如果两个字符相同或 . 则都下一位
            if(sIndex<s.length()&&(s.charAt(sIndex) == p.charAt(pIndex)||p.charAt(pIndex)=='.')){
                return helper(s,p,sIndex+1,pIndex+1);
            }
            return false;
        }
    }
    ```

## 31. 表示数值的字符串

+ 题目描述：

    请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。

    例如，字符串"+100","5e2","-123","3.1416"和"-1E-16"都表示数值。

    但是"12e","1a3.14","1.2.3","+-5"和"12e+4.3"都不是。

    注意:

    + 小数可以没有整数部分，例如.123等于0.123；
    + 小数点后面可以没有数字，例如233.等于233.0；
    + 小数点前面和后面可以有数字，例如233.666;
    + 当e或E前面没有数字时，整个字符串不能表示数字，例如.e1、e1；
    + 当e或E后面没有整数时，整个字符串不能表示数字，例如12e、12e+5.4;

+ 解法：

    完全就是规则匹配，非常无聊。

+ 代码：

    ``` java
    class Solution {
        public boolean isNumber(String s) {
            if(s.equals(".")||s.equals("")) return false;
            char[] cs = s.toCharArray();
            boolean isE = false;
            boolean isSymbol = false;
            boolean isDot = false;
            for(int i=0;i<cs.length;i++){
                if(cs[i]=='e'||cs[i]=='E'){
                    //如果是 e，他不能在第一位也不能在最后一位，并且之前不能出现过，并且前面一位必须是数字
                    if(i==0||isE||i==cs.length-1||(cs[i-1]<'0'||cs[i-1]>'9')){
                        return false;
                    }
                    isE = true;
                }else if(cs[i]=='.'){
                    //如果是 . 前面不能出现过 e，不能出现过小数点
                    if(isE||isDot||(i==cs.length-1&&(cs[i-1]>'9'||cs[i-1]<'0'))){
                        return false;
                    }
                    isDot = true;
                }else if(cs[i]=='+'||cs[i]=='-'){
                    //如果是正负号，必须在第一位或者前面一位是 e，并且不能是最后一位
                    if(i==0){
                        isSymbol = true;
                    }else{
                        if((cs[i-1]!='e'&&cs[i-1]!='E')||i==cs.length-1) return false;
                    }
                }else if(cs[i]<'0'||cs[i]>'9') return false;
            }
            return true;
        }
    }
    ```

## 32. 调整数组顺序使奇数位于偶数前面

+ 题目描述：

    输入一个整数数组，实现一个函数来调整该数组中数字的顺序。

    使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分。

    **样例：**

    输入：[1,2,3,4,5]

    输出: [1,3,5,2,4]

+ 解法：

    需要思考问题的核心是什么。即把左边的偶数和右边的奇数交换。所以可以用双指针的办法，左边遇到偶数和右边的奇数交换。

+ 代码：

    ``` java
    class Solution {
        public void reOrderArray(int [] array) {
            // 左右两个指针，左指针指向偶数，右指针指向奇数，奇偶数互换
            int left = 0;
            int right = array.length-1;
            while(left<right){
                while(left<right&&array[left]%2==1){
                    left++;
                }
                while(left<right&&array[right]%2==0){
                    right--;
                }
                int temp = array[left];
                array[left] = array[right];
                array[right] = temp;
                left++;
                right--;
            }
        }
    }
    ```

## 33. 链表中倒数第k个节点

+ 题目描述：

    输入一个链表，输出该链表中倒数第k个结点。

    **样例：**

    输入：链表：1->2->3->4->5 ，k=2

    输出：4

+ 解法：

    很简单，先跑 k 个节点就可以了。

+ 代码：

    ``` java
    /**
    * Definition for singly-linked list.
    * public class ListNode {
    *     int val;
    *     ListNode next;
    *     ListNode(int x) { val = x; }
    * }
    */
    class Solution {
        public ListNode findKthToTail(ListNode pListHead, int k) {
            ListNode node = pListHead;
            int i=0;
            while(i<k && node!=null){
                node = node.next;
                i++;
            }
            if(i<k) return null;
            while(node!=null){
                node = node.next;
                pListHead = pListHead.next;
            }
            return pListHead;
        }
    }
    ```

## 34. 链表中环的入口结点

+ 题目描述：

    给定一个链表，若其中包含环，则输出环的入口节点。

    若其中不包含环，则输出null。

+ 解法：

    快慢跑找追上的点，再一个一个跑找相遇点。

+ 代码：

    ``` java
    /**
    * Definition for singly-linked list.
    * class ListNode {
    *     int val;
    *     ListNode next;
    *     ListNode(int x) {
    *         val = x;
    *         next = null;
    *     }
    * }
    */
    class Solution {
        public ListNode entryNodeOfLoop(ListNode head) {
            ListNode fast = head;
            ListNode slow = head;
            do{
                if(fast!=null&&fast.next!=null){
                    fast = fast.next.next;
                }else{
                    return null;
                }
                slow = slow.next;
            }while(fast!=slow);
            slow = head;
            while(slow!=fast){
                slow = slow.next;
                fast = fast.next;
            }
            return slow;
        }
    }
    ```

## 35. 反转链表

+ 题目描述：

    定义一个函数，输入一个链表的头结点，反转该链表并输出反转后链表的头结点。

+ 解法：

    记录一下上一个节点。遍历节点，每个节点的下一个节点是上一个节点就可以了。

+ 代码：

    ``` java
    /**
    * Definition for singly-linked list.
    * public class ListNode {
    *     int val;
    *     ListNode next;
    *     ListNode(int x) { val = x; }
    * }
    */
    class Solution {
        public ListNode reverseList(ListNode head) {
            ListNode prev = null;
            while(head!=null){
                ListNode next = head.next;
                head.next = prev;
                prev = head;
                if(next == null) return head;
                head = next;
            }
            return head;
        }
    }
    ```

## 36. 合并两个排序的链表

+ 题目描述：

    输入两个递增排序的链表，合并这两个链表并使新链表中的结点仍然是按照递增排序的。

    **样例：**

    输入：1->3->5 , 2->4->5

    输出：1->2->3->4->5->5

+ 解法：

    一个一个的比值，队列头和队列头比值，把小的往里放。最后在把非空队列全放入。

+ 代码：

    ``` java
    /**
    * Definition for singly-linked list.
    * public class ListNode {
    *     int val;
    *     ListNode next;
    *     ListNode(int x) { val = x; }
    * }
    */
    class Solution {
        public ListNode merge(ListNode l1, ListNode l2) {
            ListNode res = new ListNode(-1);
            ListNode node = res;
            while(l1!=null && l2!=null){
                if(l1.val<l2.val){
                    node.next = l1;
                    l1 = l1.next;
                }else{
                    node.next = l2;
                    l2 = l2.next;
                }
                node = node.next;
            }
            while(l1!=null){
                node.next = l1;
                l1 = l1.next;
                node = node.next;
            }
            while(l2!=null){
                node.next = l2;
                l2 = l2.next;
                node = node.next;
            }
            return res.next;
        }
    }
    ```

## 37. 树的子结构

+ 题目描述：

    输入两棵二叉树A，B，判断B是不是A的子结构。

    我们规定空树不是任何树的子结构。

    **样例：**

    ![树的字结构](/images/posts/problems/swordOffer/WX20191212-191245.png)

+ 解法：

    先比较两棵树的节点值是否一样，如果一样在递归比较同时的左边且同时的右边（即完全相同）。如果不一样，就比较左边和根或右边和根（即可能子树完全相同）。

+ 代码：

    ``` java
    /**
    * Definition for a binary tree node.
    * public class TreeNode {
    *     int val;
    *     TreeNode left;
    *     TreeNode right;
    *     TreeNode(int x) { val = x; }
    * }
    */
    class Solution {
        public boolean hasSubtree(TreeNode pRoot1, TreeNode pRoot2) {
            if(pRoot2==null) return false;
            return helper(pRoot1,pRoot2);
        }
        public boolean helper(TreeNode pRoot1, TreeNode pRoot2) {
            if(pRoot2==null) return true;
            if(pRoot1==null) return false;
            boolean res = false;
            if(pRoot1.val==pRoot2.val){
            res = helper(pRoot1.left,pRoot2.left) && helper(pRoot1.right,pRoot2.right);
            }
            if(res) return true;
            return helper(pRoot1.left,pRoot2) || helper(pRoot1.right,pRoot2);
        }
    }
    ```

## 38. 二叉树的镜像

+ 题目描述：

    输入一个二叉树，将它变换为它的镜像。

+ 解法：

    把树的左右节点对调，之后递归他的左节点和右节点。这样每个节点都做了镜像最后的结果就是都镜像了。

+ 代码：

    ``` java
    /**
    * Definition for a binary tree node.
    * public class TreeNode {
    *     int val;
    *     TreeNode left;
    *     TreeNode right;
    *     TreeNode(int x) { val = x; }
    * }
    */
    class Solution {
        public void mirror(TreeNode root) {
            if(root != null){
                TreeNode temp = root.left;
                root.left = root.right;
                root.right = temp;
                mirror(root.left);
                mirror(root.right);
            }
        }
    }
    ```

## 39. 对称的二叉树

+ 题目描述：

    请实现一个函数，用来判断一棵二叉树是不是对称的。

    如果一棵二叉树和它的镜像一样，那么它是对称的。

+ 解法：

    判断树的左子树值和树的右子树值是否相等，如果相等再继续判断两棵树间的左右子树和右左子树是否相等。这样就是一个递归问题。

+ 代码：

    ``` java
    /**
    * Definition for a binary tree node.
    * public class TreeNode {
    *     int val;
    *     TreeNode left;
    *     TreeNode right;
    *     TreeNode(int x) { val = x; }
    * }
    */
    class Solution {
        public boolean isSymmetric(TreeNode root) {
            if(root == null) return true;
            return helper(root.left, root.right);
        }
        public boolean helper(TreeNode node1, TreeNode node2){
            if(node1==node2) return true; // 两个都为 null
            if(node1==null||node2==null||node1.val!=node2.val) return false; // 只有一个为 null
            return helper(node1.left, node2.right) && helper(node1.right, node2.left);
        }
    }
    ```

## 40. 顺时针打印矩阵

+ 题目描述：

    输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字。

    **样例：**

    输入：

    [  
    [1, 2, 3, 4],  
    [5, 6, 7, 8],  
    [9,10,11,12]  
    ]

    输出：[1,2,3,4,8,12,11,10,9,5,6,7]

+ 解法：

    记录一下当前矩阵每一行，每一列遍历到的头是哪儿，尾是哪儿，然后对应着固定行或列转就好了。

+ 代码：

    ``` java
    class Solution {
        public int[] printMatrix(int[][] matrix) {
            if(matrix.length==0) return new int[]{};
            int[] res = new int[matrix.length*matrix[0].length];
            int rowStart = 0;
            int columnStart = 0;
            int rowEnd = matrix.length-1;
            int columnEnd = matrix[0].length-1;
            int row = 0;
            int column = 0;
            int resIndex = 0;
            while(rowStart<=rowEnd&&columnStart<=columnEnd){
                while(row == rowStart && column<=columnEnd&&resIndex<matrix.length*matrix[0].length){
                    res[resIndex++]=matrix[row][column++];
                    if(column>columnEnd){
                        column--;
                        rowStart++;
                        row++;
                        break;
                    }
                }
                while(column == columnEnd && row<=rowEnd&&resIndex<matrix.length*matrix[0].length){
                    res[resIndex++]=matrix[row++][column];
                    if(row>rowEnd){
                        row--;
                        columnEnd--;
                        column--;
                        break;
                    }
                }
                while(row == rowEnd && column>=columnStart&&resIndex<matrix.length*matrix[0].length){
                    res[resIndex++]=matrix[row][column--];
                    if(column<columnStart){
                        column++;
                        rowEnd--;
                        row--;
                        break;
                    }
                }
                while(column == columnStart && row>=rowStart&&resIndex<matrix.length*matrix[0].length){
                    res[resIndex++]=matrix[row--][column];
                    if(row<rowStart){
                        row++;
                        columnStart++;
                        column++;
                        break;
                    }
                }
            }
            return res;
        }
    }
    ```

## 41. 包含min函数的栈

+ 题目描述：

    设计一个支持push，pop，top等操作并且可以在O(1)时间内检索出最小元素的堆栈。

    + push(x)–将元素x插入栈中
    + pop()–移除栈顶元素
    + top()–得到栈顶元素
    + getMin()–得到栈中最小元素

+ 解法：

    用两个栈，一个栈记录每个元素，另一个栈记录到目前为止的最小值。

+ 代码：

    ``` java
    class MinStack {
        
        Stack<Integer> stack = new Stack<>();
        Stack<Integer> minStack = new Stack<>();//永远放当前最小的元素
        
        /** initialize your data structure here. */
        public MinStack() {
            
        }
        
        public void push(int x) {
            stack.push(x);
            if(minStack.isEmpty()) minStack.push(x);
            else{
                minStack.push(minStack.peek()>x?x:minStack.peek());
            }
        }
        
        public void pop() {
            if(!minStack.isEmpty()){
                minStack.pop();
                stack.pop();
            }
        }
        
        public int top() {
            return stack.isEmpty()?-1:stack.peek();
        }
        
        public int getMin() {
            return minStack.isEmpty()?-1:minStack.peek();
        }
    }

    /**
    * Your MinStack object will be instantiated and called as such:
    * MinStack obj = new MinStack();
    * obj.push(x);
    * obj.pop();
    * int param_3 = obj.top();
    * int param_4 = obj.getMin();
    */
    ```

## 42. 栈的压入、弹出序列

+ 题目描述：

    输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。

    假设压入栈的所有数字均不相等。

    例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。

    **注意：**若两个序列长度不等则视为并不是一个栈的压入、弹出序列。若两个序列都为空，则视为是一个栈的压入、弹出序列。

    **样例：**

    输入：[1,2,3,4,5] [4,5,3,2,1]

    输出：true

+ 解法：

    对于一个栈，每部操作只有两种可能。当当前栈顶元素和 pop 该输出的相等时，一定要推出。当不等，传入下一个元素。这样可以用一个栈模拟当前 push 的情况。模拟全过程。

+ 代码：

    ``` java
    class Solution {
        //每次对原栈只有两种操作，一种是压入下一个元素，一种是弹出栈顶元素，跟 pop 的比较，如果 pop 的下一个和当前栈顶一样，就一定是弹出来了，否则就是压入下一个了
        public boolean isPopOrder(int [] pushV,int [] popV) {
            if(pushV.length!=popV.length) return false;
            Stack<Integer> push = new Stack<>();//模拟当前栈中的情况
            int pushI = 0;
            int popI = 0;
            while(popI<popV.length){
                if(push.isEmpty()||push.peek()!=popV[popI]){
                    if(pushI==pushV.length) return false;
                    push.push(pushV[pushI]);
                    pushI++;
                }else{
                    push.pop();
                    popI++;
                }
            }
            return true;
        }
    }
    ```

## 43. 不分行从上往下打印二叉树

+ 题目描述：

    从上往下打印出二叉树的每个结点，同一层的结点按照从左到右的顺序打印。

+ 解法：

    广度搜索。由于他不涉及按层打印一类的问题，所以不需要记录每一层数量。

+ 代码：

    ``` java
    /**
    * Definition for a binary tree node.
    * public class TreeNode {
    *     int val;
    *     TreeNode left;
    *     TreeNode right;
    *     TreeNode(int x) { val = x; }
    * }
    */
    class Solution {
        public List<Integer> printFromTopToBottom(TreeNode root) {
            Queue<TreeNode> q = new LinkedList<>();
            List<Integer> list = new LinkedList<>();
            if(root==null) return list;
            q.add(root);
            while(q.size()>0){
                TreeNode node = q.poll();
                list.add(node.val);
                if(node.left!=null){
                    q.add(node.left);
                }
                if(node.right!=null){
                    q.add(node.right);
                }
            }
            return list;
        }
    }
    ```

## 45. 分行从上往下打印二叉树

+ 题目描述：

    从上到下按层打印二叉树，同一层的结点按从左到右的顺序打印，每一层打印到一行。

+ 解法：

    广度搜索。需要记录每一层数量。

+ 代码：

    ``` java
    /**
    * Definition for a binary tree node.
    * public class TreeNode {
    *     int val;
    *     TreeNode left;
    *     TreeNode right;
    *     TreeNode(int x) { val = x; }
    * }
    */
    class Solution {
        public List<List<Integer>> printFromTopToBottom(TreeNode root) {
            List<List<Integer>> res = new LinkedList<>();
            LinkedList<TreeNode> list = new LinkedList<>();
            if(root==null) return res;
            list.add(root);
            while(list.size()>0){
                int thisLineSize = list.size();
                List<Integer> oneLine = new LinkedList<>();
                for(int i=0;i<thisLineSize;i++){
                    TreeNode node = list.poll();
                    oneLine.add(node.val);
                    if(node.left!=null){
                        list.add(node.left);
                    }
                    if(node.right!=null){
                        list.add(node.right);
                    }
                }
                res.add(oneLine);
            }
            return res;
        }
    }
    ```

## 44. 分行从上往下打印二叉树

+ 题目描述：

    请实现一个函数按照之字形顺序从上向下打印二叉树。

    即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行以此类推。

+ 解法：

    广度搜索。需要记录每一层数量，使用双端 LinkedList，一轮从前出，一轮从后出，记录一个标识为标识这一层从左往右还是从右往左就可以了。

+ 代码：

    ``` java
    /**
    * Definition for a binary tree node.
    * public class TreeNode {
    *     int val;
    *     TreeNode left;
    *     TreeNode right;
    *     TreeNode(int x) { val = x; }
    * }
    */
    class Solution {
        public List<List<Integer>> printFromTopToBottom(TreeNode root) {
            List<List<Integer>> res = new LinkedList<>();
            LinkedList<TreeNode> list = new LinkedList<>();
            boolean firstToLast = false;
            if(root==null) return res;
            list.add(root);
            while(list.size()>0){
                int thisLineSize = list.size();
                LinkedList<Integer> oneLine = new LinkedList<>();
                firstToLast = !firstToLast;
                for(int i=0;i<thisLineSize;i++){
                    TreeNode node = list.poll();
                    if(firstToLast){
                        oneLine.add(node.val);
                    }else{
                        oneLine.addFirst(node.val);
                    }
                    if(node.left!=null){
                        list.add(node.left);
                    }
                    if(node.right!=null){
                        list.add(node.right);
                    }
                }
                res.add(oneLine);
            }
            return res;
        }
    }
    ```

## 45. 分行从上往下打印二叉树

+ 题目描述：

    从上到下按层打印二叉树，同一层的结点按从左到右的顺序打印，每一层打印到一行。

+ 解法：

    广度搜索。需要记录每一层数量。

+ 代码：

    ``` java
    /**
    * Definition for a binary tree node.
    * public class TreeNode {
    *     int val;
    *     TreeNode left;
    *     TreeNode right;
    *     TreeNode(int x) { val = x; }
    * }
    */
    class Solution {
        public List<List<Integer>> printFromTopToBottom(TreeNode root) {
            List<List<Integer>> res = new LinkedList<>();
            LinkedList<TreeNode> list = new LinkedList<>();
            if(root==null) return res;
            list.add(root);
            while(list.size()>0){
                int thisLineSize = list.size();
                List<Integer> oneLine = new LinkedList<>();
                for(int i=0;i<thisLineSize;i++){
                    TreeNode node = list.poll();
                    oneLine.add(node.val);
                    if(node.left!=null){
                        list.add(node.left);
                    }
                    if(node.right!=null){
                        list.add(node.right);
                    }
                }
                res.add(oneLine);
            }
            return res;
        }
    }
    ```

## 46. 二叉搜索树的后序遍历序列

+ 题目描述：

    输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。

    如果是则返回true，否则返回false。

    假设输入的数组的任意两个数字都互不相同。

+ 解法：

    二叉搜索树的后序遍历，中间节点也就在最后一位。所以先拿到 root 节点。之后利用左节点比 root 小，右节点比 root 大这一特点，分开左右边。然后验证右边的是不是都比 root 小。之后一直递归下去查看每一个子节点是否也都是二叉搜索树的后序遍历。

+ 代码：

    ``` java
    class Solution {
        //二叉搜索树的后续遍历最后一个是根，左子树都在右子树左边且左子树比根小，右子树比根大。也就是说找到左右子树分界点，判断右边的有没有比根小的就可以了
        //要一直递归到空
        public boolean verifySequenceOfBST(int [] sequence) {
            if(sequence==null||sequence.length==0){
                return true;
            }
            int root = sequence[sequence.length-1];
            int firstRight = 0;
            for(;firstRight<sequence.length-1;firstRight++){
                if(sequence[firstRight]>root) break;
            }
            int[] right = new int[sequence.length-2-firstRight+1];
            for(int rightI = firstRight;rightI<sequence.length-1;rightI++){
                if(sequence[rightI]<root){
                    return false;
                }
                right[rightI-firstRight]=sequence[rightI];
            }
            int[] left = new int[firstRight];
            for(int i=0;i<firstRight;i++){
                left[i] = sequence[i];
            }
            return verifySequenceOfBST(left)&&verifySequenceOfBST(right);
        }
    }
    ```

## 47. 二叉树中和为某一值的路径

+ 题目描述：

    输入一棵二叉树和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。

    从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。

+ 解法：

    用的是 DFS 回溯法，每次判断一次当前的 target 是否能为 0 了（其实它必须到叶子节点，想多了）。把轮到的节点的值加进去后记得要拿出来。因为这是左右都要用的。

+ 代码：

    ``` java
    /**
    * Definition for a binary tree node.
    * public class TreeNode {
    *     int val;
    *     TreeNode left;
    *     TreeNode right;
    *     TreeNode(int x) { val = x; }
    * }
    */
    class Solution {
        public List<List<Integer>> findPath(TreeNode root, int sum) {
            List<List<Integer>> res = new LinkedList<>();
            LinkedList<Integer> line = new LinkedList<>();
            helper(root,sum,res,line);
            return res;
        }
        //这个的回溯因为会调用左右两次，所以要保证 target 要在进入 node 之后减去，否则等着在他的子集去减可能会左右两边重复。
        //下面这个是不一定到底的，最早理解题意错了
        public void helper2(TreeNode node, int target, List<List<Integer>> res, LinkedList<Integer> line) {
            if(node == null) return;
            line.add(node.val);
            if(target==node.val){
                res.add(new ArrayList<>(line));
            }
            target-=node.val;
            helper2(node.left,target,res,line);
            if(node.left!=null)line.removeLast();;
            helper2(node.right,target,res,line);
            if(node.right!=null)line.removeLast();;
        }
        //必须到底的
        public void helper(TreeNode node, int target, List<List<Integer>> res, LinkedList<Integer> line) {
            if(node == null) return;
            line.add(node.val);
            if(node.left==null&&node.right==null){
                if(target==node.val){
                    res.add(new ArrayList<>(line));//要新建一个队列，不能使用原来的，右边的也会往这个队列里加的
                }
                return;
            }
            target-=node.val;
            helper(node.left,target,res,line);
            if(node.left!=null)line.removeLast();//不要用 remove(new Integer(node.left.val))，因为要顺序，这样可能会删除前面的重复数字
            helper(node.right,target,res,line);
            if(node.right!=null)line.removeLast();
        }
    }
    ```

## 48. 复杂链表的复刻

+ 题目描述：

    请实现一个函数可以复制一个复杂链表。

    在复杂链表中，每个结点除了有一个指针指向下一个结点外，还有一个额外的指针指向链表中的任意结点或者null。

+ 解法：

    复制链表中的每一个节点成为他下一个节点，再把节点全部单拿出来形成一个新链表。

+ 代码：

    ``` java
    /**
    * Definition for singly-linked list with a random pointer.
    * class ListNode {
    *     int val;
    *     ListNode next, random;
    *     ListNode(int x) { this.val = x; }
    * };
    */
    class Solution {
        public ListNode copyRandomList(ListNode head) {
            if(head==null) return null;
            ListNode node = head;
            while(node!=null){
                ListNode next = node.next;
                ListNode dup = new ListNode(node.val);
                node.next = dup;
                dup.next = next;
                node = node.next.next;
            }
            node = head;
            while(node!=null){
                node.next.random = node.random==null?null:node.random.next;
                node = node.next.next;
            }
            node = head;
            ListNode first = node.next;
            node.next = node.next.next;
            node = node.next;
            ListNode changeNode = first;
            while(node!=null){
                changeNode.next = node.next;
                node.next = node.next.next;
                node = node.next;
                changeNode = changeNode.next;
            }
            return first;
        }
    }
    ```

## 49. 二叉搜索树与双向链表

+ 题目描述：

    输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。

    要求不能创建任何新的结点，只能调整树中结点指针的指向。

    **注意：**

    需要返回双向链表最左侧的节点。

+ 解法：

    中序遍历顺序即为链表顺序。

+ 代码：

    ``` java
    /**
    * Definition for a binary tree node.
    * public class TreeNode {
    *     int val;
    *     TreeNode left;
    *     TreeNode right;
    *     TreeNode(int x) { val = x; }
    * }
    */
    class Solution {
        public TreeNode convert(TreeNode root) {
            if(root==null) return null;
            //中序遍历即可
            Stack<TreeNode> stack = new Stack<>();
            TreeNode node = root;
            TreeNode left = new TreeNode(-1);
            TreeNode first = left;
            while(node!=null||!stack.isEmpty()){
                while(node!=null){
                    stack.add(node);
                    node = node.left;
                }
                if(!stack.isEmpty()){
                    node = stack.pop();
                    left.right = node;
                    node.left = left;
                    left = left.right;
                    node = node.right;
                }
            }
            TreeNode res = first.right;
            res.left = null;
            return res;
        }
    }
    ```

## 50. 序列化二叉树

+ 题目描述：

    请实现两个函数，分别用来序列化和反序列化二叉树。

    您需要确保二叉树可以序列化为字符串，并且可以将此字符串反序列化为原始树结构

+ 解法：

    前序遍历在复原。必须用“，”分开，防止两位整数。复原时把树拆开成数组后，用一个指针记录下当前到的序列号位置，之后前序遍历恢复就行。

+ 代码：

    ``` java
    /**
    * Definition for a binary tree node.
    * public class TreeNode {
    *     int val;
    *     TreeNode left;
    *     TreeNode right;
    *     TreeNode(int x) { val = x; }
    * }
    */
    class Solution {
        //前序遍历
        // Encodes a tree to a single string.
        String serialize(TreeNode root) {
            if(root==null) return "#";
            StringBuilder sb = new StringBuilder();
            Stack<TreeNode> stack = new Stack<>();
            TreeNode node = root;
            while(!stack.isEmpty()||node!=null){
                while(node!=null){
                    sb.append(node.val+",");
                    stack.add(node);
                    node = node.left;
                }
                sb.append("#,");
                if(!stack.isEmpty()){
                    node = stack.pop();
                    node = node.right;
                }
            }
            return sb.toString().substring(0,sb.length()-1);
        }

        // Decodes your encoded data to tree.
        TreeNode deserialize(String data) {
            String[] trees = data.split(",");
            return helper(trees,new int[]{0});
            // return new TreeNode(Integer.parseInt(trees[1]));
        }
        TreeNode helper(String[] trees, int[] index){
            if(index[0]>=trees.length) return null;
            if(trees[index[0]].equals("#")){
                return null;
            }
            TreeNode t = new TreeNode(Integer.parseInt(trees[index[0]]));
            index[0]++;
            t.left = helper(trees, index);
            index[0]++;
            t.right = helper(trees, index);
            return t;
        }
    }
    ```

## 51. 数字排列

+ 题目描述：

    输入一组数字（可能包含重复数字），输出其所有的排列方式。

+ 解法：

    交换字符。循环，依次固定住字符串每个位置，交换后面的字符串。然后再交换固定住的字符串。

+ 代码：

    ``` java
    class Solution {
        public List<List<Integer>> permutation(int[] nums) {
            List<List<Integer>> res = new LinkedList<>();
            List<Integer> list = new LinkedList<>();
            helper(nums,0,res,list);
            return res;
        }
        public void helper(int[] nums,int index,List<List<Integer>> res,List<Integer> list){
            if(list.size()==nums.length){
                boolean has = false;
                for(List<Integer> l:res){
                    if(l.equals(list)) return;
                }
                res.add(new LinkedList<Integer>(list));
                return;
            }
            for(int i = index;i<nums.length;i++){
                swap(nums,i,index);
                list.add(nums[index]);
                helper(nums,index+1,res,list);//固定 index 位，交换后面位
                swap(nums,index,i);
                list.remove(list.size()-1);
            }
        }
        public void swap(int[] nums,int a,int b){
            int temp = nums[a];
            nums[a] = nums[b];
            nums[b] = temp;
        }
    }
    ```

## 52. 数组中出现次数超过一半的数字

+ 题目描述：

    数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。

    假设数组非空，并且一定存在满足条件的数字。

    **要求：**

    假设要求只能使用 O(n) 的时间和额外 O(1) 的空间，该怎么做呢？

+ 解法：

    最简单的方案，对数组排序，返回中间数字，但超时间了。

    如果多过了一半，那在数组中遍历如果遇到了这个数 +1，不是这个数 -1，数一定不会小于等于 0。

    还可以用快排的方式，partition 返回的 index 是目前保证左小右大的值，因为快排保证比 index 小的都在左边，大的都在右边。所以如果 index 在中间点就是所求值。

+ 代码：

    ``` java
    class Solution {
        public int moreThanHalfNum_Solution(int[] nums) {
            Arrays.sort(nums);
            return nums[nums.length/2];
        }
        // 如果多过了一半，那如果遇到了这个数 +1，不是这个数 -1，数一定不会小于等于 0。
        public int moreThanHalfNum_Solution2(int[] nums) {
            int n = 1;
            int num = nums[0];
            for(int i=1;i<nums.length;i++){
                if(nums[i]==num) n++;
                else n--;
                if(n==0){
                    n=1;
                    num=nums[i];
                }
            }
            n=0;
            return num;
        }
    }
    ```

## 53. 最小 K 个值

+ 题目描述：

    输入n个整数，找出其中最小的k个数。

+ 解法：

    可以借助快排的思想，找到快排的 index 为 k-1 时这样左边的值就都是最小的 k 个数了。

    或者可以借助优先队列（最小堆）。

+ 代码：

    ``` java
    class Solution {
        public List<Integer> getLeastNumbers_Solution(int [] input, int k) {
            List<Integer> list = new LinkedList<>();
            if(input.length==0){
                return list;
            }
            if(k>=input.length){
                Arrays.sort(input, 0, k);
                for(int i=0;i<input.length;i++){
                    list.add(input[i]);
                }
                return list;
            }
            //交换到返回的 index 序号是 k，证明他左边的 k-1 个数都比 k 小，即最小的 k 个数。
            int start = 0;
            int end = input.length-1;
            int index = paratitions(input,start,end);
            while(index!=k-1){ //当返回的序号不是 k，如果大于 k，就在他的左边，小于 k 就在他的右边。
                if(index>k-1){
                    end = index-1;
                    index = paratitions(input,start,end);
                }else{
                    start = index+1;
                    index = paratitions(input,start,end);
                }
            }
            Arrays.sort(input, 0, k);
            for(int i=0;i<k;i++){
                list.add(input[i]);
            }
            return list;
        }
        //把第一个值当临界值，使左边的值都比它小，右边的都比他大。返回的 index 是调整后的第一个值的位置。
        public int paratitions(int [] input,int start,int end){
            int num = input[start];
            while(start<end){
                while(start<end&&input[end]>num){
                    end--;
                }
                //把 num 看成了空槽，把位置不对的数和空槽交换
                swap(input,start,end);//现在 end 的位置成空槽了
                while(start<end&&input[start]<num){
                    start++;
                }
                swap(input,end,start);//现在 start 变成了空槽
            }
            return start;//一定返回的是 start，因为最后第一个数在的位置是 start
        }
        public void swap(int[] input,int a,int b){
            int temp = input[a];
            input[a]=input[b];
            input[b]=temp;
        }

        public List<Integer> getLeastNumbers_Solution2(int [] input, int k) {
            List<Integer> list = new LinkedList<>();
            if(input.length==0){
                return list;
            }
            if(k>=input.length){
                Arrays.sort(input, 0, k);
                for(int i=0;i<input.length;i++){
                    list.add(input[i]);
                }
                return list;
            }
            PriorityQueue<Integer> p = new PriorityQueue<>(k);
            for(int i=0;i<input.length;i++){
                p.add(input[i]);
            }
            for(int i =0;i<k;i++){
                list.add(p.poll());
            }
            return list;
        }
    }
    ```

## 54. 数据流中的中位数

+ 题目描述：

    如何得到一个数据流中的中位数？

    如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。

    如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。

    **样例：**

    输入：1, 2, 3, 4

    输出：1,1.5,2,2.5

    解释：每当数据流读入一个数据，就进行一次判断并输出当前的中位数。

+ 解法：

    两个队列，一个队列放小一半的数，一个放大一半的数。小一半的先出大数，大一半的先出小数。

    分奇偶进数，但要保证进入的数字一定还满足小的都在小队列，大的都在大队列。

    返回的时候这样奇数时从小队列取，偶数取平均值。

+ 代码：

    ``` java
    class Solution {
        
        PriorityQueue<Integer> q1 = new PriorityQueue<>((a,b)->b-a);//存小一半的数，先出大的
        PriorityQueue<Integer> q2 = new PriorityQueue<>();//存大一半的数，先出小的
        int count=0;
        
        public void insert(Integer num) {
            if(count%2==0){//证明目前里面的数将是奇数了，要把多出来的数字放到 q1 里，这时候要保证小一半里的数都小于大一半里的，所以要比较和大队列里最小的数谁大。
                if(!q2.isEmpty()&&q2.peek()<num){
                    int temp = num;
                    num = q2.poll();
                    q2.add(temp);
                }
                q1.add(num);
            }else{
                if(q1.peek()>num){//q1 这时候一定有数了，要让 q1 中的小，所以要比对新数和 q1 中最大的
                    int temp = num;
                    num = q1.poll();
                    q1.add(temp);
                }
                q2.add(num);
            }
            count++;
        }

        public Double getMedian() {
            if(count==0) return -1.0;
            if(count%2==0){
                return (q1.peek()+q2.peek())/2.0;
            }else{
                return (double)q1.peek();
            }
        }

    }
    ```

## 55. 连续子数组的最大和

+ 题目描述：

    输入一个 非空 整型数组，数组里的数可能为正，也可能为负。

    数组中一个或连续的多个整数组成一个子数组。

    求所有子数组的和的最大值。

    要求时间复杂度为O(n)。

    **样例：**

    输入：[1, -2, 3, 10, -4, 7, 2, -5]

    输出：18

+ 解法：

    连续最大到新的数的时候的最大值只可能是它本身或者它本身加上之前的连续数。

+ 代码：

    ``` java
    class Solution {
        public int maxSubArray(int[] nums) {
            if(nums.length==0) return -1;
            int max = nums[0];
            int currentMax = nums[0];
            for(int i=1;i<nums.length;i++){
                currentMax=Math.max(nums[i],nums[i]+currentMax);
                max = Math.max(max,currentMax);
            }
            return max;
        }
    }
    ```

## 56. 从1到n整数中1出现的次数

+ 题目描述：

    输入一个整数n，求从1到n这n个整数的十进制表示中1出现的次数。

    例如输入12，从1到12这些整数中包含“1”的数字有1，10，11和12，其中“1”一共出现了5次。

+ 解法：

    从左往右模拟人算。比如 12345 / 12315 就能找到规律。一位一位计算每一位有多少个 1.

    首先记录当前到了第几位，digit 每次乘 10。

    之后比如说到了倒数第二位，数字被分成了三部分 123 4 5。123 是左边的，4 是当前余数，5 是右边值。

    对于左边有可能有 000 1 * 到 122 1 * 共 123*10 种 1。（* 代表 0-9）

    对于右边则要区分当前余数。如果小于 1，则此位没有 1 了。如果等于 1 则有 ### 1 0 到 ### 1 5 共 右边值 +1 个 1。如果大于 1，则有 ### 1 0 到 ### 1 9 即 digit 个 1。

+ 代码：

    ``` java
    class Solution {
        public int numberOf1Between1AndN_Solution(int n) {
            int digit = 1;//现在到的位数
            int right = 0;//当前这位右边的值
            int sum = 0;
            while(n!=0){
                //算得都仅是这一位的 1 的数量
                int yushu = n%10;
                int left = n/10; //当前这位左边的值
                n=left;
                //比如 12315 算第一位时，左边 1231，也就有从 00001 到 12301，1231*1 个 1
                //倒数第二位时也就是左边 123，也就是 0001* 到 1221* 供 123*10 个1
                sum+=left*digit;
                //比较倒数第二位。
                //如果大于 1 如 12345，则要增加 ***10 到 ***19 个 1，即 digit 个 1
                if(yushu>1){
                    sum+=digit;
                }
                //如果是 1，则要增加12310 - 12315 即右边数 +1 个 1。
                else if(yushu==1){
                    sum+=right+1;
                }
                //如果到时第二位小于 1，则不可能再有 1 了。
                right = yushu*digit+right;//下一次右边的值
                digit*=10;
            }
            return sum;
        }
    }
    ```

## 57. 数字序列中某一位的数字

+ 题目描述：

    数字以0123456789101112131415…的格式序列化到一个字符序列中。

    在这个序列中，第5位（从0开始计数）是5，第13位是1，第19位是4，等等。

    请写一个函数求任意位对应的数字。

+ 解法：

    找出数字处在哪段（几位数）。之后找到跟上一次的位数差了多少个，算出偏移多少个数字的第几位。

+ 代码：

    ``` java
    class Solution {
        public int digitAtIndex(int n) {
            //0-9 10 个数
            //10-99 90 个数 9*10 一次方
            //100-999 900 个数 9*10 二次方
            //1000-9999 9000 个数 9*10三次饭
            if(n<10) return n;
            int digit = 2; //下一个该加多少
            int number = 10; //到目前有多少个位数
            int current = 1;//当前加到了几次方
            while(n-number>(long)digit*9*(current*10)){ //一定要谨防溢出
                number+=digit*9*(current*10);//几位数 * 多少个
                current*=10;
                digit++;
            }
            int rest = n-number;
            int num = rest/digit;
            int chaoguo = rest%digit;
            int thisNumber = current*10+num;
            return String.valueOf(thisNumber).charAt(chaoguo)-'0';
        }
    }
    ```
    
## 58. 把数组排成最小的数

+ 题目描述：

    输入一个正整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。

    例如输入数组[3, 32, 321]，则打印出这3个数字能排成的最小数字321323。

    **示例：**

    输入：[3, 32, 321]

    输出：321323

+ 解法：

    对数组排序，利用字符串 compareTo 方法，比较的时候比较两个字符串的拼接。

+ 代码：

    ``` java
    class Solution {
        public String printMinNumber(int[] nums) {
            //不能用优先队列
            // PriorityQueue<Integer> q = new PriorityQueue<>((Integer i1,Integer i2)->{
            //     String s1 = i1+""+i2;//必须是比较两个相加，比如说 123 和 12301 这时如果不想加 123 就在 12301 前面了
            //     String s2 = i2+""+i1;
            //     System.out.println(i1.getClass());
            //     return s1.compareTo(s2);
            // });
            LinkedList<Integer> q = new LinkedList<>();
            for(int num:nums){
                q.add(num);
            }
            Collections.sort(q,(i1,i2)->{
                String s1 = i1+""+i2;//必须是比较两个相加，比如说 123 和 12301 这时如果不想加 123 就在 12301 前面了
                String s2 = i2+""+i1;
                // System.out.println(i1.getClass());
                return s1.compareTo(s2);
            });
            StringBuilder sb = new StringBuilder();
            q.forEach(e->sb.append(e));
            return sb.toString();
        }
    }
    ```

## 59. 把数字翻译成字符串

+ 题目描述：

    给定一个数字，我们按照如下规则把它翻译为字符串：

    0翻译成”a”，1翻译成”b”，……，11翻译成”l”，……，25翻译成”z”。

    一个数字可能有多个翻译。例如12258有5种不同的翻译，它们分别是”bccfi”、”bwfi”、”bczi”、”mcfi”和”mzi”。

    请编程实现一个函数用来计算一个数字有多少种不同的翻译方法。

    **示例：**

    输入："12258"

    输出：5

+ 解法：

    简单的话可以用回溯法，如果连续两位的数小于 25，则可以一次跨两个数。

    第二种方法是动态规划，dp[i] 表示到第 i（比字符串实际 +1）个数有多少可能，递推公式 dp[i] = dp[i-1] + dp[i-2] dp[i-2] 要看是否满足条件，必须是 length+1 因为默认长度为 0 也是一种情况。

+ 代码：

    ``` java
    class Solution {
        public int getTranslationCount(String s) {
            if(s.equals("")) return 0;
            int[] count = new int[1];
            helper(s, 0, count);
            return count[0];
        }
        public void helper(String s, int index, int[] count){
            if(index==s.length()){
                count[0]++;
                return;
            }
            if(index<s.length()-1&&s.charAt(index)!='0'&&Integer.parseInt(s.substring(index,index+2))<26){
                helper(s, index+2, count);
            }
            helper(s, index+1, count);
        }

        public int getTranslationCount2(String s) {
            if(s.equals("")){
                return 0;
            }
            char[] cs = s.toCharArray();
            int[] dp = new int[cs.length+1]; // 递推公式 dp[i] = dp[i-1] + dp[i-2] dp[i-2] 要看是否满足条件，必须是 length+1 因为默认长度为 0 也是一种情况
            dp[0] = 1;
            dp[1] = 1;
            for(int i=1;i<cs.length;i++){
                dp[i+1] = dp[i];
                if(cs[i-1]=='1'||(cs[i-1]=='2'&&cs[i]<'6')){
                    dp[i+1] += dp[i-1];
                }
            }
            return dp[cs.length];
        }
    }
    ```

## 60. 礼物的最大价值

+ 题目描述：

    在一个m×n的棋盘的每一格都放有一个礼物，每个礼物都有一定的价值（价值大于0）。

    你可以从棋盘的左上角开始拿格子里的礼物，并每次向右或者向下移动一格直到到达棋盘的右下角。

    给定一个棋盘及其上面的礼物，请计算你最多能拿到多少价值的礼物？

    **示例：**

    输入：  
    [  
    [2,3,1],  
    [1,7,1],  
    [4,6,1]  
    ]  

    输出：19

    解释：沿着路径 2→3→7→6→1 可以得到拿到最大价值礼物。

+ 解法：

    动态规划，最原始的二维可以写为：dp[i][j] = Math.max(dp[i-1][j],dp[i][j-1])+grid[i][j]

    由于只跟上和左有关，可以简化成一维数组，dp[i] = Math.max(dp[i-1],dp[i])+grid[j][i]

+ 代码：

    ``` java
    class Solution {
        public int getMaxValue(int[][] grid) {
            if(grid.length==0) return 0;
            int[] dp = new int[grid[0].length]; //递推公式 dp[i] = Math.max(dp[i-1],dp[i])+grid[i][j]，只跟他左边或上边有关，左边即 dp[i-1]，上面是 dp[i]
            for(int j=0;j<grid.length;j++){
                for(int i=0;i<grid[0].length;i++){
                    if(i==0){
                        dp[0] += grid[j][i];
                    }else{
                        dp[i] = Math.max(dp[i-1],dp[i])+grid[j][i];
                    }
                }
            }
            return dp[grid[0].length-1];
        }
    }
    ```

## 61. 最长不含重复字符的子字符串

+ 题目描述：

    请从字符串中找出一个最长的不包含重复字符的子字符串，计算该最长子字符串的长度。

    假设字符串中只包含从’a’到’z’的字符。

    **示例：**

    输入："abcabc"

    输出：3

+ 解法：

    用一个 26 的数组记录出现的数字上次出现的位置。用一个 index 记录目前最远点到了哪里。如果出现过且在最远点之内，就要更新最远点为出现位置 +1。

+ 代码：

    ``` java
    class Solution {
        public int longestSubstringWithoutDuplication(String s) {
            int[] record = new int[26]; //存储 a 到 z 出没出现过
            for(int i=0;i<26;i++){
                record[i]=-1;
            }
            char[] cs = s.toCharArray();
            int maxLength = 0;
            int index = 0;//目前到最远的指针地址
            for(int i=0;i<cs.length;i++){
                char c = cs[i];
                if(record[c-'a']==-1||record[c-'a']<index){ //后面的判断条件是为了看出现的字符会不会是目前 index 之前的，之前的就相当于没出现过
                    record[c-'a']=i;
                }else{
                    maxLength = Math.max(i-index,maxLength);
                    index = record[c-'a']+1;
                    record[c-'a']=i;
                    
                }
            }
            maxLength = Math.max(cs.length-index,maxLength);
            return maxLength;
        }
    }
    ```

## 62. 丑数

+ 题目描述：

    我们把只包含质因子2、3和5的数称作丑数（Ugly Number）。

    例如6、8都是丑数，但14不是，因为它包含质因子7。

    求第n个丑数的值。

+ 解法：

    丑数都是 2、3、5 组成的，也就是说每个丑数对应 2、3、5 中的一个 index。找到各自 index 每次都乘 2、3、5 取最小的就是下一个丑数。同时用循环更新最近的 2、3、5 的 index，之后再乘 2、3、5 必有一个是接下来的最小值。

+ 代码：

    ``` java
    class Solution {
        public int getUglyNumber(int n) {
            int i2 = 0;// i2, i3, i5 表示的是他最近的值在 res 中的 index
            int i3 = 0;
            int i5 = 0;
            int[] res = new int[n];
            res[0] = 1;
            int i = 1;
            while(i<n){
                // 每个最 index 的值乘 2 3 5 中最小的就是下一个的值
                int min = Math.min(res[i2]*2,Math.min(res[i3]*3,res[i5]*5));
                res[i++] = min;
                while(res[i2]*2<=min) i2++;
                while(res[i3]*3<=min) i3++;
                while(res[i5]*5<=min) i5++;
            }
            return res[n-1];
        }
    }
    ```

## 63. 字符串中第一个只出现一次的字符

+ 题目描述：

    在字符串中找出第一个只出现一次的字符。

    如输入"abaccdeff"，则输出b。

    如果字符串中不存在只出现一次的字符，返回#字符。

    **样例：**

    输入："abaccdeff"

    输出：'b'

+ 解法：

    第一次遍历用一个 map 存储出现字符和次数，第二次遍历读取存储的数量为 1 的返回。

+ 代码：

    ``` java
    class Solution {
        public char firstNotRepeatingChar(String s) {
            Map<Character,Integer> map = new HashMap<>();
            char[] cs = s.toCharArray();
            for(int i=0;i<cs.length;i++){
                Integer num = map.get(cs[i]);
                if(num == null){
                    num = 0;
                }
                num+=1;
                map.put(cs[i],num);
            }
            for(int i=0;i<cs.length;i++){
                if(map.get(cs[i]) == 1){
                    return cs[i];
                }
            }
            return '#';
        }
    }
    ```

## 64. 字符流中第一个只出现一次的字符

+ 题目描述：

    请实现一个函数用来找出字符流中第一个只出现一次的字符。

    例如，当从字符流中只读出前两个字符”go”时，第一个只出现一次的字符是’g’。

    当从该字符流中读出前六个字符”google”时，第一个只出现一次的字符是’l’。

    如果当前字符流没有存在出现一次的字符，返回#字符。

    **样例：**

    输入："google"

    输出："ggg#ll"

    解释：每当字符流读入一个字符，就进行一次判断并输出当前的第一个只出现一次的字符。

+ 解法：

    用一个 map 存储出现次数，list 存储顺序。没有出现过就加到 map 同时加到 map 里。出现 1 次就再有就从 list 中移出去。

    还有第二种思路是字符最多256个，创建一个 256 大小的数组和一个 index 数字。index 大小代表放进去的顺序。数组默认是 0 如果是 0 也就代表数字没出现过。如果不是 0 代表出现一次，这时再添加时就代表数字出现将不止一次，就把数置为 -2。在找第一个时，寻找最小的正数就可以了。

+ 代码：

    ``` java
    class Solution {
        Map<Character,Integer> map = new HashMap<>();
        LinkedList<Character> list = new LinkedList<>();
        //Insert one char from stringstream   
        public void insert(char ch){
            Integer num = map.get(ch);
            if(num==null){
                list.add(ch);
                map.put(ch,1);
            }else if(num==1){
                list.remove(new Character(ch));
                map.put(ch,2);
            }
        }
        //return the first appearence once char in current stringstream
        public char firstAppearingOnce(){
            if(list.isEmpty()) return '#';
            return list.peek();
        }
    }
    ```

## 65. 数组中的逆序对

+ 题目描述：

    在数组中的两个数字如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。

    输入一个数组，求出这个数组中的逆序对的总数。

    **样例：**

    输入：[1,2,3,4,5,6,0]

    输出：6

+ 解法：

    用归并排序的方法来做。这样分成两半后，每一半都是有序的。当一般中一个数大于另一半中某个数时，这个数后面的所有数也都大于这个数。在完成后最终数组也都排序了。

+ 代码：

    ``` java
    class Solution {
        int res = 0;
        public int inversePairs(int[] nums) {
            int[] copys = new int[nums.length];
            mergeSort(nums, copys, 0, nums.length-1);
            return res;
        }
        public void mergeSort(int[] nums, int[] copys, int start, int end){
            if(start>=end) return;
            int mid = start+(end-start)/2;
            mergeSort(nums, copys, start, mid);
            mergeSort(nums, copys, mid+1, end);
            // 到这个地方的时候，start 到 mid 和 mid 到 end 间都有序了。
            // 两个数先从头比，如果一个数的某个位置比另一半某个位置大了，他后面的所有数字也都大于另一半这个数。
            int index = start; // 对应 temp 中的 index，因为原数组只是分段有序，先让 copys 有序，再复制。
            int i = start;
            int j = mid+1;
            while(i<=mid && j<=end){
                if(nums[i]>nums[j]){
                    copys[index++] = nums[j++];
                    res+=mid-i+1; // 他后面的数也都比他大
                }else{
                    copys[index++] = nums[i++];
                }
            }
            while(i<=mid) copys[index++] = nums[i++];
            while(j<=end) copys[index++] = nums[j++];
            while(start<=end){
                nums[start] = copys[start];
                start++;
            }
        }
    }
    ```

## 66. 两个链表的第一个公共结点

+ 题目描述：

    输入两个链表，找出它们的第一个公共结点。

    当不存在公共节点时，返回空节点。

+ 解法：

    先求出两个栈的长度，让长的先走差值，这样汇合点就会同时到达。

    或者都把节点放到两个栈中，找到第一个不相同的节点。

+ 代码：

    ``` java
    /**
    * Definition for singly-linked list.
    * public class ListNode {
    *     int val;
    *     ListNode next;
    *     ListNode(int x) {
    *         val = x;
    *         next = null;
    *     }
    * }
    */
    class Solution {
        public ListNode findFirstCommonNode(ListNode headA, ListNode headB) {
            ListNode nodeA = headA;
            ListNode nodeB = headB;
            int countA = 0;
            int countB = 0;
            while(nodeA!=null){
                nodeA = nodeA.next;
                countA++;
            }
            while(nodeB!=null){
                nodeB = nodeB.next;
                countB++;
            }
            if(countA>countB){
                int walk = countA-countB;
                for(int i=0;i<walk;i++){
                    headA = headA.next;
                }
            }else{
                int walk = countB-countA;
                for(int i=0;i<walk;i++){
                    headB = headB.next;
                }
            }
            while(headA!=headB){
                if(headA==null||headB==null) return null;
                headA = headA.next;
                headB = headB.next;
            }
            return headA;
        }
    }
    ```