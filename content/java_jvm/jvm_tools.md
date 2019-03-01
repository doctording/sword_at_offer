---
title: "JDK的命令行工具使用"
layout: page
date: 2019-02-15 00:00
---

[TOC]

# Java自带的各类型工具

名称 | 主要作用
-|-
jps | JVM Process Status Tool, 显示指定系统内所有的HotSpot虚拟机进程
jstat | JVM Statistics Monitoring Tool, 用于收集HotSpot虚拟机各方面的运行数据
jinfo | Configuration Info for Java, 显示虚拟机配置信息
jmap | Memory Map for Java, 生成虚拟机的内存转储快照(heapdump文件)
jhat | JVM Heap Dump Brower, 用于分心heapdump文件，它会建立一个HTTP/HTML服务器, 让用户可以在浏览器上查看分析结果
jstack | Stack Trce for Java, 显示虚拟机的线程快照

## jps

```js
mubi@mubideMacBook-Pro webapps $ jps --help
illegal argument: --help
usage: jps [-help]
       jps [-q] [-mlvV] [<hostid>]

Definitions:
    <hostid>:      <hostname>[:<port>]
```

* 参数

-q：只输出进程 ID
-m：输出传入 main 方法的参数
-l：输出完全的包名，应用主类名，jar的完全路径名
-v：输出jvm参数
-V：输出通过flag文件传递到JVM中的参数

这些进程的本地虚拟机唯一ID(Local VIrtual Machine Identifier, LVMID), 对于本地虚拟机进程来说，LVMID与操作系统的进程ID(Process Identifier, PID)是一致的

## jstat

```js
mubi@mubideMacBook-Pro webapps $ jstat --help
invalid argument count
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

Definitions:
  <option>      An option reported by the -options option
  <vmid>        Virtual Machine Identifier. A vmid takes the following form:
                     <lvmid>[@<hostname>[:<port>]]
                Where <lvmid> is the local vm identifier for the target
                Java virtual machine, typically a process id; <hostname> is
                the name of the host running the target Java virtual machine;
                and <port> is the port number for the rmiregistry on the
                target host. See the jvmstat documentation for a more complete
                description of the Virtual Machine Identifier.
  <lines>       Number of samples between header lines.
  <interval>    Sampling interval. The following forms are allowed:
                    <n>["ms"|"s"]
                Where <n> is an integer and the suffix specifies the units as
                milliseconds("ms") or seconds("s"). The default units are "ms".
  <count>       Number of samples to take before terminating.
  -J<flag>      Pass <flag> directly to the runtime system.
```

* options

-class                 显示ClassLoad的相关信息；
-compiler           显示JIT编译的相关信息；
-gc                     显示和gc相关的堆信息；
-gccapacity 　　  显示各个代的容量以及使用情况；
-gcmetacapacity 显示metaspace的大小
-gcnew               显示新生代信息；
-gcnewcapacity  显示新生代大小和使用情况；
-gcold                 显示老年代和永久代的信息；
-gcoldcapacity    显示老年代的大小；
-gcutil　　           显示垃圾收集信息；
-gccause             显示垃圾回收的相关信息（通-gcutil）,同时显示最后一次或当前正在发生的垃圾回收的诱因；
-printcompilation 输出JIT编译的方法信息；

* 例1

```js
mubi@mubideMacBook-Pro webapps $ jps
4402 Jps
2611 Launcher
1812
3800
82874
44363 MacLauncher
28461 ApacheJMeter.jar
mubi@mubideMacBook-Pro webapps $ jstat -gc 28461
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
 0.0   1024.0  0.0   1024.0 659456.0 158720.0  388096.0   29131.9   51324.0 49395.6 7344.0 6748.7     27    0.919   5      0.921    1.840
mubi@mubideMacBook-Pro webapps $
```

S0C：第一个幸存区的大小
S1C：第二个幸存区的大小
S0U：第一个幸存区的使用大小
S1U：第二个幸存区的使用大小
EC：伊甸园区的大小
EU：伊甸园区的使用大小
OC：老年代大小
OU：老年代使用大小
MC：方法区大小
MU：方法区使用大小
CCSC:压缩类空间大小
CCSU:压缩类空间使用大小
YGC：年轻代垃圾回收次数
YGCT：年轻代垃圾回收消耗时间
FGC：老年代垃圾回收次数
FGCT：老年代垃圾回收消耗时间
GCT：垃圾回收消耗总时间

* 例2

```js
 jstat -gccapacity 8207
 NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC
     0.0 1048576.0  82944.0    0.0 7168.0  75776.0        0.0  1048576.0   965632.0   965632.0      0.0 1083392.0  40572.0      0.0 1048576.0   6320.0      6     0
```

NGCMN：年轻代(young)中初始化(最小)的大小(字节)
NGCMX：年轻代(young)的最大容量 (字节)
NGC：年轻代(young)中当前的容量 (字节)
S0C：年轻代中第一个survivor（幸存区）的容量 (字节)
S1C：年轻代中第二个survivor（幸存区）的容量 (字节)
EC：年轻代中Eden（伊甸园）的容量 (字节)
OGCMN：old代中初始化(最小)的大小 (字节)
OGCMX：old代的最大容量(字节)
OGC：old代当前新生成的容量 (字节)
OC：Old代的容量 (字节)
MCMN：metaspace(元空间)中初始化(最小)的大小 (字节)
MCMX：metaspace(元空间)的最大容量 (字节)
MC：metaspace(元空间)当前新生成的容量 (字节)
CCSMN：最小压缩类空间大小
CCSMX：最大压缩类空间大小
CCSC：当前压缩类空间大小
YGC：从应用程序启动到采样时年轻代中gc次数
FGC：从应用程序启动到采样时old代(全gc)gc次数

* 例3 每隔1秒打印一次gc信息，总共打印5次

```js
jstat -gcutil 8207 1000 5
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00 100.00  43.24   3.73  97.04  92.86      6    0.058     0    0.000    0.058
  0.00 100.00  43.24   3.73  97.04  92.86      6    0.058     0    0.000    0.058
  0.00 100.00  43.24   3.73  97.04  92.86      6    0.058     0    0.000    0.058
  0.00 100.00  43.24   3.73  97.04  92.86      6    0.058     0    0.000    0.058
  0.00 100.00  43.24   3.73  97.04  92.86      6    0.058     0    0.000    0.058
```
