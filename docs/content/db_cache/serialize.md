---
title: "序列化问题"
layout: page
date: 2020-09-15 18:00
---

[TOC]

# 序列化

## 二叉树序列化(leetcode)

* ac代码

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Codec {

    // Encodes a tree to a single string.
    public String serialize(TreeNode root) {
        if(root == null){
            return "[]";
        }
        StringBuilder sb = new StringBuilder();
        sb.append("[");
        Queue<TreeNode> que = new LinkedList<>();
        que.offer(root);
        while(!que.isEmpty()){
            TreeNode node = que.poll();
            if(node != null) {
                String tmpVal = String.format("%d,", node.val);
                sb.append(tmpVal);
                que.offer(node.left);
                que.offer(node.right);
            }else{
                sb.append("null,");
            }
        }
        String rs = sb.toString();
        // 去除尾部的null
        int len = rs.length();
        int i = len - 1;
        while (!(rs.charAt(i) >= '0' && rs.charAt(i) <= '9')){
            i --;
        }
        rs = rs.substring(0, i+1);
        rs += "]";
        return rs;
    }

    // Decodes your encoded data to tree.
    public TreeNode deserialize(String data) {
        if(data.equals("[]")){
            return null;
        }
        // 转化为node数组
        String rs = data.substring(1, data.length() - 1);
        String[] arr = rs.split(",");
        TreeNode[] nodeArr = new TreeNode[arr.length];
        for (int i = 0; i < arr.length; i++) {
            if (arr[i].equalsIgnoreCase("null")) {
                nodeArr[i] = null;
            } else {
                int val = Integer.valueOf(arr[i]);
                nodeArr[i] = new TreeNode(val);
            }
        }
        int index = 0;
        int p = 1;
        while (p < nodeArr.length) {
            if (nodeArr[index] == null) {
                index++;
            } else {
                nodeArr[index].left = nodeArr[p];
                if(p + 1 < arr.length){
                    nodeArr[index].right = nodeArr[p+1];
                }
                p += 2;
                index ++;
            }
        }
        return nodeArr[0];
    }
}

// Your Codec object will be instantiated and called as such:
// Codec codec = new Codec();
// codec.deserialize(codec.serialize(root));
```

* 测试代码

```java
public static void main(String[] args) throws Exception{
    TreeNode node1 = new TreeNode(1);
    TreeNode node2 = new TreeNode(2);
    TreeNode node3 = new TreeNode(3);
    TreeNode node4 = new TreeNode(4);
    TreeNode node5 = new TreeNode(5);
    node1.left = node2;
    node1.right = node3;

    node3.left = node4;
    node3.right = node5;

    Codec codec = new Codec();
    codec.deserialize(codec.serialize(node1));
}
```

## 序列化问题

## 什么是序列化？

* 序列化：序列化是将对象转化为字节流
* 反序列化：反序列化是将字节流转化为对象

序列化和反序列化的本质是用特定的二进制协议，描述和解释对象。可以跨语言、跨介质传输保存

## 序列化用途？

* 序列化可以将对象的字节序列持久化-保存在内存、文件、数据库中。
* 在网络上传送对象的字节序列
* RMI(远程方法调用RPC)
* 实现对象的深拷贝

## Java序列化机制原理是什么？

Java序列化就是将一个对象转化为一个二进制表示的字节数组，通过保存或则转移这些二进制数组达到持久化的目的。要实现序列化，需要实现java.io.Serializable接口。反序列化是和序列化相反的过程，就是把二进制数组转化为对象的过程。在反序列化的时候，必须有原始类的模板才能将对象还原。

## transient 关键字

当某个字段被声明为transient后，默认序列化机制就会忽略该字段。此处将Person类中的age字段声明为transient，如下所示，

```java
public class Person implements Serializable {
    transient private Integer age = null;
}
```

再执行SimpleSerial应用程序，会有如下输出：

```java
arg constructor
[John, null, MALE]
```

可见，age字段未被序列化。

附：测试程序

```java
public class SimpleSerial {

