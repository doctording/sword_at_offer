---
title: "Comparison method violates its general contract!"
layout: page
date: 2019-03-23 00:00
---

[TOC]

# 报错：Comparison method violates its general contract!

```java
package com.mb;

import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.Random;
import java.util.stream.Collectors;


class Size implements Comparable{
    private Integer size;

    public Size(Integer size) {
        this.size = size;
    }

    @Override
    public int compareTo(Object o) {

        if(this == o){
            return 0;
        }else if (o != null && o instanceof Size) {
            Size s2 = (Size) o;
            if(this.size <= s2.size){
                return -1;
            }
            return 1;
        }else{
            return -1;
        }
    }

    @Override
    public String toString() {
        return String.format("%d",size);
    }

    public Integer getSize() {
        return size;
    }
}

public class Main {

    public static void main(String[] args) throws Exception {
        List<Size> sizeList = new ArrayList<>(16);
        int n = 100;
        for(int i=0;i<n;i++){
            sizeList.add(new Size(new Random().nextInt(3)));
        }

        System.out.println(sizeList);
        List<Size> sizeListSorted = sizeList
                .stream()
                .sorted(Comparator.comparing(Size::getSize))
                .collect(Collectors.toList());
        System.out.println(sizeListSorted);

        sizeList.sort((c1, c2)->{
            int size1 = c1.getSize();
            int size2 = c2.getSize();
            if(size1 <= size2){
                return -1;
            }
            return 1;
        });
        System.out.println(sizeList);
    }

}
```

* output

```js
/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/bin/java "-javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=52278:/Applications/IntelliJ IDEA.app/Contents/bin" -Dfile.encoding=UTF-8 -classpath /Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/charsets.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/deploy.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/cldrdata.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/dnsns.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/jaccess.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/jfxrt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/localedata.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/nashorn.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/sunec.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/sunjce_provider.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/sunpkcs11.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/zipfs.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/javaws.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/jce.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/jfr.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/jfxswt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/jsse.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/management-agent.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/plugin.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/resources.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/rt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/lib/ant-javafx.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/lib/dt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/lib/javafx-mx.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/lib/jconsole.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/lib/packager.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/lib/sa-jdi.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/lib/tools.jar:/Users/mubi/IdeaProjects/untitled/out/production/untitled:/Users/mubi/IdeaProjects/untitled/mysql-connector-java-5.1.39.jar com.mb.Main
[2, 2, 2, 1, 0, 1, 0, 1, 1, 2, 1, 1, 1, 1, 2, 1, 2, 0, 2, 1, 1, 1, 1, 0, 2, 1, 1, 0, 2, 1, 1, 2, 2, 2, 1, 0, 1, 1, 1, 2, 1, 2, 0, 0, 1, 2, 2, 1, 2, 0, 2, 0, 0, 0, 2, 1, 0, 0, 0, 1, 0, 0, 2, 2, 1, 1, 2, 2, 0, 2, 2, 1, 2, 0, 1, 2, 1, 1, 0, 1, 0, 0, 1, 2, 2, 1, 0, 0, 1, 1, 1, 0, 0, 1, 0, 2, 1, 2, 0, 1]
Exception in thread "main" java.lang.IllegalArgumentException: Comparison method violates its general contract!
[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2]
	at java.util.TimSort.mergeHi(TimSort.java:899)
	at java.util.TimSort.mergeAt(TimSort.java:516)
	at java.util.TimSort.mergeCollapse(TimSort.java:441)
	at java.util.TimSort.sort(TimSort.java:245)
	at java.util.Arrays.sort(Arrays.java:1512)
	at java.util.ArrayList.sort(ArrayList.java:1462)
	at com.mb.Main.main(Main.java:59)

Process finished with exit code 1

```

修改如下： 区分**等于**

```java
sizeList.sort((c1, c2)->{
        int size1 = c1.getSize();
        int size2 = c2.getSize();
        if(size1 < size2){
            return -1;
        }else if(size1 == size2){
            return 0;
        }
        return 1;
    });
```

当x == y时，sgn(compare(x, y)) = -1，-sgn(compare(y, x)) = 1，这违背了sgn(compare(x, y)) == -sgn(compare(y, x))约束

或者虚拟机参数设置: `-Djava.util.Arrays.useLegacyMergeSort=true`
