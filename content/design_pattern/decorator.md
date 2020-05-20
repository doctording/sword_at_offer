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
