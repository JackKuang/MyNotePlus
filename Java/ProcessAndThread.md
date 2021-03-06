# 进程与线程

## 一、介绍

* 进程

> 一个在内存中运行的应用程序。每个进程都有自己独立的一块内存空间，一个进程可以有多个线程。

* 线程

> 进程中的一个执行任务（控制单元），负责当前进程中程序的执行。一个进程至少有一个线程，一个进程可以运行多个线程，多个线程可共享数据。

* 漫画讲解线程、进程[进程与线程的一个简单解释](http://www.ruanyifeng.com/blog/2013/04/processes_and_threads.html)

![img](http://img.hurenjieee.com/uPic/v2-63a65cbe6d5a4222a1326f2310ef48ee_1440w.jpg)

## 二、区别

| 区别     | 进程                  | 线程                                       |
| -------- | --------------------- | ------------------------------------------ |
| 执行单位 | 操作系统资源分配      | 进程任务调度和资源                         |
| 存储     | 独立的代码和数据空间  | 共享代码和数据空间                         |
| 切换开销 | 较大                  | 较小                                       |
| 内存     | 由操作系统分配        | 由进程分配                                 |
| 信息交互 | 1. Socket<br />2. RPC | 1. 共享内存=>堆<br />2. 调度通信=>多线程锁 |

