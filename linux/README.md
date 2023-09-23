# 简介
+ 先建立起内核的大体框架，理解各个子系统的设计理念和构建思想，了解上层的抽象，提供的API
+ 了解主要函数，设计思想，实现了什么样的功能，达成了什么样的目的

库的概念，什么是动态库和静态库，如何自己制作动态库和静态库


系统移植
uboot移植、rootfs制作、内核剪裁移植，
usb、网卡驱动移植，
uboot、linux启动流程，
自己添加uboot命令

Linux驱动
字符设备架构、inode、cdev、file_operations、file之间关系；
platform总线、设备树；
内存概念
同步互斥机制，自旋锁、信号量、互斥体，原子操作；
中断、中断底半部；
等待队列，poll的实现；
常见设备的驱动的编写和代码分析；
网络设备：网卡驱动分析、netfilter使用；
USB、TTY、SPI、IIC、PCIE等架构。

Unix 系统实现 Linux、基本系统数据类型
文件操作函数: open read wrire close dup fcntl ioctl stat chmod access chdir
系统编程接口的基本特性和高级特性
Linux进程环境、如何创建进程、线程，程序的存储空间分配、环境变量
进程组、会话以及任务控制、进程优先级和调度
动态库和静态库
进程间通信：管道和FIFO、消息队列、信号量、共享内存、内存映射
套接字和网络编程
+ 套接字选项
+ unix domain 协议和编程
+ io多路复用：select、poll、epoll、kqueue
+ 序列化技术
+ 零拷贝技术
+ 开源网络库：muduo、libevent

三高问题：高并发，高可用，高性能,
+ 高并发：多进程，多线程，协程，异步回调，容量评估
+ 高可用：负载均衡，限流隔离，容灾，异地多活，容灾演练
+ 高性能：cdn 网络，池化技术，集群化，缓存技术

1、进程与线程区别
2、线程同步的方式：互斥锁、自旋锁、读写锁、条件变量
3、互斥锁与自旋锁的底层区别
4、孤儿进程与僵尸进程
5、死锁及避免
6、多线程与多进程比较
7、进程间通信：PIPE、FIFO、消息队列、信号量、共享内存、socket
8、管道与消息队列对比
9、fork进程的底层：读时共享，写时复制
10、线程上下文切换的流程
11、进程上下文切换的流程
12、进程的调度算法
13、阻塞IO与非阻塞IO
14、同步与异步的概念
15、静态链接与动态链接的过程
16、虚拟内存概念（非常重要）
17、MMU地址翻译的具体流程
18、缺页处理过程
19、缺页置换算法：最久未使用算法、先进先出算法、最佳置换算法


## 信号
1.信号中的基本概念；2.使用信号相关的函数；3.信号内核实现原理；4.信号捕捉函数signal、sigaction；5.使用信号完成子进程的回收；6.发送信号时如何进行参数传递；

## 高并发
1.多路IO转接模型；2.select函数；3.fd_set相关操作函数；
4.select多路IO转接模型poll操作函数；5.epoll多路IO模型；
6.线程池模型的设计思想；7.多进程并发服务器；
8.多线程并发服务器；
9.libevent库;10.epoll反应堆模型；11.使用BufferEvent、evBuffer；
WebServer

nginx的部署、nginx维护机制、nginx通信架构模型、nginx的高并发性能架构

## 负载均衡
1.Reactor模式并发Service
C++网络服务器框架开发	1.io_buffer缓冲处理2.event callback模型3.定时器队列管理4.定时器队列超时事件5.eventLoop初始化6.tcp/udp server API封装7.常见服务器处理机制

2.DNS与路由Service	
1.初始化one loop per thread模型 server2.route信息存储3.RouerVersion及时间戳存储4.ChangeLog存储5.Backend Thread后台守护线程

3.负载均衡代理Service	
1.节点获取服务2.节点调用结果上报服务3.负载节点调度模型4.健康检查5.LoadBalance负载均衡算法

4.信息上报Service	
1.Single Thread TCP Server模型2.消息封装内容3.一致性hash算法4.hash数据结构

5.开发者API设计	
1.API缓冲层api调度方式2.cpp接口api封装3.python接口api封装4.java接口api封装

6.压力测试	
1.qps压力测试2.单元测试

## 进阶
+ Linux 系统构建，内核裁剪，根文件系统
+ Linux设备驱动开发

## 系统运行时参数命令
ipcs: 进程间通信设施状态
uptime: 运行时长
iostat: cpu 平均负载和磁盘活动
sar: 监控，收集和汇报系统活动
mpstat: 监控多处理器
nmon: 调优和基准测量工具
glances
strace
ftptop
powertop
mytop: mysql线程及性能
htop/top/atop
netstat
ethtool
tcpdump
telnet
iptraf 网络统计信息
iftop 带宽使用情况

## 工程管理
+ makefile/cmake/configure
+ git
+ linux系统运行时参数命令

## 中间件
+ redis
+ mysql
+ kafka
+ gRPC
+ nginx
+ skynet
+ DPDK

## 元原生
+ docker/kubernetes
+ TiDB 数据库
+ 分布式存储Ceph

## 性能分析
+ gtests,火焰图
+ ebpf
+ 

# 进程
+ 进程和线程创建
+ 进程、线程、协程区别
+ 进程、线程的通信方式
+ 怎样的方式可以减少数据竞争

# 内存
+ 虚拟内存、地址映射
+ 运行时，内存分区
+ malloc, brk, mmap 原理
  + MMU、PTW

## 参考
+ [深入理解Linux内存管理](https://blog.csdn.net/orangeboyye/article/details/125998574)
+ [Ariane处理器源码剖析（五）续：MMU](https://zhuanlan.zhihu.com/p/473208374)

# 中断
+ 上下部


# 调试
+ [coredump配置、产生、分析以及分析示例](https://www.cnblogs.com/arnoldlu/p/11160510.html)
+ gdb 命令
