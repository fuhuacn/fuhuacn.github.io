---
layout: post
title: 二分法与栈专题
categories: Prolems
description: 二分法与栈专题
keywords: 二分法,栈
---

目录

* TOC
{:toc}

## 69. x 的平方根 简单

* 题目描述

    实现 int sqrt(int x) 函数。

    计算并返回 x 的平方根，其中 x 是非负整数。

    由于返回类型是整数，结果只保留整数的部分，小数部分将被舍去。

    **Example:**

    > 输入: 8  
    输出: 2  
    说明: 8 的平方根是 2.82842...,  
    由于返回类型是整数，小数部分将被舍去。

* 解法

    二分法，要取右中位数，并且右边界减一，因为要取下界值。

* 代码

    ``` java
    class Solution {
        public int mySqrt(int x) {
            long left = 0;
            long right = x/2+1;
            while(left<right){
                long mid = right+(left-right)/2;
                if(mid*mid>x){
                    right = mid-1;
                }else if(mid*mid<x){
                    left = mid;
                }else{
                    return (int)mid;
                }
            }
            return (int)left;
        }
    }
    ```

## 34. 在排序数组中查找元素的第一个和最后一个位置 中等

* 题目描述

    给定一个按照升序排列的整数数组 nums，和一个目标值 target。找出给定目标值在数组中的开始位置和结束位置。

    你的算法时间复杂度必须是 O(log n) 级别。

    如果数组中不存在目标值，返回 [-1, -1]。


    **Example:**

    > 输入: nums = [5,7,7,8,8,10], target = 8  
    输出: [3,4]

* 解法

    两次二分法，一次取数组左限，一次取数组右限。注意取整，那个方向有中间 +1 或 -1 就往那个方向取整。

* 代码

    ``` java
    class Solution {
        public int[] searchRange(int[] nums, int target) {
            int[] res = new int[2];
            if(nums.length==0){
                res[0] = -1;
                res[1] = -1;
                return res;
            }
            //两次二分，一次找上界，一次找下界
            int left = 0;
            int right = nums.length-1;
            while(left<right){
                int mid = left+(right-left)/2; //小取整，因为是左 +1
                if(nums[mid]<target){ //只要小于目标值，证明最左限一定在他右边
                    left = mid+1;
                }else{
                    right = mid; //即使相等也保持不动，一步一步的把 mid 值带下去，下取整当两数一样时也会取下值。
                }
            }
            res[0] = left;
            if(nums[left]!=target){
                res[0] = -1;
                res[1] = -1;
                return res;
            }
            left = 0;
            right = nums.length-1;
            while(left<right){
                int mid = right+(left-right)/2; //大取整，因为是右 -1
                if(nums[mid]>target){
                    right = mid-1;
                }else{
                    left = mid;
                }
            }
            res[1] = right;
            return res;
        }
    }
    ```

## 74. 搜索二维矩阵 中等

* 题目描述

    编写一个高效的算法来判断 m x n 矩阵中，是否存在一个目标值。该矩阵具有如下特性：

    - 每行中的整数从左到右按升序排列。
    - 每行的第一个整数大于前一行的最后一个整数。

    **Example:**

    > 输入:  
    matrix = [  
    [1,   3,  5,  7],  
    [10, 11, 16, 20],  
    [23, 30, 34, 50]  
    ]  
    target = 3  
    输出: true

* 解法

    把二维矩形拉成一个长数组二分法。/ % 定位在二维数组中的位置。

* 代码

    ``` java
    class Solution {
        public boolean searchMatrix(int[][] matrix, int target) {
            //把二维数组拉平就可以用二分法做
            int y = matrix.length;
            if(y==0) return false;
            int x = matrix[0].length;
            int left = 0;
            int right = x*y-1;
            while(left<=right){
                int mid = left+(right-left)/2;
                int value = matrix[mid/x][mid%x];
                if(value==target) return true;
                else if(value>target){
                    right = mid-1;
                }else{
                    left = mid+1;
                }
            }
            return false;
        }
    }
    ```

## 240 搜索二维矩阵 中

* 题目描述

  Write an efficient algorithm that searches for a value in an m x n matrix. This matrix has the following properties:

  * Integers in each row are sorted in ascending from left to right.
  * Integers in each column are sorted in ascending from top to bottom.

    **Example:**  
    > Consider the following matrix:
    [  
    [1,   4,  7, 11, 15],  
    [2,   5,  8, 12, 19],  
    [3,   6,  9, 16, 22],  
    [10, 13, 14, 17, 24],  
    [18, 21, 23, 26, 30]  
    ]  
    Given target = 5, return true.  
    Given target = 20, return false.

* 解法

    抓住每个元素左边比他小，下边比他大的特点，从右上角（即头）开始遍历。

* 代码

    ``` java
    class Solution {
        //抓住每个元素左边比他小，下边比他大的特点，从右上角（即头）开始遍历
        public boolean searchMatrix(int[][] matrix, int target) {
            if(matrix.length==0) return false;
            int columns = matrix[0].length;
            int rows = matrix.length;
            int i = 0;
            int j = columns-1;
            while(i<rows && j>-1){
                int num = matrix[i][j];
                if(num == target) return true;
                else if(num<target) i++;
                else if(num>target) j--;
            }
            return false;
        }
    }
    ```

## 153. 寻找旋转排序数组中的最小值 中等

