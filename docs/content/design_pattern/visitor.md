---
title: "访问者模式"
layout: page
date: 2019-03-20 00:00
---

[TOC]

# 访问者(Visitor)模式

* 如果访问操作在类里面，想要另外一种访问方式，不得不修改类；
* 将访问操作独立出来，想要另外一种访问方式直接新增一个类，设计新的访问方式

## 例子程序

* interface Element

```java
package design.visit;

public interface Element {
    /**
     * 抽象接口，接受一个Visitor来访问
     * @param visitor
     */
    void accept(Visitor visitor);
}
```

* class Component

```java
package design.visit;

public abstract class Component implements Element {

    public void add(Component component){

    }

    public void remove(Component component){

    }
}
```

* class Item

```java
package design.visit;

/**
 * 具体数据类
 */
public class Item extends Component {

    private String name;
    private float price;

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }

    @Override
    public String toString() {
        return "Item{" +
                "name='" + name + '\'' +
                ", price=" + price +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public float getPrice() {
        return price;
    }

    public void setPrice(float price) {
        this.price = price;
    }
}
```

* abstract class Visitor

```java
package design.visit;

public abstract class Visitor {
    /**
     * 访问Element的抽象接口
     * @param e
     */
    public abstract void visit(Element e);
}
```

* class ItemVisitor

```java
package design.visit;

public class ItemVisitor extends Visitor {

    @Override
    public void visit(Element e) {
        if(e instanceof Item){
            Item item = (Item)e;
            System.out.println(item);
        }
    }
}
```

* class ItemVisitor2

```java
package design.visit;

public class ItemVisitor2 extends Visitor {

    @Override
    public void visit(Element e) {
        if(e instanceof Item){
            Item item = (Item)e;
            System.out.println("name:" + item.getName());
        }
    }
}
```

* 测试

```java
public class DesignTest {

    public static void main(String[] args) {
        Item item = new Item();
        item.setName("a");
        item.setPrice(1.1f);

        new ItemVisitor().visit(item);
        new ItemVisitor2().visit(item);
    }
}
/*
Item{name='a', price=1.1}
name:a
*/
```
