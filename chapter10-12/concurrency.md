# 并发编程

如果逻辑控制流在时间上重叠，那么它们就是*并发的*。这种常见的现象称作*并发*。并发可以看做是一种操作系统内核用来运行多个程序的机制。不仅是内核，并发在应用程序中也很重要：

* 访问慢速 I/O 设备。应用程序在等待来自慢速 I/O 设备的数据到达时，内核会先运行其他进程使 CPU 保持繁忙。应用程序也可利用这种方式交替执行 I/O 请求和其他任务来利用并发。

* 与人交互。计算机程序要有同时执行多个任务的能力。每次用户请求某种操作，一个独立的并发逻辑流被创建来执行这个操作。

* 通过推迟工作以降低延迟。有时应用程序可以通过推迟其他工作和并发地执行它们，利用并发降低某些操作的延迟。

* 服务多个网络客户端。非并发中一个服务端只能同时为一个客户端提供服务，一个慢速客户端可能会阻塞后续的其他客户端。利用并发为每个客户端创建独立的逻辑流，可以让服务器同时为多个客户端提供服务。

* 在多核机器上并行计算。许多现代系统配备了多核处理器，被划分成并发流的应用程序通常在多核机器上运行得更快，因为这些流在每个核心上会并行执行，而不是交错执行。

* 进程。用这种方法，每个逻辑流都是一个进程，由内核来调度维护。因为进程有独立的虚拟地址空间，控制流想要和其他流通信必须使用某种显式的进程间通信（IPC[^IPC]）机制。

* I/O 多路复用。应用程序在一个进程的上下文中显式地调度他们自己的逻辑流。逻辑流被模型化为状态机。数据到达文件描述符后，主程序显式的从一个状态转换到另一个状态。因为程序是一个单独的进程，所以所有流都共享同一个地址空间。

* 线程。线程运行在单一进程上下文的逻辑流中，由内核调度。

-------

[^IPC]: Inter-Process Communication，进程间通信。Unix 中可以通过半双工Unix管道、命名管道、消息队列、信号量、共享内存、Socket 等方法实现通信。