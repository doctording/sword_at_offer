---
title: "快排(没事手写写)"
layout: page
date: 2018-11-24 00:00
---

[TOC]

# Java快排

```java
int partion(int[] arr, int left, int right){
    int pivot = arr[left];
    int i = left;
    int j = right;
    while(i < j){
        while(i < j && arr[j] >= pivot){
            j --;
        }
        arr[i] = arr[j];
        while(i < j && arr[i] < pivot){
            i ++;
        }
        arr[j] = arr[i];
    }
    arr[i] = pivot;
    return i;
}

void quickSort(int[] arr, int left, int right){
    if(left >= right){
        return;
    }
    int pos = partion(arr, left, right);
    quickSort(arr, left, pos - 1);
    quickSort(arr, pos + 1, right);
}

public static void main(String[] args) throws Exception {
    Solution solution = new Solution();
    int[] arr = {4,3,5, 6,8,1, 4, 2,7,9};
    int n = arr.length;
    for(int i=0;i<n;i++){
        System.out.print(arr[i] + " ");
    }
    System.out.println();

    solution.quickSort(arr, 0, n - 1);

    for(int i=0;i<n;i++){
        System.out.print(arr[i] + " ");
    }

}
```
