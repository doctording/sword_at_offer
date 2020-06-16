---
title: "int(基本类型) & Integer(包装类类型)"
layout: page
date: 2019-03-24 00:00
---

[TOC]

# int & Integer

* Java8程序验证

```java
package com.mb;

public class Main {

    public static void main(String[] args) throws Exception {
        int i = 128;
        Integer i2 = 128;
        Integer i3 = new Integer(128);
        //Integer会自动拆箱为int，所以为true
        System.out.println(i == i2);    // true
        System.out.println(i == i3);    // true
        System.out.println("**************");
        Integer i5 = 127;//java在编译的时候,被翻译成-> Integer i5 = Integer.valueOf(127);
        Integer i6 = 127;
        System.out.println(i5 == i6);   //true, 127走了缓存，i5 i6是同一个对象
        Integer i5_1 = 128;
        Integer i6_1 = 128;
        System.out.println(i5_1 == i6_1);//false, 128不走缓存，i5_1 i6_1不是同一个对象
        Integer ii5 = new Integer(127);
        System.out.println(i5 == ii5);  // false， ii5是new出来的新对象
        Integer i7 = new Integer(128);
        Integer i8 = new Integer(123);
        System.out.println(i7 == i8);   // false
    }
}
/*output

true
true
**************
true
false
false
false

 */
```

* 建议是：包装类型间的判断使用`equals`, 而不是`==`

## 源码分析（Java 8）

## `valueOf(int i)` 源码（享元模式）

```java
/**
    * Cache to support the object identity semantics of autoboxing for values between
    * -128 and 127 (inclusive) as required by JLS.
    *
    * The cache is initialized on first usage.  The size of the cache
    * may be controlled by the {@code -XX:AutoBoxCacheMax=<size>} option.
    * During VM initialization, java.lang.Integer.IntegerCache.high property
    * may be set and saved in the private system properties in the
    * sun.misc.VM class.
    */

private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}

// valueOf 会使用 IntegerCache
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

对于`-128`到`127`之间的数，会进行缓存，`Integer i = 127`时，会将`127`进行缓存，下次再写`Integer j = 127`时，就会直接从缓存中取，就不会`new`了

eg:

```java
public class Main {
    public static void main(String[] args) {

        Integer i1 = 100;
        Integer i2 = 100;
        Integer i3 = 200;
        Integer i4 = 200;

        System.out.println(i1==i2);
        System.out.println(i3==i4);
    }
}
```

输出：

```java
true
false
```

## `equals(Object obj)`源码

```java
public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue();
    }
    return false;
}
```

## 自动装箱 & 拆箱

* `int`的**自动装箱**都是通过`Integer.valueOf()`方法来实现的

* `Integer`的**自动拆箱**都是通过`Integer.intValue`来实现的

## 基本类型与包装类型的异同

* 存储方式及位置的不同，基本类型是直接存储变量的值，保存在堆栈中能高效的存取；封装类型需要通过引用指向实例，具体的实例保存在堆中；

* 初始值的不同，封装类型的初始值为null，基本类型的的初始值视具体的类型而定，比如int类型的初始值为0，boolean类型为false；

* 使用方式的不同，比如与集合类合作使用时只能使用包装类型。

* 什么时候该用包装类，什么时候该用基本类型，看基本的业务来定：这个字段允不允许null值，如果允许，则必然要用封装类；否则，基本类型就可以了。如果用到比如泛型和反射调用函数，就需要用包装类！
