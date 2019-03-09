---
title: "System.arraycopy, Array.copyOf"
layout: page
date: 2018-11-24 00:00
---

[TOC]

# System.arraycopy

使用说明：

```java
/*
* @param      src      the source array.
* @param      srcPos   starting position in the source array.
* @param      dest     the destination array.
* @param      destPos  starting position in the destination data.
* @param      length   the number of array elements to be copied.
* @exception  IndexOutOfBoundsException  if copying would cause
*               access of data outside array bounds.
* @exception  ArrayStoreException  if an element in the <code>src</code>
*               array could not be stored into the <code>dest</code> array
*               because of a type mismatch.
* @exception  NullPointerException if either <code>src</code> or
*               <code>dest</code> is <code>null</code>.
*/
public static native void arraycopy(Object src,  int  srcPos,
                                Object dest, int destPos,
                                int length);
```

## 浅拷贝

```java
public static void testArraycopy(){
    class Obj{
        int a;
        String b;
        public Obj(){
            a=0;
            b="";
        }
        public Obj(int _a, String _b){
            a= _a;
            b=_b;
        }
    }
    int N = 3;
    Obj obj1 = new Obj(1, "a");
    Obj obj2 = new Obj(2, "b");
    Obj obj3 = new Obj(3, "c");

    Obj[] st  = {obj1, obj2, obj3};
    Obj[] dt  = new Obj[N];
    System.arraycopy(st, 0, dt, 0, N);

    // false
    System.out.println("两个数组地址是否相同：" + (st == dt));
    for(int i=0;i<N;i++){
        // true
        System.out.println("两个数组内容"+i+"是否相同：" + (st[0] == dt[0]));
    }
}
```

## 对比for效率高

```java
public static void testArrayCopyOfEfficient(){
    final int N = 10000;
    String[] srcArray = new String[N];
    String[] forArray = new String[srcArray.length];
    String[] arrayCopyArray  = new String[srcArray.length];

    //初始化数组
    for(int index  = 0 ; index  < srcArray.length ; index ++){
        srcArray[index] = String.valueOf(index);
    }

    long forStartTime = System.nanoTime();
    for(int index  = 0 ; index  < srcArray.length ; index ++){
        forArray[index] = srcArray[index];
    }
    long forEndTime = System.nanoTime();
    System.out.println("for方式复制数组："  + (forEndTime - forStartTime) + "纳秒");

    long arrayCopyStartTime = System.nanoTime();
    System.arraycopy(srcArray,0,arrayCopyArray,0,srcArray.length);
    long arrayCopyEndTime = System.nanoTime();
    System.out.println("System.arraycopy复制数组："  + (arrayCopyEndTime - arrayCopyStartTime) + "纳秒");
}
```

## 非线程安全

```java
import java.util.Arrays;

/**
 * @Author mubi
 * @Date 2018/11/24 6:00 PM
 */
public class ArrayCopyThreadSafe {
    private static int[] arrayOriginal = new int[1024 * 1024 * 10];
    private static int[] arraySrc = new int[1024 * 1024 * 10];
    private static int[] arrayDist = new int[1024 * 1024 * 10];

    private void modify() {
        for (int i = 0; i < arraySrc.length; i++) {
            arraySrc[i] = i + 1;
        }
    }

    private synchronized void modify2() {
        for (int i = 0; i < arraySrc.length; i++) {
            arraySrc[i] = i + 1;
        }
    }

    private void copy() {
        System.arraycopy(arraySrc, 0, arrayDist, 0, arraySrc.length);
    }

    private synchronized void copy2() {
        System.arraycopy(arraySrc, 0, arrayDist, 0, arraySrc.length);
    }

    private void copy3() {
        synchronized (this) {
            System.arraycopy(arraySrc, 0, arrayDist, 0, arraySrc.length);
        }
    }

    private synchronized void init() {
        for (int i = 0; i < arraySrc.length; i++) {
            arrayOriginal[i] = i + 1;
            arraySrc[i] = i;
            arrayDist[i] = 0;
        }
    }

    private static void doThreadSafeCheck() throws Exception {
        ArrayCopyThreadSafe arrayCopyThreadSafe = new ArrayCopyThreadSafe();
        for (int i = 0; i < 100; i++) {
            System.out.println("run count: " + (i + 1));
            arrayCopyThreadSafe.init();

            Thread threadModify = new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.print("modify");
                    arrayCopyThreadSafe.modify2();
                }
            });

            Thread threadCopy = new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.print("copy");
                    arrayCopyThreadSafe.copy();
                }
            });


            threadModify.start();
            Thread.sleep(2);
            threadCopy.start();

            threadModify.join();
            threadCopy.join();

            if (!Arrays.equals(arrayOriginal, arrayDist)) {
                throw new RuntimeException("System.arraycopy is not thread safe");
            }
        }
    }

    public static void main(String[] args) throws Exception {
        doThreadSafeCheck();
    }
}
```

# Array.copyOf

// TODO