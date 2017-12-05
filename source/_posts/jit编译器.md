---
title: jit编译器
date: 2017-12-04 10:27:09
tags: java jit
---
# jit编译器
JIT编译器，英文写作Just-In-Time Compiler，中文意思是即时编译器。
> 在Java编程语言和环境中，即时编译器（JIT compiler，just-in-time compiler）是一个把Java的字节码（包括需要被解释的指令的程序）转换成可以直接发送给处理器的指令的程序。当你写好一个Java程序后，源语言的语句将由Java编译器编译成字节码，而不是编译成与某个特定的处理器硬件平台对应的指令代码（比如，Intel的Pentium微处理器或IBM的System/390处理器）。字节码是可以发送给任何平台并且能在那个平台上运行的独立于平台的代码。

### 热点编译
对于程序来说，通常只有一部分代码被经常执行，而应用的性能就取决于这些代码执行得有多快。这些关键代码段被称为应用的热点，代码执行的越多就被认为是越热。  
因此jvm执行代码时，并不会立即编译代码。有两个基本理由。
1. 如果代码只执行一次，那编译完全就是浪费精力，对于只执行一次的代码，解释执行字节码比先编译然后执行的速度快。但如果代码是经常被调用的方法，或者是运行很多次迭代的循环，编译就值得了。编译的代码更快，多次执行累计节约的时间超过了编译所话费的时间。这种权衡是编译器先解释执行代码的原因之一。编译器可以找出哪个方法被调用得足够频繁，可以进行编译。
2. jvm执行特定方法或者循环的次数越多，它就会越了解这段代码。这使得jvm可以在编译代码时进行大量优化。

### 代码缓存
jvm编译代码时，会在代码缓存中保留编译之后的汇编语言指令集。代码缓存的大小固定，所以一旦填满，jvm就不能编译更多代码了。  
如果代码缓存过小，就可能会有一些热点被变异了，儿其他则没有。代码缓存填满时，jvm会发出以下警告
> Java HotSpot(TM) 64-Bit Server VM warning: CodeCache is full. Compiler has been disabled.

> -XX:ReservedCodeCacheSize=N标志可以设置代码缓存的最大值

使用jconsole memory(内存)面板的Memory Pool(内存池) Code Cache图表，可以监控代码缓存

### 编译阀值
代码执行的频度是触发代码编译最主要的因素。一旦执行达到一定次数，且达到了编译阀值，编译器就可以获得足够的信息编译代码了。  
编译是基于两种jvm计数器的：方法调用计数器和方法中的循环回边计数器。回边实际上可看作是循环完成执行的次数。  
jvm执行某个java方法时，会检查该方法的两种计数器总数，然后判定该方法是否适合编译。如果适合，该方法就进入编译队列。

### 检测编译过程
如果开启PrintCompilation(-XX:+PrintCompilation)，每次编译一个方法或循环时，jvm就会打印一行被编译的内容信息。  
编译日志需要在程序启动时开启PrintCompilation，如果程序启动时没有开启这个标志，可以使用jstat了解编译器内部的部分工作情况
```bash
$ jstat -compiler 8844
Compiled Failed Invalid   Time   FailedType FailedMethod
   28695      4       0    88.39          1 com/intellij/openapi/vfs/newvfs/impl/VirtualDirectoryImpl a
```

