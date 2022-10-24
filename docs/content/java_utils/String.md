---
title: "String,StringBuffer,StringBuilder"
layout: page
date: 2018-11-24 00:00
---

[TOC]

# Java字符串相关

## String

* String是个`Final`类

Final类的对象是只读的，可以在多线程环境下安全的共享，不用额外的同步开销等。<font color='red'>安全、高效</a>

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used fo r character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -6849794470754667710L;

    /**
     * Class String is special cased within the Serialization Stream Protocol.
     *
     * A String instance is written into an ObjectOutputStream according to
     * <a href="{@docRoot}/../platform/serialization/spec/output.html">
     * Object Serialization Specification, Section 6.2, "Stream Elements"</a>
     */
    private static final ObjectStreamField[] serialPersistentFields =
        new ObjectStreamField[0];

    /**
     * Initializes a newly created {@code String} object so that it represents
     * an empty character sequence.  Note that use of this constructor is
     * unnecessary since Strings are immutable.
     */
    public String() {
        this.value = "".value;
    }
```

* charAt方法，throw `StringIndexOutOfBoundsException`

```java
public char charAt(int index) {
        if ((index < 0) || (index >= value.length)) {
            throw new StringIndexOutOfBoundsException(index);
        }
        return value[index];
    }
```

* String的equals方法

判断了两个String对象，首先是否同一个对象，然后是否长度相等，接着判断每一个字符是否相等。

```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

* String的`substring`、`concat`,`replace`都是返回新的String对象，原有对象不会被改变

```java
 public String substring(int beginIndex, int endIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    if (endIndex > value.length) {
        throw new StringIndexOutOfBoundsException(endIndex);
    }
    int subLen = endIndex - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
    return ((beginIndex == 0) && (endIndex == value.length)) ? this
            : new String(value, beginIndex, subLen);
}
```

```java
public String concat(String str) {
    int otherLen = str.length();
    if (otherLen == 0) {
        return this;
    }
    int len = value.length;
    char buf[] = Arrays.copyOf(value, len + otherLen);
    str.getChars(buf, len);
    return new String(buf, true);
}
```

```java
 public String replace(char oldChar, char newChar) {
    if (oldChar != newChar) {
        int len = value.length;
        int i = -1;
        char[] val = value; /* avoid getfield opcode */

        while (++i < len) {
            if (val[i] == oldChar) {
                break;
            }
        }
        if (i < len) {
            char buf[] = new char[len];
            for (int j = 0; j < i; j++) {
                buf[j] = val[j];
            }
            while (i < len) {
                char c = val[i];
                buf[i] = (c == oldChar) ? newChar : c;
                i++;
            }
            return new String(buf, true);
        }
    }
    return this;
}
```

* 比较字符串的大小`compareTo`

```java
 public int compareTo(String anotherString) {
    int len1 = value.length;
    int len2 = anotherString.value.length;
    int lim = Math.min(len1, len2);
    char v1[] = value;
    char v2[] = anotherString.value;

    int k = 0;
    while (k < lim) {
        char c1 = v1[k];
        char c2 = v2[k];
        if (c1 != c2) {
            return c1 - c2;
        }
        k++;
    }
    return len1 - len2;
}
```

### split方法

```java
String s = "aa.a..ccc.b"; // "aa" "a" "" "ccc" "b"
String[] arr = s.split("\\.");// `.`必须转义, 如果连续`.`,则会split空格出来
System.out.println(arr);
```

* 根据java指定的字符，分割字符串的方法是：String temp[]=result.split(",");
* "."和"|"都是转义字符，必须得加"\\";

## new String("xx") 和 = "xx"的区别

