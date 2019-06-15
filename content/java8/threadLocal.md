---
title: "ThreadLocal"
layout: page
date: 2019-02-15 00:00
---

[TOC]

# ThreadLocal

`ThreadLocal`并不是一个`Thread`，而是`Thread`的局部变量

threadLocal用于保存某个线程共享变量：对于同一个static ThreadLocal，不同线程只能从中get，set，remove自己的变量，而不会影响其他线程的变量

## 实现思路

Thread类有一个类型为ThreadLocal.ThreadLocalMap的实例变量threadLocals，也就是说每个线程有一个自己的ThreadLocalMap。ThreadLocalMap有自己的独立实现，可以简单地将它的key视作ThreadLocal，value为代码中放入的值（实际上key并不是ThreadLocal本身，而是它的一个弱引用）。每个线程在往某个ThreadLocal里塞值的时候，都会往自己的ThreadLocalMap里存，读也是以某个ThreadLocal作为引用，在自己的map里找对应的key，从而实现了线程隔离。

## 使用场景

1. 在进行对象跨层传递的时候，可以考虑`ThreadLocal`，避免方法多次传递，打破层次间的约束

2. 线程间数据隔离

3. 进行事务操作，用于存储线程事务信息

## 使用`ThreadLocal`设计线程上下文

* 转自如下

作者：香沙小熊
链接：https://www.jianshu.com/p/72df0c834696
来源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。

* 每个action调用context的时候传入进去

```java
public class ExecutionTask implements Runnable {

    private QueryFromDBAction queryAction = new QueryFromDBAction();

    private QueryFromHttpAction httpAction = new QueryFromHttpAction();

    @Override
    public void run() {

        final Context context = new Context();
        queryAction.execute(context);
        System.out.println("The name query successful");
        httpAction.execute(context);
        System.out.println("The cardId query successful");

        System.out.println("The Name is " + context.getName() + " and CardId " + context.getCardId());
    }
}
```

```java
public class QueryFromDBAction {

    public void execute(Context context) {

        try {
            Thread.sleep(1000L);
            String name = "Jack " + Thread.currentThread().getName();
            context.setName(name);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class QueryFromHttpAction {
    public void execute(Context context) {
        try {
            String name = context.getName();
            String cardId = getCardId(name);
            context.setCardId(cardId);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    private String getCardId(String name) {
        try {
            Thread.sleep(1000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "Tom " + Thread.currentThread().getId();
    }

}
```

* 使用`ThreadLocal`

```java
public final class ActionContext {

    private static final ThreadLocal<Context> threadLocal = new ThreadLocal() {
        @Override
        protected Object initialValue() {
            return new Context();
        }
    };

    public static ActionContext getActionContext() {
        return ContextHolder.actionContext;
    }

    public Context getContext() {

        return threadLocal.get();
    }

    private static class ContextHolder {
        private final static ActionContext actionContext = new ActionContext();

    }
}
```

```java
public class ExecutionTask implements Runnable {

    private QueryFromDBAction queryAction = new QueryFromDBAction();

    private QueryFromHttpAction httpAction = new QueryFromHttpAction();

    @Override
    public void run() {

        final Context context = ActionContext.getActionContext().getContext();
        queryAction.execute();
        System.out.println("The name query successful");
        httpAction.execute();
        System.out.println("The cardId query successful");

        System.out.println("The Name is " + context.getName() + " and CardId " + context.getCardId());
    }
}
```

```java
public class QueryFromDBAction {

    public void execute() {

        try {
            Thread.sleep(1000L);
            String name = "Jack " + Thread.currentThread().getName();
            ActionContext.getActionContext().getContext().setName(name);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

## TODO
