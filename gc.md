#### gceasy

##### Throughput（吞吐量）

运行用户代码的时间占总运行时间的比例。吞吐量99%意味着100秒的程序执行时间应用程序线程运行了99秒， 而在这一时间段内GC线程只运行了1秒。

**Productive Work（生产性工作）:** This is basically the amount of time your application spends in processing your customer’s transactions.

**Non-Productive Work（非生产性工作）:** This is basically the amount of time your application spend in house-keeping work, primarily **Garbage collection**.

Let’s say your application runs for 60 minutes. In this 60 minutes let’s say 2 minutes is spent on GC activities.

It means application has spent 3.33% on GC activities (i.e. 2 / 60 * 100)

It means application **throughput** is **96.67%** (i.e. 100 – 3.33).

What is the acceptable throughput %?  It depends on the **application（应用程序）** and **business demands（业务需求）**. Typically one should target for more than **95%** throughput.

##### Latency

**Average GC time**：

**Maximum GC time**：during GC pauses, entire（整个） JVM freezes（冻结）

**GC Time Distribution（GC时间分布）:** understand（明白，了解） how many GC events are completing（完成） with in what time range 

##### Footprint（占用空间）

Footprint is basically the amount **CPU** consumed. Based on your GC algorithm（算法）, based on your memory settings, CPU consumption will vary（变化，不同）. Some GC algorithms will consume more CPU (like Parallel, CMS), whereas other algorithms such as Serial will consume less CPU

##### GC Causes

**Allocation Failure（分配失败） ：**Allocation Failure happens when there isn't enough free space to create new objects in Young generation. Allocation failures triggers Young GC.

**Metadata GC Threshold（元空间阈值）：**1. Configured metaspace size is too small than actual requirement 2. There is a classloader leak（类加载器泄露） (very unlikely, but possible)

**Ergonomics （人机工程学）：**GC ergonomics tries to grow or shrink（缩小） the heap to meet a specified goal such as minimum pause time and/or throughput.