```java
public class Main {

    // main 线程
    public static void main(String[] args) {
        String s1 = "test";
        String s2 = "test";
        System.out.println("s1和s2是同一个字符串" + (s1 == s2));//true

        String s3 = new String("test");
        System.out.println("s1和s3是同一个字符串" + (s1 == s3));//false
        String s4 = "tes" + "t";
        System.out.println("s1和s4是同一个字符串" + (s1 == s4));//true
        System.out.println("s3和s4是同一个字符串" + (s3 == s4));//false

        String s5 = new String("test");
        System.out.println("s1和s5是同一个字符串" + (s1 == s5));//false
        System.out.println("s3和s5是同一个字符串" + (s3 == s5));//false

        String s6 = s3.intern();
        System.out.println("s1和s6是同一个字符串" + (s1 == s6));//true
        System.out.println("s3和s6是同一个字符串" + (s3 == s6));//false

        String s7 = "test";
        System.out.println("s1和s7是同一个字符串" + (s1 == s7));//true
        System.out.println("s3和s7是同一个字符串" + (s3 == s7));//false
        System.out.println("s6和s7是同一个字符串" + (s6 == s7));//true

        String s8 = new String("add");
        String s9 = "add";
        System.out.println("s8和s9是同一个字符串" + (s8 == s9));//false

        String s10 = new String("delete").intern();
        String s11 = "delete";
        System.out.println("s10和s11是同一个字符串" + (s10 == s11));//true
    }

}
```

* string = "xx"，如果常量池中存在这个字符串，那么不会产生新的字符串，会引用已经存在的字符串，如果常量池中没有，则会在常量池中生成一个
* 每次new String("xx")会在堆中创建一个"xx"对象,然后在常量池中也创建一个对象,这俩不是同一个对象；如果常量池中已经有"xx"对象，则不需要在常量池中创建；但是堆中只要new就会产生对象，且不可被引用
* String.intern()方法会返回常量池中的"xx"对象,如果常量池已有则直接返回;如果没有,就把堆中的对象放入常量池,他俩是同一个对象,然后返回

### new String("xx")创建了几个对象

问：`String s=new String("xyz")`创建了几个对象，分为两种情况：

1. 如果String常量池中，已经创建了"xyz"，则不会继续创建，此时只创建了一个对象new String("xyz")；
2. 如果String常量池中，没有创建"xyz"，则会创建两个对象：一个对象是常量池中的"xyz"，一个对象是堆中new String("xyz")。

### 不可变类有什么好处？

最主要的好处就是**安全**，因为知晓这个对象不可能会被修改，在多线程环境下也是线程安全的

你想想看，你引用的对象是一个不可变的值，那么谁都无法修改它，那它永远就是不变的，别的线程也休息动它分毫，你可以放心大胆的用。

然后，配合常量池可以节省内存空间，且获取效率也更高（如果常量池里面已经有这个字符串对象了，就不需要新建，直接返回即可）。

## StringBuffer（synchronized修饰）

final类，继承了`AbstractStringBuilder`

### 线程安全

StringBuffer基本上所有的操作方法都加上了`synchronized`修饰，保证线程安全

```java
 public final class StringBuffer
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence
{

    /**
     * A cache of the last value returned by toString. Cleared
     * whenever the StringBuffer is modified.
     */
    private transient char[] toStringCache;

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    static final long serialVersionUID = 3388685877147921107L;

    /**
     * Constructs a string buffer with no characters in it and an
     * initial capacity of 16 characters.
     */
    public StringBuffer() {
        super(16);
    }

    /**
     * Constructs a string buffer with no characters in it and
     * the specified initial capacity.
     *
     * @param      capacity  the initial capacity.
     * @exception  NegativeArraySizeException  if the {@code capacity}
     *               argument is less than {@code 0}.
     */
    public StringBuffer(int capacity) {
        super(capacity);
    }

    /**
     * Constructs a string buffer initialized to the contents of the
     * specified string. The initial capacity of the string buffer is
     * {@code 16} plus the length of the string argument.
     *
     * @param   str   the initial contents of the buffer.
     */
    public StringBuffer(String str) {
        super(str.length() + 16);
        append(str);
    }

    /**
     * Constructs a string buffer that contains the same characters
     * as the specified {@code CharSequence}. The initial capacity of
     * the string buffer is {@code 16} plus the length of the
     * {@code CharSequence} argument.
     * <p>
     * If the length of the specified {@code CharSequence} is
     * less than or equal to zero, then an empty buffer of capacity
     * {@code 16} is returned.
     *
     * @param      seq   the sequence to copy.
     * @since 1.5
     */
    public StringBuffer(CharSequence seq) {
        this(seq.length() + 16);
        append(seq);
    }
```

* `append`方法之一