    public static void main(String[] args) throws Exception {
        File file = new File("person.out");

        ObjectOutputStream oout = new ObjectOutputStream(new FileOutputStream(file));
        Person person = new Person("John", 101, Gender.MALE);
        oout.writeObject(person);
        oout.close();

        ObjectInputStream oin = new ObjectInputStream(new FileInputStream(file));
        Object newPerson = oin.readObject(); // 没有强制转换到Person类型
        oin.close();
        System.out.println(newPerson);
    }
}
```

附:Person定义

```java
public class Person implements Serializable {

    private String name = null;

    private Integer age = null;

    private Gender gender = null;

    public Person() {
        System.out.println("none-arg constructor");
    }

    public Person(String name, Integer age, Gender gender) {
        System.out.println("arg constructor");
        this.name = name;
        this.age = age;
        this.gender = gender;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Gender getGender() {
        return gender;
    }

    public void setGender(Gender gender) {
        this.gender = gender;
    }

    @Override
    public String toString() {
        return "[" + name + ", " + age + ", " + gender + "]";
    }
}
```

### 序列化时,你希望某些成员不要序列化?你如何实现它？

如果你不希望任何字段是对象的状态的一部分, 然后声明它静态或瞬态(使用 transient )根据你的需要, 这样就不会是在 Java 序列化过程中被包含在内

* transient: adj. 短暂的; 转瞬即逝的; 倏忽; 暂住的; 过往的; 临时的; n. 暂住某地的人; 过往旅客; 临时工;

### 在 Java 序列化期间,哪些变量未序列化？

这个问题问得不同, 但目的还是一样的, Java开发人员是否知道静态和瞬态变量的细节。由于静态变量属于类, 而不是对象, 因此它们不是对象状态的一部分, 因此在 Java 序列化过程中不会保存它们。由于 Java 序列化仅保留对象的状态,而不是对象本身。瞬态变量也不包含在 Java 序列化过程中, 并且不是对象的序列化状态的一部分。在提出这个问题之后,面试官会询问后续内容, 如果你不存储这些变量的值, 那么一旦对这些对象进行反序列化并重新创建这些变量, 这些变量的价值是多少？这是要考虑的。

## Serializable的作用

writeObject中的部分源码如下

```java
private void writeObject0(Object obj, boolean unshared) throws IOException {

    if (obj instanceof String) {
        writeString((String) obj, unshared);
    } else if (cl.isArray()) {
        writeArray(obj, desc, unshared);
    } else if (obj instanceof Enum) {
        writeEnum((Enum) obj, desc, unshared);
    } else if (obj instanceof Serializable) {
        writeOrdinaryObject(obj, desc, unshared);
    } else {
        if (extendedDebugInfo) {
            throw new NotSerializableException(cl.getName() + "\n"
                    + debugInfoStack.toString());
        } else {
            throw new NotSerializableException(cl.getName());
        }
    }

}
```

从上述代码可知，如果被写对象的类型是String，或数组，或Enum，或Serializable，那么就可以对该对象进行序列化，否则将抛出NotSerializableException。

### 在 Java 中的序列化和反序列化过程中使用哪些方法？

Java 序列化由java.io.ObjectOutputStream类完成。该类是一个筛选器流, 它封装在较低级别的字节流中, 以处理序列化机制。要通过序列化机制存储任何对象, 我们调用 ObjectOutputStream.writeObject(savethisobject), 并反序列化该对象, 我们称之为 ObjectInputStream.readObject()方法。调用以 writeObject() 方法在 java 中触发序列化过程。关于 readObject() 方法, 需要注意的一点很重要一点是, 它用于从持久性读取字节, 并从这些字节创建对象, 并返回一个对象, 该对象需要类型强制转换为正确的类型。

## serialVersionUID

serialVersionUID 是 Java 为每个序列化类产生的版本标识。它可以用来保证在反序列时，发送方发送的和接受方接收的是可兼容的对象。如果接收方接收的类的 serialVersionUID 与发送方发送的 serialVersionUID 不一致，会抛出 InvalidClassException。

如果可序列化类没有显式声明 serialVersionUID，则序列化运行时将基于该类的各个方面计算该类的默认 serialVersionUID 值。尽管这样，还是建议在每一个序列化的类中显式指定 serialVersionUID 的值。因为不同的 jdk 编译很可能会生成不同的 serialVersionUID 默认值，从而导致在反序列化时抛出 `java.io.InvalidClassException`, 异常。

serialVersionUID 字段必须是 static final long 类型。
