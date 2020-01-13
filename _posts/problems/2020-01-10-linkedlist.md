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

## 206. 反转链表 简单

* 题目描述

    反转一个单链表。

    **Example:**

    >输入: 1->2->3->4->5->NULL  
    输出: 5->4->3->2->1->NULL

* 解法

    循环和递归两种方式。

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
        public ListNode reverseList(ListNode head){
            ListNode prev = null;
            ListNode next = head;
            while(next!=null){
                ListNode temp = next.next;
                next.next = prev;
                prev = next;
                next = temp;
            }
            return prev;
        }

        ListNode res = null;
        public ListNode reverseList2(ListNode head) {
            if(head == null) return null;
            helper(head);
            return res;
        }
        public ListNode helper(ListNode head){
            if(head.next == null){
                res = head;
                return head;
            }
            ListNode node = helper(head.next);
            node.next = head;
            head.next = null; // 要用这话，目的是首个链表下一个设为 null
            return head;
        }

        public ListNode reverseList3(ListNode head){
            if(head==null||head.next == null){
                return head;
            }
            ListNode p = reverseList(head.next); //记录下的最深的节点，永远返回 p
            head.next.next = head; //反转就是让他的下一个节点指向自己
            head.next = null; // 要用这话，目的是首个链表下一个设为 null
            return p;
        }
    }
    ```

## 92. 反转链表 II 中等

* 题目描述

    反转从位置 m 到 n 的链表。请使用一趟扫描完成反转。

    **Example:**

    >输入: 1->2->3->4->5->NULL, m = 2, n = 4  
    输出: 1->4->3->2->5->NULL

* 解法

    先提前走 m 位。然后开始反转。之后在拼接头拼尾，尾拼头。

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
        public ListNode reverseBetween(ListNode head, int m, int n) {
            if(head==null) return null;
            ListNode res = head;
            ListNode prev = null; // 反转后头连接的地方
            for(int i=0;i<m-1;i++){
                prev = head;
                head = head.next;
            }
            ListNode firstReverse = head; // 第一个开始反转的点，未来他就是最后一个点，要连接之后的下一个
            ListNode before = null;
            for(int i=m;i<=n;i++){
                ListNode temp = head.next;
                head.next = before;
                before = head;
                head = temp;
            }
            // 循环完后，此时的 head 是反转后的下一个，也就是反转后的头应该连接的地方。before 应该是 prev 连接的地方。
            firstReverse.next = head;
            if(prev!=null){
                prev.next = before;
                return res;
            }else{ //prev 为空证明从第一个数就开始反转
                return before;
            }
        }
    }
    ```

## 61. 旋转链表 中等

* 题目描述

    给定一个链表，旋转链表，将链表每个节点向右移动 k 个位置，其中 k 是非负数。

    **Example:**

    >输入: 1->2->3->4->5->NULL, k = 2  
    输出: 4->5->1->2->3->NULL  
    解释:  
    向右旋转 1 步: 5->1->2->3->4->NULL  
    向右旋转 2 步: 4->5->1->2->3->NULL

* 解法

    其实就是把倒数长度取余数个节点移到链表头。求出余数后，用快慢节点方法找到移的位置，把它设为返回值，尾部连接到头就可以了。

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
        public ListNode rotateRight(ListNode head, int k) {
            if(head == null) return null;
            // k%长度 就是把倒数第几个节点移到头
            ListNode node = head;
            ListNode tail = null; // 尾节点以后直接接到头
            int count = 0;
            while(node!=null){
                tail = node;
                node = node.next;
                count++;
            }
            int rotate = k%count;
            if(rotate == 0) return head;
            ListNode slow = head;
            node = head;
            // 快慢节点，先走 k 步
            for(int i=0;i<rotate;i++){
                node = node.next;
            }
            while(node.next!=null){
                node = node.next;
                slow = slow.next;
            }
            ListNode res = slow.next;
            slow.next = null;
            tail.next = head;
            return res;
        }
    }
    ```

## 61.重排链表 中等

* 题目描述

    给定一个单链表 L：L0→L1→…→Ln-1→Ln ，

    将其重新排列后变为： L0→Ln→L1→Ln-1→L2→Ln-2→…

    你不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。

    **Example:**

    >给定链表 1->2->3->4->5, 重新排列为 1->5->2->4->3.

* 解法

    比较直观的一个方法是把链表分成两部分，后半部分放到栈里。分成两部分可以用快慢指针，后半部分可以用反转链表。

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
        public void reorderList(ListNode head) {
            if(head == null) return;
            // 把整个链表一分为二，后一半扔到栈里，往外吐
            ListNode node = head;
            ListNode fast = head;
            while(fast!=null && fast.next!=null){
                node = node.next;
                fast = fast.next.next;
            }
            // 前一半不动，后一半放到栈里
            Stack<ListNode> stack = new Stack<>();
            while(node!=null){
                ListNode temp = node.next; 
                node.next = null;
                stack.push(node);
                node = temp;
            }
            node = head;
            while(stack.size()>0){
                ListNode temp = stack.pop();
                temp.next = node.next;
                node.next = temp;
                node = node.next.next;
            }
            node.next = null;
        }
    }
    ```

## 21. 合并两个有序链表 简单

* 题目描述

    将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

    **Example:**

    >输入：1->2->4, 1->3->4  
    输出：1->1->2->3->4->4

* 解法

    两个链表依次把小的往里面添加就好了。

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
        public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
            ListNode head = new ListNode(-1);
            ListNode res = head;
            while(l1!=null && l2!=null){
                if(l1.val<=l2.val){
                    head.next = l1;
                    l1 = l1.next;
                }else{
                    head.next = l2;
                    l2 = l2.next;
                }
                head = head.next;
            }
            while(l1!=null){
                head.next = l1;
                l1 = l1.next;
                head = head.next;
            }
            while(l2!=null){
                head.next = l2;
                l2 = l2.next;
                head = head.next;
            }
            return res.next;
        }
    }
    ```

## 160. 相交链表 简单

* 题目描述

    编写一个程序，找到两个单链表相交的起始节点。

    **Example:**

    >略

* 解法

    用两个指针分别从两个链表头部开始扫描，每次分别走一步；
    如果指针走到null，则从另一个链表头部开始走；
    当两个指针相同时，
    如果指针不是null，则指针位置就是相遇点；
    如果指针是 null，则两个链表不相交；

* 代码

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
    public class Solution {
        public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
            // 两个指针节点分别从两个头走当一个走到尾后，从另一个头开始走。
            // 相当于所有节点都会走一轮长短路径
            ListNode nodeA = headA;
            ListNode nodeB = headB;
            while(nodeA!=nodeB){
                if(nodeA == null){
                    nodeA = headB;
                }else{
                    nodeA = nodeA.next;
                }
                if(nodeB == null){
                    nodeB = headA;
                }else{
                    nodeB = nodeB.next;
                }
            }
            return nodeA;
        }
    }
    ```