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
