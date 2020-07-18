---
title: "Facade模式"
layout: page
date: 2019-03-20 00:00
---

[TOC]

# `Facade`模式

n.(建筑物的) 正面，立面; (虚假的) 表面，外表; `美[fəˈsɑːd]`

门面模式；外观模式

* 外部与一个子系统的通信必须通过一个统一的门面(Facade)对象进行，这就是门面模式。

eg1: 如果把医院作为一个子系统，按照部门职能，这个系统可以划分为挂号、门诊、划价、化验、收费、取药等。看病的病人要与这些部门打交道，就如同一个子系统的客户端与一个子系统的各个类打交道一样，不是一件容易的事情。---> 接待员

eg2: 造个房子：找水泥工，找瓦匠，木匠，装修等各样的活，也是要与这些角色意义打交道。---> 包工头

## 门面类

综合`电话机`和`相机`功能的门面类

```java
public class FacadeCameraPhone {
    private Phone mPhone;
    private Camera mCamera;

    public FacadeCameraPhone() {
        mPhone = new PhoneImpl();
        mCamera = new CameraImpl();
    }

    public void deil(){
        mPhone.dail();
    }

    public void close(){
        mPhone.hangup();
    }

    public void takePicture(){
        mCamera.takePicture();
    }
}
```

* Camera

```java
public interface Camera {
    //拍照片
    void takePicture();

```

* Phone

```java
public interface Phone {
    //打电话
    void dail();
    //挂电话
    void hangup();
}
```
