---
title: "适配器模式"
layout: page
date: 2020-06-24 00:00
---

[TOC]

# 适配器模式

`client`不能直接访问`Adaptee`, `Adapter`是适配器，它将`Adaptee`转换成`Client`能访问的间接接口，这样`client`能间接访问`Adaptee`

继承或依赖

## 例子

* 问题：目前有一个mp3播放器，但是文件有mp3,mp4,vlc格式的，需要能把mp4,vlc格式的也能播放出来
* 解决：mp3播放器上装一个适配器 mp4,vl能变成mp3格式的

* 最后效果效果：原来mp4,vlc不能使用mp3播放器，通过适配器间接的能使用mp3播放器；即有如下公式

```java
mp3播放器 + 适配器 = mp4播放器
mp3播放器 + 适配器 = vlc播放器
...
```

### 原有设计

* 只能播放Mp3的播放器

```java
public interface MediaPlayer {

    void play(String audioType, String fileName);

}
```

```java
/**
 * @Author mubi
 * @Date 2020/6/24 22:28
 *
 * 只能播放mp3
 */
public class Mp3Player implements MediaPlayer {

    @Override
    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("mp3")) {
            System.out.println("mp3 playing " + fileName);
        } else {
            System.out.println("Mp3Player can not play audioType:" + audioType + " file:" + fileName);
        }
    }
}
```

* 测试

```java
 static void test1(){
    MediaPlayer player = new Mp3Player();
    player.play("mp3", "/audio/a.mp3");
    player.play("mp4", "/audio/b.mp4");
    player.play("vlc", "/audio/c.vlc");
}
```

输出如下

```java
mp3 playing /audio/a.mp3
Mp3Player can not play audioType:mp4 file:/audio/b.mp4
Mp3Player can not play audioType:vlc file:/audio/c.vlc
```

### 组合方式实现适配

```java
public interface AdvanceMediaPlayer {

    void play(String audioType, String fileName);

}
```

* MediaToAdvanceAdapter

```java
public class MediaToAdvanceAdapter implements AdvanceMediaPlayer{

    // 组合方式
    MediaPlayer mediaPlayer;

    public MediaToAdvanceAdapter(MediaPlayer mediaPlayer) {
        this.mediaPlayer = mediaPlayer;
    }

    @Override
    public void play(String audioType, String fileName) {
        if(!audioType.equalsIgnoreCase("mp3")){
            System.out.print(String.format("audioType from %s to mp3...", audioType));
        }
        mediaPlayer.play("mp3", fileName);
    }
}
```

* 测试

```java
static void test2(){
    MediaPlayer player = new Mp3Player();
    AdvanceMediaPlayer advanceMediaPlayer = new MediaToAdvanceAdapter(player);
    advanceMediaPlayer.play("mp3", "/audio/a.mp3");
    advanceMediaPlayer.play("mp4", "/audio/b.mp4");
    advanceMediaPlayer.play("vlc", "/audio/c.vlc");
}
```

输出如下

```java
mp3 playing /audio/a.mp3
audioType from mp4 to mp3...mp3 playing /audio/b.mp4
audioType from vlc to mp3...mp3 playing /audio/c.vlc
```
