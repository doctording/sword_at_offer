---
title: "数组，集合转换等问题"
layout: page
date: 2019-06-25 00:00
---

[TOC]

# 数组，二维数组，集合

## List<Integer> 与 int[] 与 Integer[]

```java
List<Integer> list = Arrays.asList(1, 3, 2);
Integer[] arr = new Integer[list.size()];
list.toArray(arr);
for(int i=0;i<arr.length;i++) {
    System.out.print(arr[i] + (i==arr.length?"":" "));
}
System.out.println();

int[] arrInt = list.stream().mapToInt(Integer::valueOf).toArray();
for(int i=0;i<arrInt.length;i++) {
    System.out.print(arrInt[i] + (i==arr.length?"":" "));
}
System.out.println();
```