```java
@Override
public synchronized StringBuffer append(String str) {
    toStringCache = null;
    super.append(str);
    return this;
}

public AbstractStringBuilder append(String str) {
    if (str == null)
        return appendNull();
    int len = str.length();
    ensureCapacityInternal(count + len);
    str.getChars(0, len, value, count);
    count += len;
    return this;
}

public void getChars(int srcBegin, int srcEnd, char dst[], int dstBegin) {
    if (srcBegin < 0) {
        throw new StringIndexOutOfBoundsException(srcBegin);
    }
    if (srcEnd > value.length) {
        throw new StringIndexOutOfBoundsException(srcEnd);
    }
    if (srcBegin > srcEnd) {
        throw new StringIndexOutOfBoundsException(srcEnd - srcBegin);
    }
    System.arraycopy(value, srcBegin, dst, dstBegin, srcEnd - srcBegin);
}
```

### 对于大量的字符串操作，StringBuffer优于String

```java
final static int time = 10000;

public static void testString () {
    String s="";
    long begin = System.currentTimeMillis();
    for(int i=0; i<time; i++){
        s += "java";
    }
    long over = System.currentTimeMillis();
    System.out.println("操作"+s.getClass().getName()+"类型使用的时间为："+(over-begin)+"毫秒");
}

public static void testStringBuffer () {
    StringBuffer sb = new StringBuffer();
    long begin = System.currentTimeMillis();
    for(int i=0; i<time; i++){
        sb.append("java");
    }
    long over = System.currentTimeMillis();
    System.out.println("操作"+sb.getClass().getName()+"类型使用的时间为："+(over-begin)+"毫秒");
}
```

## StringBuilder

### 非线程安全

适用于单线程下在字符缓冲区进行大量操作的情况

### append字符串 对比 String 快

```java
// 对于String，会不断的创建、回收对象，速度会很慢。
static void testStringAdd(){
    int num = 10000;
    String[] arrs = new String[num];
    for(int i=0; i<arrs.length; ++i) {
        arrs[i] = String.valueOf(i * i);
    }
    String s = "";
    long start = System.currentTimeMillis();
    for(int i=0; i<arrs.length; ++i) {
        s += arrs[i];
    }
    System.out.println(s.length());
    long end = System.currentTimeMillis();
    System.out.println("cost:" + (end - start));
}

static void testStringBuilderAdd(){
    int num = 10000;
    String[] arrs = new String[num];
    for(int i=0; i<arrs.length; ++i) {
        arrs[i] = String.valueOf(i * i);
    }
    StringBuilder stringBuilder = new StringBuilder();
    long start = System.currentTimeMillis();
    for(int i=0; i<arrs.length; ++i) {
        stringBuilder.append(arrs[i]);
    }
    String s = stringBuilder.toString();
    System.out.println(s.length());
    long end = System.currentTimeMillis();
    System.out.println("cost:" + (end - start));
}
```

### reverse方法

```java
StringBuilder sb = new StringBuilder("abc");
String revSb = sb.reverse().toString();
System.out.println(revSb); // cba
```

## Java String类为什么是final的？

* value，offset和count这三个变量都是private的，并且没有提供setValue， setOffset和setCount等公共方法来修改这些值，所以在String类的外部无法修改String

1. 为了实现字符串池

字符串池的实现可以在运行时节约很多`heap`空间，因为不同的字符串变量都指向池中的同一个字符串。但如果字符串是可变的，那么String intern将不能实现，因为这样的话，如果变量改变了它的值，那么其它指向这个值的变量的值也会一起改变

2. 为了安全 & 线程安全

final不可变，确保线程安全

3. 为了实现String可以创建HashCode不可变性，提高效率

因为字符串是不可变的，所以在它创建的时候HashCode就被缓存了，不需要重新计算。这就使得字符串很适合作为Map中的键，字符串的处理速度要快过其它的键对象。这就是HashMap中的键往往都使用字符串。

## final的底层JVM实现

对于final域，编译器和处理器要遵守两个重排序规则：

1. 在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。

（先写入final变量，后调用该对象引用）；原因：编译器会在final域的写之后，插入一个`StoreStore`屏障

2. 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。

（先读对象的引用，后读final变量）；编译器会在读final域操作的前面插入一个`LoadLoad`屏障
