---
title: jinfo:JAVA进程运行时修改虚拟机参数利器（无需重启）
date: 2017-11-11 17:00:50
tags:
---
> 转 http://www.open-open.com/lib/view/open1437018491912.html

虚拟机总会有出现问题的时候，并且你深深的知道如果通过配置一些参数使得虚拟机打印一些运行时信息，可以通过这种途径改善虚拟机的运行状况。如 XX:+HeapDumpOnOutOfMemoryError和XX:+PrintGCDetails。但是有时候这些参数会丢失，这是件令人头痛的事情。
因此，你痛苦的耸了耸肩，杀掉虚拟机进程，修改启动参数并祈祷之前的糟糕状况可以在重新启动之后重现。现在你拥有足够的证据来查找问题的根源，并改正。前面描述的方式，很显然需要一次额外的重启并添加一些调试参数。实际上，不需要重启虚拟机的方法是有的，这种方法好处是大大的。
JDK打包工具里面为我们提供了一个很好的小工具。jinfo是一个命令行工具，通过这个工具可以获取JAVA进程运行时的一些信息，通过使用jinfo 和一些参数，可以动态修改JAVA进程的虚拟机参数，这些参数并不是万能的，但是在一些场景中是非常有用的。虚拟机的这些参数可以通过下面的命令查看：
```bash
$ java -XX:+PrintFlagsFinal -version| grep manageable
     intx CMSAbortablePrecleanWaitMillis            = 100                                 {manageable}
     intx CMSTriggerInterval                        = -1                                  {manageable}
     intx CMSWaitDuration                           = 2000                                {manageable}
     bool HeapDumpAfterFullGC                       = false                               {manageable}
     bool HeapDumpBeforeFullGC                      = false                               {manageable}
     bool HeapDumpOnOutOfMemoryError                = false                               {manageable}
    ccstr HeapDumpPath                              =                                     {manageable}
    uintx MaxHeapFreeRatio                          = 100                                 {manageable}
    uintx MinHeapFreeRatio                          = 0                                   {manageable}
     bool PrintClassHistogram                       = false                               {manageable}
     bool PrintClassHistogramAfterFullGC            = false                               {manageable}
     bool PrintClassHistogramBeforeFullGC           = false                               {manageable}
     bool PrintConcurrentLocks                      = false                               {manageable}
     bool PrintGC                                   = false                               {manageable}
     bool PrintGCDateStamps                         = false                               {manageable}
     bool PrintGCDetails                            = false                               {manageable}
     bool PrintGCID                                 = false                               {manageable}
     bool PrintGCTimeStamps                         = false                               {manageable}
java version "1.8.0_112"
Java(TM) SE Runtime Environment (build 1.8.0_112-b15)
Java HotSpot(TM) 64-Bit Server VM (build 25.112-b15, mixed mode)
```
我们通过-XX:+PrintFlagsFinal -version参数可以获取JVM的所有选项，然而manageable才是我们所感兴趣的，通过JDK的管理接口（com.sun.management.HotSpotDiagnosticMCBean API）可以动态写入。非常相似的MBean也可以通过Jconsole查看，对于我来说命令行的方式是非常方便的。
下面是一个使用jinfo的例子：
我们通过动态打开JVM的GC日志开关例子来演示jinfo如何使用。
我们先通过jps来查看当前运行状态的虚拟机进程：
```bash
$ jps -l
31907 app.jar
5574 sun.tools.jps.Jps
```
我们通过使用jinfo来打开虚拟机GC日志打印参数，-XX:+PrintGC和-XX:+PrintGCDetails。使用命令行方式微小的差别在于需要为jinfo同时指定-XX:+PrintGC和-XX:+PrintGCDetails参数。除了通过启动脚本可以设置参数，PrintGC默认是打开的，因此我们只需要打开PrintGCDetails参数。
```bash
$ jinfo -flag +PrintGCDDetails 31907
$ jinfo -flag +PrintGC 31907
```
