---
title: "《Java编程思想》第3章：操作符，第4章：控制执行流程"
layout: page
date: 2019-02-14 00:00
---

[TOC]

# 8种基本类型

```java
System.out.println("=================");
/*
Byte: 8
Short: 16
Character: 16
Integer: 32
Float: 32
Long: 64
Double: 64
Boolean: false
*/
System.out.println("Byte: " + Byte.SIZE);
System.out.println("Short: " + Short.SIZE);
System.out.println("Character: " + Character.SIZE);
System.out.println("Integer: " + Integer.SIZE);
System.out.println("Float: " + Float.SIZE);
System.out.println("Long: " + Long.SIZE);
System.out.println("Double: " + Double.SIZE);
System.out.println("Boolean: " + Boolean.toString(false));

System.out.println("=================");
/*
Byte: [-128,127]
Short: [-32768,32767]
Integer: [-2147483648,2147483647]
Float: [1.401298e-45,3.402823e+38]
Double: [4.900000e-324,1.797693e+308]
*/
System.out.println(String.format("Byte: [%d,%d]", Byte.MIN_VALUE, Byte.MAX_VALUE));
System.out.println(String.format("Short: [%d,%d]", Short.MIN_VALUE, Short.MAX_VALUE));
System.out.println(String.format("Integer: [%d,%d]", Integer.MIN_VALUE, Integer.MAX_VALUE));
System.out.println(String.format("Float: [%e,%e]", Float.MIN_VALUE, Float.MAX_VALUE));
System.out.println(String.format("Double: [%e,%e]", Double.MIN_VALUE, Double.MAX_VALUE));
```

# 类型转换

类型转换（cast）， 在适当的时候,Java会将一种数据类型自动的转换为另一种。

窄化转化（narrowing conversion） 即：将能容纳更多信息的数据类型转换成无法容纳那么多信息的类型，这可能面临信息丢失的危险

# switch

* byte，short，char能够隐含的转化为int，可以作为switch； long 不能隐含转换为int，不能switch

* switch enum 可结合使用

不同Java版本可能支持不一样，目前Java8也支持String的switch
