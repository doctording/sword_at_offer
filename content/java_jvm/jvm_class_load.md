---
title: "Java 类加载"
layout: page
date: 2019-02-15 00:00
---

[TOC]

# Class 文件

Class文件是一组以8位字节为基础单位的`二进制流`，任何一个Class文件都对应唯一一个类或接口的定义信息

## 魔数与Class文件的版本

## 常量池

主要存放两大类常量： 字面量（Literal 和 符号引用（Symbolic Reference）

1. 字面量：如文本字符串，声明为final的常量值

2. 符号引用： 类和接口的全限定名，字段的名称和描述符，方法的名称和描述符

Class文件中不会保存各个方法，字段的最终内存布局信息，因此这些字段，方法的符号引用不经过运行期转换的话，无法得到真正的内存入口地址，也就无法直接被虚拟机使用。当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建时或运行时解析，翻译到具体的内存地址之中。

## 访问标志

用于识别一些类或者接口层次的访问信息，包括：这个Class是类还是接口；是否定义为public类型；是否定义为abstrac类型；如果是类的话，是否被声明为final等。

## 类索引，父类索引，接口索引

## 字段表(field_info)

用来描述接口或者类中声明的变量， 可以包括的信息有：

字段的作用域(public,private,protected修饰符)，是实例变量还是类变量（static修饰符），可变性（final）,并发可见性（volatile修饰符,是否强制从主内存读写），可否被序列化（transient修饰符），字段数据类型（基本类型，对象，数组），字段名称。

## 方法表

方法签名：方法名称，参数顺序，参数类型

## 属性表

上述Class文件，字段表，方法表都可以携带自己的属性表集合

# 程序编译和代码优化

* 词法，语法分析

词法分析将源代码的字符流转变为标记(Token)集合

语法分析根据Token序列构造抽象语法树（Abstract Syntax Tree）

* Annotation，编译期间读取，修改，添加抽象语法树中的任意符号

* 语义分析与字节码生成

## 类加载

```js
连接( 验证 =》 准备 =》 解析)
    => 初始化
        => 使用
            => 卸载
```

### 验证

验证是链接阶段的第一步，这一步主要的目的是确保class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身安全。 验证阶段主要包括四个检验过程：文件格式验证、元数据验证、字节码验证和符号引用验证。

### 准备

准备阶段是正式为类变量分配内存并设置类变量的初始值阶段，即在方法区中分配这些变量所使用的内存空间。

```java
public static int value  = 12;
```

变量value在准备阶段过后的初始值为0而不是12，因为这时候尚未开始执行任何java方法，而把value赋值为123的putstatic指令是程序被编译后，存放于类构造器`<clinit>()`方法之中，所以把value赋值为12的动作将在初始化阶段才会被执行。

相对于一些特殊的情况，如果类字段的字段属性表中存在ConstantValue属性，那在准备阶段变量value就会被初始化为ConstantValue属性所指定的值，建设上面类变量value定义为：

```java
public static final int value = 123;
```

编译时javac将会为value生成ConstantValue属性，在准备阶段虚拟机就会根据ConstantValue的设置将value设置为123。

### 解析

解析阶段是虚拟机常量池内的符号引用替换为直接引用的过程。

直接引用可以是指向目标的指针，相对偏移量或是一个能间接定位到目标的句柄。如果有了直接引用，那引用的目标必定已经在内存中存在。

### 初始化

初始化阶段是类加载最后一个阶段，前面的类加载阶段之后，除了在加载阶段可以自定义类加载器以外，其它操作都由JVM主导。到了初始阶段，才开始真正执行类中定义的Java程序代码。

初始化阶段是执行类构造器`<client>`方法的过程。

`<client>`方法是由编译器自动收集类中的**类变量的赋值操作和静态语句块**中的语句合并而成的。虚拟机会保证`<client>`方法执行之前，父类的`<client>`方法已经执行完毕。

#### 什么时候需要对类进行初始化？

