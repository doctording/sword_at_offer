---
title: "JVM命令工具(jstack,jmap,jcmd...)"
layout: page
date: 2019-02-15 00:00
---

[TOC]

# Java自带的各类型命令工具

名称 | 主要作用
-|-
jps | JVM Process Status Tool, 显示指定系统内所有的HotSpot虚拟机进程
jstat | JVM Statistics Monitoring Tool, 用于收集HotSpot虚拟机各方面的运行数据
jinfo | Configuration Info for Java, 显示虚拟机配置信息
jmap | Memory Map for Java, 生成虚拟机的内存转储快照(heapdump文件)
jhat | JVM Heap Dump Brower, 用于分心heapdump文件，它会建立一个HTTP/HTML服务器, 让用户可以在浏览器上查看分析结果
jstack | Stack Trace for Java, 显示虚拟机的线程快照
jcmd | 可以用它来查看Java进程，导出堆、线程信息、执行GC，还可以进行采样分析的一个多功能工具

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

### 例1（jstat -gc）查看gc信息

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

### 例2（jstat -gccapacity）

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

### 例3 (jstat -gcutil)查看gc情况

```js
jstat -gcutil 8207 1000 5
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00 100.00  43.24   3.73  97.04  92.86      6    0.058     0    0.000    0.058
  0.00 100.00  43.24   3.73  97.04  92.86      6    0.058     0    0.000    0.058
  0.00 100.00  43.24   3.73  97.04  92.86      6    0.058     0    0.000    0.058
  0.00 100.00  43.24   3.73  97.04  92.86      6    0.058     0    0.000    0.058
  0.00 100.00  43.24   3.73  97.04  92.86      6    0.058     0    0.000    0.058
```

## jmap

### dump jvm堆内存信息

```java
jmap -dump:format=b,file=xx.hprof <pid>
```

## jstack

### 查看java stack和native stack的线程信息

```java
jstack <PID>
jstack -F <PID>
```

* 参数选项

```java
-F  to force a thread dump. Use when jstack <pid> does not respond (process is hung)
-m  to print both java and native frames (mixed mode)
-l  long listing. Prints additional information about locks
-h or -help to print this help message
```

## jcmd

* 查看当前机器上所有的`jvm`进程信息

```java
jcmd -l
```

* 列出某个jvm进程可以执行的操作

```java
mubi@mubideMacBook-Pro Downloads $ jcmd 22553 help
22553:
The following commands are available:
JFR.stop
JFR.start
JFR.dump
JFR.check
VM.native_memory
VM.check_commercial_features
VM.unlock_commercial_features
ManagementAgent.stop
ManagementAgent.start_local
ManagementAgent.start
GC.rotate_log
Thread.print
GC.class_stats
GC.class_histogram
GC.heap_dump
GC.run_finalization
GC.run
VM.uptime
VM.flags
VM.system_properties
VM.command_line
VM.version
help

For more information about a specific command use 'help <command>'.
mubi@mubideMacBook-Pro Downloads $
```

* 查看进程内存区域的详情(jvm加上`-XX:NativeMemoryTracking=detail`)

```java
mubi@mubideMacBook-Pro Downloads $ jcmd 22553 VM.native_memory detail > 22553.native_detail
mubi@mubideMacBook-Pro Downloads $ head 22553.native_detail
22553:

Native Memory Tracking:

Total: reserved=6033952KB, committed=2097428KB
-                 Java Heap (reserved=4194304KB, committed=1537024KB)
                            (mmap: reserved=4194304KB, committed=1537024KB)

-                     Class (reserved=1125078KB, committed=85206KB)
                            (classes #14483)
mubi@mubideMacBook-Pro Downloads $
```

### Native Memory Tracking(简称:NMT) 分析

参考文档见：<a href="https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr007.html">NMT</a>

```java
22553:

Native Memory Tracking:

Total: reserved=6033952KB, committed=2097428KB
-                 Java Heap (reserved=4194304KB, committed=1537024KB)
                            (mmap: reserved=4194304KB, committed=1537024KB) 
 
-                     Class (reserved=1125078KB, committed=85206KB)
                            (classes #14483)
                            (malloc=6870KB #17632) 
                            (mmap: reserved=1118208KB, committed=78336KB) 
 
-                    Thread (reserved=130665KB, committed=130665KB)
                            (thread #128)
                            (stack: reserved=130048KB, committed=130048KB)
                            (malloc=404KB #641) 
                            (arena=214KB #255)
 
-                      Code (reserved=254310KB, committed=25318KB)
                            (malloc=4710KB #11332) 
                            (mmap: reserved=249600KB, committed=20608KB) 
 
-                        GC (reserved=159021KB, committed=148641KB)
                            (malloc=5777KB #297) 
                            (mmap: reserved=153244KB, committed=142864KB) 
 
-                  Compiler (reserved=177KB, committed=177KB)
                            (malloc=46KB #844) 
                            (arena=131KB #3)
 
-                  Internal (reserved=143983KB, committed=143983KB)
                            (malloc=143951KB #18708) 
                            (mmap: reserved=32KB, committed=32KB) 
 
-                    Symbol (reserved=21199KB, committed=21199KB)
                            (malloc=17164KB #176970) 
                            (arena=4036KB #1)
 
-    Native Memory Tracking (reserved=3693KB, committed=3693KB)
                            (malloc=120KB #1832) 
                            (tracking overhead=3573KB)
 
-               Arena Chunk (reserved=1521KB, committed=1521KB)
                            (malloc=1521KB) 
 
Virtual memory map:
 
[0x00000001025e7000 - 0x00000001025ef000] reserved and committed 32KB for Internal from
    [0x00000001032b4ae8] PerfMemory::create_memory_region(unsigned long)+0x728
    [0x00000001032b41ef] PerfMemory::initialize()+0x39
    [0x0000000103370c79] Threads::create_vm(JavaVMInitArgs*, bool*)+0x13b
    [0x000000010312586e] JNI_CreateJavaVM+0x76
```

