---
title: linux iostat
date: 2017-11-06 15:24:38
tags:
---
### 简介
> iostat主要用于监控系统设备的IO负载情况，iostat首次运行时显示自系统启动开始的各项统计信息，之后运行iostat将显示自上次运行该命令以后的统计信息。用户可以通过指定统计的次数和时间来获得所需的统计信息。

1. **-d** 显示设备（磁盘）使用状态
2. **-k** 指定iostat的部分输出结果以kB为单位，而不是以扇区数为单位
3. **-c** 显示更详细的io设备统计信息
```bash
$ iostat -dkx 1 5
Linux 2.6.32-696.1.1.el6.x86_64 (iZuf650me9vkz4vfzq4htwZ) 	11/06/2017 	_x86_64_	(4 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.02    26.90    0.28    1.23    10.82   112.50   164.15     0.08   53.10    6.34   63.66   1.95   0.29

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     2.00    0.00    2.00     0.00    16.00    16.00     0.00    1.00    0.00    1.00   1.00   0.20

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
```
* **rrqm/s** 每秒对该设备的读请求被合并次数，文件系统会对读取同块(block)的请求进行合并
* **wrqm/s** 每秒对该设备的写请求被合并次数
* **r/s** 每秒完成的读取次数
* **w/s** 每秒完成的写次数
* **rkB/s** 每秒读数据量(kB为单位)
* **wkB/s** 每秒写数据量(kB为单位)
* **avgrq-sz** 平均每次IO操作的数据量(扇区数为单位)
* **avgqu-sz** 平均等待处理的IO请求队列长度
* **await** 平均每次IO请求等待时间(包括等待时间和处理时间，毫秒为单位)
* **r_await** 平均每次IO请求读等待时间
* **w_await** 平均每次IO请求写等待时间
* **svctm** 平均每次IO请求的处理时间(毫秒为单位)
* **%util** 采集周期内用于IO操作的时间比率，即IO队列非空的时间比率
