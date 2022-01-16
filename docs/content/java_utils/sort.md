---
title: "常见排序"
layout: page
date: 2018-11-24 00:00
---

[TOC]

# 常见排序

## 数组快排

```java
class Solution {
    // leetcode 912. 排序数组
    public int[] sortArray(int[] nums) {
        if(nums == null || nums.length == 0){
            return new int[0];
        }
        quickSort(nums, 0, nums.length - 1);
        return nums;
    }

    void quickSort(int[] arr, int left, int right){
        int p = partition(arr, left, right);
        if(p != -1){
            quickSort(arr, left, p - 1);
            quickSort(arr, p + 1, right);
        }
    }

    int partition(int[] arr, int left, int right){
        if(left > right){
            return -1;
        }
        int pivot = arr[left];
        int low = left;
        int high = right;
        while(low < high){
            while(low < high && arr[high] >= pivot){
                high --;
            }
            arr[low] = arr[high];
            while(low < high && arr[low] < pivot){
                low ++;
            }
            arr[high] = arr[low];
        }
        arr[low] = pivot;
        return low;
    }
}
```

## 数组堆排序

大根堆

```java
class Solution {
    // leetcode 912. 排序数组
    public int[] sortArray(int[] nums) {
        if(nums == null || nums.length == 0){
            return new int[0];
        }
        heapSort(nums, nums.length);
        return nums;
    }

    void heapSort(int[] arr, int n){
        // build big heap
        for(int i=n/2;i>=0;i--){
            adjust(arr, i, n);
        }

        for(int i=n-1;i>=1;i--){
            // 与堆顶交换
            swap(arr, 0, i);
            // 然后剩下的继续构造堆
            adjust(arr, 0, i);
        }
    }

    void adjust(int[] arr, int index, int n){
        if(index < n){
            int left = 2 * (index + 1) - 1;
            int right = 2 * (index + 1) + 1 - 1;
            if(left >= n && right >= n){
                return;
            }else if(left < n && right >= n){
                if(arr[left] > arr[index]){
                    swap(arr, left, index);
                    adjust(arr, left, n);
                }
            }else if(left >= n && right < n){
                if(arr[right] > arr[index]){
                    swap(arr, right, index);
                    adjust(arr, right, n);
                }
            }else {
                if(arr[left] >= arr[right]){
                    if(arr[left] > arr[index]){
                        swap(arr, left, index);
                        adjust(arr, left, n);
                    }
                }else {
                    if(arr[right] > arr[index]){
                        swap(arr, right, index);
                        adjust(arr, right, n);
                    }
                }
            }
        }
    }

    void swap(int[] arr, int i, int j){
        int tmp = arr[i];
        arr[i] = arr[j];
        arr[j] = tmp;
    }
}
```

## 数组归并排序

地址：https://leetcode.com/problems/sort-an-array/， `O(n)`的空间复杂度

```java
class Solution {
    public int[] sortArray(int[] nums) {
        if (nums == null || nums.length == 0) {
            return nums;
        }
        int n = nums.length;
        int[] tmp = new int[n];
        sortMerge(nums, 0, n-1, tmp);
        return nums;
    }
    
    void sortMerge(int[] nums, int left, int right, int[] tmp) {
        if (left >= right) {
            return;
        }
        int mid = (right - left) / 2 + left;
        sortMerge(nums, left, mid, tmp);
        sortMerge(nums, mid + 1, right, tmp);
        merge(nums, left, mid, right, tmp);
    }
        
    void merge(int[] nums , int left , int mid , int right , int[] tmp) {
        int index = left;
        int i = left;
        int j = mid +1;
        while (i <= mid && j <= right) {
            if (nums[i] <= nums[j]) {
                tmp[index ++] = nums[i ++];
            } else {
                tmp[index++] = nums[j ++];
            }
        }
        while (i <= mid) {
            tmp[index ++] = nums[i ++];
        }
        while (j <= right) {
            tmp[index ++] = nums[j ++];
        }
        for (int k = left; k <= right; k ++) {
            nums[k] = tmp[k];
        }
    }

}
```

## 链表归并排序

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    // leetcode 148 归并排序
    public ListNode sortList(ListNode head) {
        if(head == null || head.next == null){
            return head;
        }
        ListNode fast = head;
        ListNode slow = head;
        ListNode pre = null;
        while (fast != null && fast.next != null){
            fast = fast.next.next;
            pre = slow;
            slow = slow.next;
        }
        // 断开链表
        if(pre != null){
            pre.next = null;
        }
        ListNode left = sortList(head);
        ListNode right = sortList(slow);
        return mergeTwoLists(left, right);
    }

    // leetcode 21 合并两个有序链表
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode node = new ListNode(-1);
        ListNode nodeTail = node;
        ListNode p = l1;
        ListNode q = l2;
        while (p != null && q != null) {
            if (p.val < q.val) {
                ListNode pNext = p.next;

                nodeTail.next = p;
                nodeTail = p;
                nodeTail.next = null;

                p = pNext;
            } else {
                ListNode qNext = q.next;

                nodeTail.next = q;
                nodeTail = q;
                nodeTail.next = null;

                q = qNext;
            }
        }
        while (p != null) {
            ListNode pNext = p.next;

            nodeTail.next = p;
            nodeTail = p;
            nodeTail.next = null;

            p = pNext;
        }
        while (q != null) {
            ListNode qNext = q.next;

            nodeTail.next = q;
            nodeTail = q;
            nodeTail.next = null;

            q = qNext;
        }
        return node.next;
    }
}
```