* 题目描述

    假设按照升序排序的数组在预先未知的某个点上进行了旋转。

    ( 例如，数组 [0,1,2,4,5,6,7] 可能变为 [4,5,6,7,0,1,2] )。

    请找出其中最小的元素。

    你可以假设数组中不存在重复元素。

    **Example:**  
    > 输入: [3,4,5,1,2]  
    输出: 1

* 解法

    二分法，把中间数跟最左边数比，如果比最左边数小，就证明就是最小的数或者在最小数右边。反之在左边。

* 代码

    ``` java
    class Solution {
        public int findMin(int[] nums) {
            if(nums.length==0) return 0;
            if(nums[nums.length-1]>=nums[0]) return nums[0];
            int start = 0;
            int end = nums.length-1;
            int left = nums[0];
            while(start<end){
                int mid = start+(end-start)/2;
                if(nums[mid]>=left){ //比左边数大，证明一定不是最小值还在右边，如果等，证明就是最左边的数，由于最左边肯定不是最小的，所以还在右边
                    start = mid+1;
                }else{
                    end = mid;
                }
            }
            return nums[start];
        }
    }
    ```

## 162. 寻找峰值 中等

* 题目描述

    峰值元素是指其值大于左右相邻值的元素。

    给定一个输入数组 nums，其中 nums[i] ≠ nums[i+1]，找到峰值元素并返回其索引。

    数组可能包含多个峰值，在这种情况下，返回任何一个峰值所在位置即可。

    你可以假设 nums[-1] = nums[n] = -∞。

    **Example:**  
    > 输入: nums = [1,2,1,3,5,6,4]  
    输出: 1 或 5  
    解释: 你的函数可以返回索引 1，其峰值元素为 2；  
    或者返回索引 5， 其峰值元素为 6。

* 解法

    由于左右两边都取负无穷，所以可以用二分法。找到中间值，用中间值和中间值下一个比较，如果中间值大于中间值下一个，说明左边一定存在峰值。反之右边窜中峰值

* 代码

    ``` java
    class Solution {
        //由于左右两边都取负无穷，所以可以用二分法。找到中间值，用中间值和中间值下一个比较，如果中间值大于中间值下一个，说明左边一定存在峰值。反之右边窜中峰值
        public int findPeakElement(int[] nums) {
            if(nums.length==1) return 0;
            int left=0;
            int right=nums.length-1;
            while(left<right){
                int mid = left+(right-left)/2;
                if(nums[mid]>nums[mid+1]){
                    right = mid;
                }else{
                    left = mid+1;
                }
            }
            return left;
        }
    }
    ```

## 155. 最小栈 简单

* 题目描述

    设计一个支持 push，pop，top 操作，并能在常数时间内检索到最小元素的栈。

    - push(x) -- 将元素 x 推入栈中。
    - pop() -- 删除栈顶的元素。
    - top() -- 获取栈顶元素。
    - getMin() -- 检索栈中的最小元素。

    **Example:**  
    > 略

* 解法

    两个栈，一个放最小值的栈，两个栈大小同步。

* 代码

    ``` java
    class MinStack {
        
        Stack<Integer> stack = new Stack<>();
        Stack<Integer> minStack = new Stack<>();

        /** initialize your data structure here. */
        public MinStack() {
            
        }
        
        public void push(int x) {
            stack.push(x);
            if(minStack.size()==0 || x<minStack.peek()){
                minStack.push(x);
            }else{
                minStack.push(minStack.peek());
            }
        }
        
        public void pop() {
            if(stack.size()<=0) return;
            stack.pop();
            minStack.pop();
        }
        
        public int top() {
            return stack.peek();
        }
        
        public int getMin() {
            return minStack.peek();
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

## 496. 下一个更大元素 I 简单

* 题目描述

    给定两个没有重复元素的数组 nums1 和 nums2 ，其中nums1 是 nums2 的子集。找到 nums1 中每个元素在 nums2 中的下一个比其大的值。

    nums1 中数字 x 的下一个更大元素是指 x 在 nums2 中对应位置的右边的第一个比 x 大的元素。如果不存在，对应位置输出-1。

    **Example:**  
    > 输入: nums1 = [4,1,2],   nums2 = [1,3,4,2].  
    输出: [-1,3,-1]  
    解释:  
    对于num1中的数字4，你无法在第二个数组中找到下一个更大的数字，因此输出 -1。  
    对于num1中的数字1，第二个数组中数字1右边的下一个较大数字是 3。
    对于num1中的数字2，第二个数组中没有下一个更大的数字，因此输出 -1。

* 解法

    一个栈，一个 map，栈遇到小的循环弹出，最后从 map 里取。

* 代码

    ``` java
    class MinStack {
        
        Stack<Integer> stack = new Stack<>();
        Stack<Integer> minStack = new Stack<>();

        /** initialize your data structure here. */
        public MinStack() {
            
        }
        
        public void push(int x) {
            stack.push(x);
            if(minStack.size()==0 || x<minStack.peek()){
                minStack.push(x);
            }else{
                minStack.push(minStack.peek());
            }
        }
        
        public void pop() {
            if(stack.size()<=0) return;
            stack.pop();
            minStack.pop();
        }
        
        public int top() {
            return stack.peek();
        }
        
        public int getMin() {
            return minStack.peek();
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