1. 使用`new`该类实例化对象的时候
2. 读取或设置`类静态字段`的时候（但被final修饰的字段，在编译器时就被放入常量池(static final)的静态字段除外）
3. 调用`类静态方法`的时候
4. 使用反射`Class.forName("xxx")`对类进行反射调用的时候，该类需要初始化；
5. 初始化一个类的时候，有父类，`先初始化父类`（注：1. 接口除外，父接口在调用的时候才会被初始化；2.子类引用父类静态字段，只会引发父类初始化）；
6. 被标明为启动类的类（即包含`main()方法`的类）要初始化；
7. 当使用JDK1.7的动态语言支持时，如果一个`java.invoke.MethodHandle`实例最后的解析结果REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化。

以上情况称为对一个类进行主动引用，且有且只要以上几种情况需要对类进行初始化：

* 所有类变量初始化语句和静态代码块都会在编译时被前端编译器放在收集器里头，存放到一个特殊的方法中，这个方法就是`<clinit>`方法，即类/接口初始化方法，该方法只能在类加载的过程中由JVM调用；

* 编译器收集的顺序是由语句在源文件中出现的顺序所决定的，静态语句块中只能访问到定义在静态语句块之前的变量；

* 如果超类还没有被初始化，那么优先对超类初始化，但在`<clinit>`方法内部不会显示调用超类的`<clinit>`方法，由JVM负责保证一个类的`<clinit>`方法执行之前，它的超类`<clinit>`方法已经被执行。

* JVM必须确保一个类在初始化的过程中，如果是多线程需要同时初始化它，仅仅只能允许其中一个线程对其执行初始化操作，其余线程必须等待，只有在活动线程执行完对类的初始化操作之后，才会通知正在等待的其他线程。(所以可以利用静态内部类实现线程安全的单例模式)

* 如果一个类没有声明任何的类变量，也没有静态代码块，那么可以没有类`<clinit>`方法；

### 类加载器

加载动作放到JVM外部实现，以便让应用程序决定如何获取所需的类, 类加载器大致可以分为以下3部分：

#### 启动类加载器 Bootstrap ClassLoader

将存放于`<JAVA_HOME>\lib`目录中的，或者被`-Xbootclasspath`参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如 rt.jar 名字不符合的类库即使放在lib目录中也不会被加载）类库加载到虚拟机内存中。启动类加载器无法被Java程序直接引用。

#### 扩展类加载器 Extension ClassLoader

将`<JAVA_HOME>\lib\ext`目录下的，或者被java.ext.dirs系统变量所指定的路径中的所有类库加载。开发者可以直接使用扩展类加载器。

#### 应用程序类加载器 Application ClassLoader

负责加载用户类路径(ClassPath)上所指定的类库,开发者可直接使用。

#### * 双亲委派模型 *

工作过程为：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的加载器都是如此，因此所有的类加载请求都会传给顶层的启动类加载器，只有当父加载器反馈自己无法完成该加载请求（该加载器的搜索范围中没有找到对应的类）时，子加载器才会尝试自己去加载

```java
protected synchronized Class loadClass(String name, boolean resolve)
        throws ClassNotFoundException {
    // 首先检查该name指定的class是否有被加载
    Class c = findLoadedClass(name);
    if (c == null) {
        try {
            if (parent != null) {
                // 如果parent不为null，则调用parent的loadClass进行加载
                c = parent.loadClass(name, false);
            } else {
                // parent为null，则调用BootstrapClassLoader进行加载
                c = findBootstrapClass0(name);
            }
        } catch (ClassNotFoundException e) {
            // 如果仍然无法加载成功，则调用自身的findClass进行加载
            c = findClass(name);
        }
    }
    if (resolve) {
        resolveClass(c);
    }
    return c;
}
```

#### 类加载题目例子（Java7环境）

