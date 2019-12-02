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

    采用交换策略，即数组都在 0 - n-1 内，从头开始把对应的值和对应的位置做交换，当发现值一样的时候就不交换也就证明此时冲突了。其实这种思想也可以想象成链表，对应的值是对应的链表位置，跳到对应的链表位置在找下一个链表的位置，撞上了即结束。

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