* reserved表示应用可用的内存大小, committed 为真正使用的内存

```java
 From the sample output below, you will see reserved and committed memory. Note that only committed memory is actually used. For example, if you run with -Xms100m -Xmx1000m, the JVM will reserve 1000 MB for the Java Heap. Since the initial heap size is only 100 MB, only 100MB will be committed to begin with. For a 64-bit machine where address space is almost unlimited, there is no problem if a JVM reserves a lot of memory. The problem arises if more and more memory gets committed, which may lead to swapping or native OOM situations.
```

### 举例heap内存分布

* 堆可用内存总大小:4194304KB,映射地址`[0x00000006c0000000 - 0x00000007c0000000]`

`committed`的映射地址范围为已经使用的内存

58880KB + 80384KB + 1310720KB + 87040KB = 1537024KB(与Native Memory Tracking中的`Java Heap`一致)

```java
[0x00000006c0000000 - 0x00000007c0000000] reserved 4194304KB for Java Heap from
    [0x00000001033b1e8a] ReservedSpace::initialize(unsigned long, unsigned long, bool, char*, unsigned long, bool)+0x14a
    [0x00000001033b211e] ReservedHeapSpace::ReservedHeapSpace(unsigned long, unsigned long, bool, char*)+0x78
    [0x0000000103380b80] Universe::reserve_heap(unsigned long, unsigned long)+0x84
    [0x000000010329fe27] ParallelScavengeHeap::initialize()+0x89

	[0x00000006c4e80000 - 0x00000006c8800000] committed 58880KB from
            [0x00000001032ce10b] PSVirtualSpace::expand_by(unsigned long)+0x3d
            [0x00000001032c35fa] PSOldGen::expand_by(unsigned long)+0x1c
            [0x00000001032c3724] PSOldGen::expand(unsigned long)+0xa8
            [0x00000001032c380a] PSOldGen::resize(unsigned long)+0xb4

	[0x00000006c0000000 - 0x00000006c4e80000] committed 80384KB from
            [0x00000001032ce10b] PSVirtualSpace::expand_by(unsigned long)+0x3d
            [0x00000001032c3d14] PSOldGen::initialize_virtual_space(ReservedSpace, unsigned long)+0x72
            [0x00000001032c3d85] PSOldGen::initialize(ReservedSpace, unsigned long, char const*, int)+0x49
            [0x0000000102ea09b2] AdjoiningGenerations::AdjoiningGenerations(ReservedSpace, GenerationSizer*, unsigned long)+0x36c

	[0x0000000770000000 - 0x00000007c0000000] committed 1310720KB from
            [0x00000001032ce10b] PSVirtualSpace::expand_by(unsigned long)+0x3d
            [0x00000001032ce7f9] PSYoungGen::resize_generation(unsigned long, unsigned long)+0x57
            [0x00000001032cf032] PSYoungGen::resize(unsigned long, unsigned long)+0x24
            [0x00000001032cc81c] PSScavenge::invoke_no_policy()+0xfb2

	[0x000000076ab00000 - 0x0000000770000000] committed 87040KB from
            [0x00000001032ce10b] PSVirtualSpace::expand_by(unsigned long)+0x3d
            [0x00000001032cee37] PSYoungGen::initialize_virtual_space(ReservedSpace, unsigned long)+0x6f
            [0x00000001032cedb9] PSYoungGen::initialize(ReservedSpace, unsigned long)+0x41
            [0x0000000102ea0962] AdjoiningGenerations::AdjoiningGenerations(ReservedSpace, GenerationSizer*, unsigned long)+0x31c
```

## linux pmap 命令(查看进程的内存映像信息)

参考文档见：<a href="https://linux.die.net/man/1/pmap">linux pmap 帮助文档</a>

The pmap command reports the memory map of a process or processes.

* 参数

-x	extended	Show the extended format.(显示扩展格式)
-d	device	Show the device format.(显示设备格式)
-q	quiet	Do not display some header/footer lines.(不显示头尾行)
-V	show version	Displays version of program.(显示版本)

* 字段说明

1. Address:	start address of map
2. Kbytes:	size of map in kilobytes
3. RSS:	resident set size in kilobytes
4. Dirty:	dirty pages (both shared and private) in kilobytes
5. Mode:	permissions on map: read, write, execute, shared, private (copy on write)
6. Mapping:	file backing the map, or '[ anon ]' for allocated memory, or '[ stack ]' for the program stack
7. Offset:	offset into the file
8. Device:	device name (major:minor)

## linux strace 命令(追踪进程执行时的系统调用和所接收的信号)

参考文档见：<a href="https://linux.die.net/man/1/pmap">linux strace 帮助文档</a>

```java
-p pid

Attach to the process with the process ID pid and begin tracing. The trace may be terminated at any time by a keyboard interrupt signal ( CTRL -C). strace will respond by detaching itself from the traced process(es) leaving it (them) to continue running. Multiple -p options can be used to attach to up to 32 processes in addition to command (which is optional if at least one -p option is given).
```
