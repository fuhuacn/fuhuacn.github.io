---
layout: post
title: LeetCode 链表专题
categories: Prolems
description: LeetCode 链表专题
keywords: leetcode,链表
---

目录

* TOC
{:toc}

## 19. 删除链表的倒数第N个节点 中等

* 题目描述

    给定一个链表，删除链表的倒数第 n 个节点，并且返回链表的头结点。

    **Example:**

    >给定一个链表: 1->2->3->4->5, 和 n = 2. 当删除了倒数第二个节点后，链表变为 1->2->3->5.

* 解法

    提前走 n 个节点。需要注意特殊情况，比如说要删除的是第一个节点。

* 代码

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
        public ListNode removeNthFromEnd(ListNode head, int n) {
            ListNode node = head;
            ListNode res = head;
            for(int i=0;i<n;i++){
                node = node.next;
            }
            if(node == null) return res.next; // 表示删除的是第一个节点
            while(node.next!=null){
                node = node.next;
                head = head.next;
            }
            head.next = head.next.next;
            return res;
        }
    }
    ```

## 83. 删除排序链表中的重复元素 简单

* 题目描述

    给定一个排序链表，删除所有重复的元素，使得每个元素只出现一次。

    **Example:**

    >输入: 1->1->2->3->3 输出: 1->2->3

* 解法

    循环删除，但一定要注意最后一个重复时有可能走到 null。导致空指针报错。

* 代码

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
        public ListNode deleteDuplicates(ListNode head) {
            ListNode preNode = new ListNode(Integer.MIN_VALUE);
            ListNode first = preNode;
            ListNode node = head;
            while(node!=null){
                while(node!=null && node.val == preNode.val){ // 一定注意最后一个 node 有可能为空
                    node = node.next;
                }
                preNode.next = node;
                preNode = node;
                if(node!=null){
                    node = node.next;
                }
            }
            return first.next;
        }
        // 之前的想的太复杂了
        public ListNode deleteDuplicates2(ListNode head) {
            ListNode first = head;
            while(first!=null && first.next!=null){ //first 下一个为空也就没有重复了
                if(first.val == first.next.val){
                    first.next = first.next.next;
                }else{
                    first = first.next;
                }
            }
            return head;
        }
    }
    ```