```java

class Singleton{
    private static Singleton singleton = new Singleton();
    static {
        System.out.println("static block");
    }
    public static int value1;
    public static int value2 = 0;
    public static int value3 = 10;

    private Singleton(){
        System.out.println(String.format("before Singleton constructor: value1:%d, value2:%d, value3:%d",
                value1,value2,value3));
        value1++;
        value2++;
        value3++;
        System.out.println(String.format("after Singleton constructor: value1:%d, value2:%d, value3:%d",
                value1,value2,value3));
    }

    public static Singleton getInstance(){
        return singleton;
    }

}

class Singleton2{
    public static int v1;
    public static int v2 = 0;
    public static int v3 = 10;
    static {
        System.out.println("static block");
    }
    private static Singleton2 singleton2 = new Singleton2();

    private Singleton2(){
        System.out.println(String.format("before Singleton2 constructor: v1:%d, v2:%d, v3:%d",
                v1,v2,v3));
        v1++;
        v2++;
        v3++;
        System.out.println(String.format("before Singleton2 constructor: v1:%d, v2:%d, v3:%d",
                v1,v2,v3));
    }

    public static Singleton2 getInstance2(){
        return singleton2;
    }

}

public class MainTest {
    public static final int _1MB = 1024 * 1024;

    public static void main(String[] args) throws Exception{
        // idea 提示 不应该通过类的实例访问静态成员，不过程序本身无任何问题，结果如下：
        Singleton singleton = Singleton.getInstance();
        // 1
        System.out.println("Singleton1 value1:" + singleton.value1);
        // 0
        System.out.println("Singleton1 value2:" + singleton.value2);
        // 10
        System.out.println("Singleton1 value3:" + singleton.value3);

        Singleton2 singleton2 = Singleton2.getInstance2();
        // 1
        System.out.println("Singleton2 v1:" + singleton2.v1);
        // 1
        System.out.println("Singleton2 v2:" + singleton2.v2);
        // 11
        System.out.println("Singleton2 v3:" + singleton2.v3);
    }

}
```

##### 结果分析

```js
before Singleton constructor: value1:0, value2:0, value3:0
after Singleton constructor: value1:1, value2:1, value3:1
static block
Singleton1 value1:1
Singleton1 value2:0
Singleton1 value3:10
static block
before Singleton2 constructor: v1:0, v2:0, v3:10
before Singleton2 constructor: v1:1, v2:1, v3:11
Singleton2 v1:1
Singleton2 v2:1
Singleton2 v3:11
```

* 《Java编程思想》第5章：调用构造函数是编译器的责任，必须要让编译器知道调用的是哪个方法；分配内存空间后，就会调用构造函数，确保使用对象前，对象已经被初始化了。

对象创建的过程：（**类初始化 和 类实例化**）

1. 首次创建对象时，类中的静态方法/静态字段首次被访问时，java解释器必须先查找类路径，以定位.class文件
2. 然后载入`.class`(这将创建一个class对象)，有关静态初始化的所有动作都会执行，按顺序执行。因此，**静态初始化只在Class对象首次加载的时候进行一次**
3. 当用`new XX()`创建对象时，首先在**堆上为对象分配足够的存储空间**
4. 这块存储空间会被清零，这就自动地将对象中的所有基本类型数据都设置成了缺省值（对数字来说就是0，对布尔型和字符型也相同），而引用则被设置成了null。
5. 执行所有出现于字段定义处的初始化动作（**非静态对象的初始化**）
6. 执行构造器。

###### Singleton 结果分析

1. 首先执行main中的`Singleton singleton = Singleton.getInstance()`,

2. 访问了静态方法`访问静态方法`, 开始加载类`Singleton`

3. 随后，类的验证，准备

这里会将为singleton(引用类型)设置为null,value1,value2,value3（基本数据类型）设置默认值0 ）

4. 类的初始化（按照赋值语句进行修改）

```java
private static Singleton singleton = new Singleton();
public static int value1;
public static int value2 = 0;
public static int value3 = 10;
```

`new Singleton()` 不在进行类加载过程(上面已经类加载过了)，直接对象初始化, 如下

```js
before Singleton constructor: value1:0, value2:0, value3:0
after Singleton constructor: value1:1, value2:1, value3:1
```

