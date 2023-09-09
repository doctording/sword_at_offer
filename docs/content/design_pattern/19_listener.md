---
title: "监听者模式"
layout: page
date: 2019-03-20 00:00
---

[TOC]

# 监听者

监听器模式指的是事件源经过事件的封装传给监听器，当事件源触发事件之后，监听器收到事件的通知并执行事件回调方法。

* 事件

```java
public class Event {
    // 事件类型
    private String type;
    // 事件的数据
    private String data;
    // 事件源
    private EventSource source;

    public Event(String type, String data, EventSource source) {
        this.type = type;
        this.data = data;
        this.source = source;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public String getData() {
        return data;
    }

    public void setData(String data) {
        this.data = data;
    }

    public EventSource getSource() {
        return source;
    }

    public void setSource(EventSource source) {
        this.source = source;
    }
}
```

* 事件源（管理事件监听器，触发事件）

```java
public class EventSource {

    // 监听者对象
    private List<EventListener> listeners = new ArrayList<>();

    // 注册监听者
    public void registerListener(EventListener listener) {
        listeners.add(listener);
    }

    // 移除监听者
    public void removeListener(EventListener listener) {
        listeners.remove(listener);
    }

    // 事件触发，通知每个listener
    public void fireEvent(Event event) {
        for (EventListener listener : listeners) {
            listener.handleEvent(event);
        }
    }
}
```

* 事件监听的抽象接口

```java
public interface EventListener {
    // 处理监听的事件
    void handleEvent(Event event);
}
```

---

测试类:

```java
public class ListenerMainTest {
    public static void main(String[] args) {
        // 事件源, 用来触发事件通知
        EventSource eventSource = new EventSource();
        // 监听者
        EventListener listener1 = new EventListener() {
            @Override
            public void handleEvent(Event event) {
                System.out.println("listener 1：" + event.getData());
            }
        };
        EventListener listener2 = new EventListener() {
            @Override
            public void handleEvent(Event event) {
                System.out.println("listener 2：" + event.getData());
            }
        };

        // 注册监听者
        eventSource.registerListener(listener1);
        eventSource.registerListener(listener2);

        // 事件触发
        eventSource.fireEvent(new Event("greeting", "Hello World!", eventSource));
        // 移除监听者1
        eventSource.removeListener(listener1);
        eventSource.fireEvent(new Event("greeting", "Hello Again!", eventSource));
    }
}
```
