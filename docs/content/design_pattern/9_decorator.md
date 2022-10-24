---
title: "装饰器模式"
layout: page
date: 2019-03-20 00:00
---

[TOC]

# 装饰器模式

* 意图：动态地给一个对象添加一些额外的职责。就增加功能来说，装饰器模式相比生成子类更为灵活。

* 主要解决：一般的，我们为了扩展一个类经常使用继承方式实现，由于继承为类引入静态特征，并且随着扩展功能的增多，子类会很膨胀。

* 何时使用：在不想增加很多子类的情况下扩展类。

* 如何解决：将具体功能职责划分，同时继承装饰者模式。

eg:

* Shape 可以 draw

```java
public interface Shape {
   void draw();
}
```

* 继承方式，给draw加一个红色边框处理修饰

```java
public class RedShapeDecorator extends ShapeDecorator {

   public RedShapeDecorator(Shape decoratedShape) {
      super(decoratedShape);
   }

   @Override
   public void draw() {
      decoratedShape.draw();
      setRedBorder(decoratedShape);
   }

   private void setRedBorder(Shape decoratedShape){
      System.out.println("Border Color: Red");
   }
}
```

## 继承和组合？

* 继承(`Is A`?)
   1. 在继承中，子类自动继承父类的非私有成员(default类型视是否同包而定)，在需要时，可选择直接使用或重写。
   2. 在继承中，创建子类对象时，无需创建父类对象，因为系统会自动完成；而在组合中，创建组合类的对象时，通常需要创建其所使用的所有类的对象。

* 组合(`Has A`?)
   1. 在组合中，组合类与调用类之间低耦合；而在继承中子类与父类高耦合。
   2. 可动态组合。

合成复用原则（Composite Reuse Principle，CRP）又叫组合/聚合复用原则（Composition/Aggregate Reuse Principle，CARP）。它要求在软件复用时，要尽量先使用组合或者聚合等关联关系来实现，其次才考虑使用继承关系来实现。