然后是如下的初始化语句，（注意如下的静态操作只会进行一次）

```java
static {
    System.out.println("static block");
}
public static int value1; // 没有赋值，取在对象实例的构造函数执行后的值
public static int value2 = 0; // 类初次赋值
public static int value3 = 10;  // 类初次赋值
```

所以最后main函数中打印

```js
Singleton1 value1:1
Singleton1 value2:0
Singleton1 value3:10
```

###### Singleton2 分析

与`Singleton`的初始化语句顺序不一样,`new`对象，是在类初始变量之后

```java
public static int v1;
public static int v2 = 0;
public static int v3 = 10;
static {
    System.out.println("static block");
}
private static Singleton2 singleton2 = new Singleton2();
```

* output，注意到构造函数执行之前是：v1:0, v2:0, v3:10

```java
static block
before Singleton2 constructor: v1:0, v2:0, v3:10
before Singleton2 constructor: v1:1, v2:1, v3:11
Singleton2 v1:1
Singleton2 v2:1
Singleton2 v3:11
```

###### 静态初始化再次验证

类中静态初始化语句，只会在类加载的时候执行一次

* code1

```java
Singleton2 singleton2 = Singleton2.getInstance2();
System.out.println("Singleton2 v1:" + singleton2.v1);
System.out.println("Singleton2 v2:" + singleton2.v2);
System.out.println("Singleton2 v3:" + singleton2.v3);

Singleton2 singleton22 = Singleton2.getInstance2();
System.out.println("Singleton2 v1:" + singleton22.v1);
System.out.println("Singleton2 v2:" + singleton22.v2);
System.out.println("Singleton2 v3:" + singleton22.v3);
```

* output1

```java
static block
before Singleton2 constructor: v1:0, v2:0, v3:10
before Singleton2 constructor: v1:1, v2:1, v3:11
Singleton2 v1:1
Singleton2 v2:1
Singleton2 v3:11
Singleton2 v1:1
Singleton2 v2:1
Singleton2 v3:11
```

* code2

```java
Singleton singleton = Singleton.getInstance();
System.out.println("Singleton1 value1:" + singleton.value1);
System.out.println("Singleton1 value2:" + singleton.value2);
System.out.println("Singleton1 value3:" + singleton.value3);

Singleton singleton2 = new Singleton();
System.out.println("Singleton1 value1:" + singleton2.value1);
System.out.println("Singleton1 value2:" + singleton2.value2);
System.out.println("Singleton1 value3:" + singleton2.value3);
```

* output2

```java
before Singleton constructor: value1:0, value2:0, value3:0
after Singleton constructor: value1:1, value2:1, value3:1
static block
Singleton1 value1:1
Singleton1 value2:0
Singleton1 value3:10
before Singleton constructor: value1:1, value2:0, value3:10
after Singleton constructor: value1:2, value2:1, value3:11
Singleton1 value1:2
Singleton1 value2:1
Singleton1 value3:11
```

# 自定义类加载器

自定义类加载器的一般步骤

1. 继承`ClassLoader`

2. 重写`loadClass()`方法

3. 重写`findClass()`方法
    * class文件路径判断和获取
    * 将class文件载入内存
    * 对载入内存的字节码数据，调用`defineClass()`方法将字节码转化为类

## MyClassLoader 实践

* `Hello.java`编译成`Hello.class`

```java
package com.other;

public class Hello {

    public void test() {
        System.out.println("Loader Class is:" + getClass().getClassLoader().getClass());
    }
}
```

* MyComOtherClassLoader

