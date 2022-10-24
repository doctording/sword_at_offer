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

* 问题：目前有一个mp3播放器，但是文件有mp3,mp4,vlc格式的，mp3格式的能正常播放，但是需要能把mp4,vlc格式的也能播放出来
* 解决：mp3播放器上装一个适配器，能把mp4,vl能变成mp3格式的

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

### 组合方式实现适配：文件格式转化

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

## 扩展成高级接口并适配：播放器适配成全新播放器

* 高级播放器与其实现

```java
public interface AdvancedMediaPlayer extends MediaPlayer{
    void playVlc(String fileName);
    void playMp4(String fileName);
}
```

```java
public class Mp4Player implements AdvancedMediaPlayer{

    @Override
    public void playVlc(String fileName) {
        //什么也不做
    }

    @Override
    public void playMp4(String fileName) {
        System.out.println("Playing mp4 file. Name: "+ fileName);
    }

    @Override
    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("mp4")) {
            playMp4(fileName);
        } else {
            System.out.println("Mp3Player can not play audioType:" + audioType + " file:" + fileName);
        }
    }
}
```

```java
public class VlcPlayer implements AdvancedMediaPlayer{
    @Override
    public void playVlc(String fileName) {
        System.out.println("Playing vlc file. Name: "+ fileName);
    }

    @Override
    public void playMp4(String fileName) {
        //什么也不做
    }

    @Override
    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("vlc")) {
           playVlc(fileName);
        } else {
            System.out.println("Mp3Player can not play audioType:" + audioType + " file:" + fileName);
        }
    }
}
```

* MediaAdapter 适配了 MediaPlayer

```java
public class MediaAdapter implements MediaPlayer {

    MediaPlayer mediaPlayer;

    public MediaAdapter(String audioType){
        if (audioType.equalsIgnoreCase("mp3")) {
            mediaPlayer = new Mp3Player();
        } else if(audioType.equalsIgnoreCase("vlc") ){
            mediaPlayer = new VlcPlayer();
        } else if (audioType.equalsIgnoreCase("mp4")){
            mediaPlayer = new Mp4Player();
        }
    }

    @Override
    public void play(String audioType, String fileName) {
        mediaPlayer.play(audioType, fileName);
    }
}
```

全新的播放器与测试

```java
public class AudioPlayer implements MediaPlayer{
    MediaAdapter mediaAdapter;

    @Override
    public void play(String audioType, String fileName) {
        mediaAdapter = new MediaAdapter(audioType);
        mediaAdapter.play(audioType, fileName);
    }
}
```

* 测试

```java
static void test2(){
    AudioPlayer player = new AudioPlayer();
    player.play("mp3", "/audio/a.mp3");
    player.play("mp4", "/audio/b.mp4");
    player.play("vlc", "/audio/c.vlc");
}
```
