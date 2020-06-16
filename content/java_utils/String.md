---
title: "String,StringBuffer,StringBuilder"
layout: page
date: 2018-11-24 00:00
---

[TOC]

# Java字符串相关

## String

* String是个`Final`类

Final类的对象是只读的，可以在多线程环境下安全的共享，不用额外的同步开销等。

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

* charAt方法,throw `StringIndexOutOfBoundsException`

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

* 比较字符串的大小`compareTo`

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

## StringBuffer

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

## StringBuilder > StringBuffer > String

### 非线程安全

适用于单线程下在字符缓冲区进行大量操作的情况

### append字符串 对比 string +

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

## Java String类为什么是final的？

1. 为了实现字符串池

字符串池的实现可以在运行时节约很多`heap`空间，因为不同的字符串变量都指向池中的同一个字符串。但如果字符串是可变的，那么String intern将不能实现，因为这样的话，如果变量改变了它的值，那么其它指向这个值的变量的值也会一起改变

2. 为了安全 & 线程安全

final不可变,安全

3. 为了实现String可以创建HashCode不可变性，提高效率

因为字符串是不可变的，所以在它创建的时候HashCode就被缓存了，不需要重新计算。这就使得字符串很适合作为Map中的键，字符串的处理速度要快过其它的键对象。这就是HashMap中的键往往都使用字符串。