```java
package com.mb;


import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.lang.reflect.Method;

/**
 * @Author mubi
 * @Date 2019/3/20 10:54 PM
 */
public class MyComOtherClassLoader extends ClassLoader{
    public String path;
    public String packageName;
    public String className;

    public MyComOtherClassLoader() {
        super(ClassLoader.getSystemClassLoader());
    }

    private String classNameToPath() {
        // 得到类文件的URL
        return path + "/" + packageName.replace('.', '/')
                + "/"
                + className + ".class";
    }

    @Override
    public Class loadClass(String name) throws ClassNotFoundException {
        // 非 com.test package下面的类，都用默认的加载， 否则用自定义的加载方法
        if (!name.contains("com.other")) {
            // 是否已经被加载
            Class loadedClass = findLoadedClass(name);
            if (loadedClass == null) {
                // 用父类去加载该类
                loadedClass = getParent().loadClass(name);
                return loadedClass;
            } else {
                return loadedClass;
            }
        }


        int i = name.lastIndexOf('.');
        packageName = "";
        if (i != -1) {
            packageName = name.substring(0, i);
//            System.out.println("package: " + packageName);
            className = name.substring(i + 1);
//            System.out.println("class name: " + name);
            SecurityManager sm = System.getSecurityManager();
            if (sm != null) {
                sm.checkPackageAccess(packageName);
            }
        }
        //依然调用父类的方法
//        return super.loadClass(name);
        return this.findClass(name);
    }

    @Override
    public Class<?> findClass(String name) throws ClassNotFoundException {
        int i = name.lastIndexOf('.');
        packageName = "";
        if (i != -1) {
            packageName = name.substring(0, i);
//            System.out.println("package: " + packageName);
            className = name.substring(i + 1);
//            System.out.println("class name: " + name);
            SecurityManager sm = System.getSecurityManager();
            if (sm != null) {
                sm.checkPackageAccess(packageName);
            }
        }
        Class<?> clazz = this.findLoadedClass(name); // 父类已加载
        if(null != clazz){
            return clazz;
        }

//        System.out.println("findClass param name: " + name);
        byte [] b = this.getClassBytes();
//        System.out.println("b len:" + b.length);
        clazz=defineClass(null, b, 0, b.length);
        return clazz;
    }
    public byte[] getClassBytes() {
        String classPath = classNameToPath();
//        System.out.println("classPath:" + classPath);
        File file=new File(classPath);
        try(FileInputStream fis = new FileInputStream(file);ByteArrayOutputStream bos=new ByteArrayOutputStream();) {
            byte[] b=new byte[1024*2];
            int n;
            while((n=fis.read(b))!=-1){
                bos.write(b, 0, n);
            }
            return bos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }


    static void testMyClassLoaderHello() throws Exception{
        String path = "/Users/mubi/IdeaProjects/untitled/out/production/untitled";
        MyComOtherClassLoader myClassLoader = new MyComOtherClassLoader();
        myClassLoader.path = path;
        Class clazz = myClassLoader.loadClass("com.other.Hello");
        Object obj = clazz.newInstance();
        System.out.println("===" + obj.getClass());
        Method method = clazz.getDeclaredMethod("test", null);
        Object c = method.invoke(obj, null);
        if(c != null){
            System.out.println("method return: " + c.getClass());
        }else {
            System.out.println("method return:" + c);
        }
    }

    static void testMyClassLoaderString() throws Exception{
        MyComOtherClassLoader myClassLoader = new MyComOtherClassLoader();
        Class clazz = myClassLoader.loadClass("java.lang.String");
        Object obj = clazz.newInstance();
        System.out.println("===" + obj.getClass());
        Method method = clazz.getDeclaredMethod("length", null);
        Object c = method.invoke(obj, null);
        if(c != null){
            System.out.println("method return: " + c.getClass());
        }else {
            System.out.println("method return:" + c);
        }
    }

    public static void main(String[] args) throws Exception {
        testMyClassLoaderHello();
        testMyClassLoaderString();
    }

}
/*
===class com.other.Hello
Loader Class is:class com.thread.MyClassLoader2
method return:null
===class java.lang.String
method return: class java.lang.Integer
*/
```

## 自定义类加载器的优缺点 // TODO

* 加密

* 从非标准的来源加载代码

* 部署在同一个服务器上的两个Web应用程序所使用的Java类库可以实现相互隔离。

* 部署在同一个服务器上的两个Web应用程序所使用的Java类库可以相互共享

* 支持热替换