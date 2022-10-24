---
title: "String.intern()问题"
layout: page
date: 2018-11-24 00:00
---

[TOC]

# Java字符串相关

## intern()方法例子

```java
String yesA = "aaabbb";
String yesB = new String("aaa") + new String("bbb");
String yesC = yesB.intern();
System.out.println(yesA == yesB); // false
System.out.println(yesA == yesC); // ture
```

intern 方法的作用是，判断下 yesB 引用指向的值在字符串常量里面是否有，如果没有就在字符串常量池里面新建一个 aaabbb 对象，返回其引用，如果有则直接返回引用。

```java
  /**
     * Returns a canonical representation for the string object.
     * <p>
     * A pool of strings, initially empty, is maintained privately by the
     * class {@code String}.
     * <p>
     * When the intern method is invoked, if the pool already contains a
     * string equal to this {@code String} object as determined by
     * the {@link #equals(Object)} method, then the string from the pool is
     * returned. Otherwise, this {@code String} object is added to the
     * pool and a reference to this {@code String} object is returned.
     * <p>
     * It follows that for any two strings {@code s} and {@code t},
     * {@code s.intern() == t.intern()} is {@code true}
     * if and only if {@code s.equals(t)} is {@code true}.
     * <p>
     * All literal strings and string-valued constant expressions are
     * interned. String literals are defined in section 3.10.5 of the
     * <cite>The Java&trade; Language Specification</cite>.
     *
     * @return  a string that has the same contents as this string, but is
     *          guaranteed to be from a pool of unique strings.
     */
    public native String intern();
```

----

```java
String yesB = new String("aaa") + new String("bbb");
String yesC = yesB.intern();
String yesA = "aaabbb"; 
System.out.println(yesA == yesB); // true
System.out.println(yesA == yesC); // true
```

1. String yesB = new String("aaa") + new String("bbb");

此时，堆内会新建一个 aaabbb 对象(对于 aaa 和 bbb 的对象讨论忽略)，字符串常量池里不会创建，因为并没有出现 aaabbb 这个字面量。

2. String yesC = yesB.intern();

此时，会在字符串常量池内部创建 aaabbb 对象？关键点来了。
在 JDK 1.6 时，字符串常量池是放置在**永久代**(非堆，堆后面的一块区域)的，所以必须新建一个对象放在常量池中。
但 JDK 1.7 之后字符串常量池是放在**堆内**的，而堆里已经有了刚才 new 过的 aaabbb 对象，所以没必要浪费资源，不用再存储一份对象，直接存储堆中的引用即可。

所以 yesC 这个常量存储的引用和 yesB 一样。

3. String yesA = "aaabbb";

同理，在 1.7 中 yesA 得到的引用与 yesC 和 yesB 一致，都指向堆内的 aaabbb 对象。

4. 最终的答案都是 true

<font color='red'>得站在不同的 JDK 版本来回答，不然就是错的<font>
