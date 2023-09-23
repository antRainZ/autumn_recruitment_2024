# 参考书籍
《深入理解 C++11：C++11 新特性解析与应用》
《深入应用 C++11：代码优化与工程级应用》
《C++17 完全指南》
《Cpp 17 in Detail》

# c
+ 数据类型
  + 基本类型
  + 构造类型
  + 指针类型
  + 空类型
  + 指针与数组,sizeof
+ 函数调用模型
  + 函数指针定义的三种方式
  + 回调函数设计
  + 接口的封装和设计
+ 内存管理,内存模型
  + 内存四区特点、内存操作函数
+ gun c 扩展
  + 结构体初始化
  + 属性
+ c 的面向对象思想
+ 文件操作,输入与输出
  + stat/lstat函数；文件属性相关函数；链接相关概念及函数；目录操作相关概念及函数；
  + dup、dup2函数；
  + fcntl函数
+ 多进程,多线程编程
  + 同步: 锁,信息量,条件变量，屏障
  + 通信: 管道,共享内存,信号量,消息队列,信号
+ 网络编程: TCP,UDP,原始套接字

# 工具
+ gcc: 常见选项,调优,工具链
  + 预处理，编译，汇编，链接
  + 共享库
  + gcc的工作流程和掌握常见参数；
  + Linux下的静态库与共享库(windows动态库)的制作和使用
+ gdb: 启动,断点,检查数据,观察窗口,检查栈,检查源代码,改变程序的运行,多线程调试
+ make: makefile->cmake
+ glibc

# C++
+ 精进基石：数据结构，设计模式，新特性，Linux工程管理
  + stl 容器，智能指针，正则表达式
  + 新特性：线程，协程，原子操作，lamda表达式
  + Linux工程管理：Makefile/cmake/configure, git
+ 高性能网络设计：
  + 网络编程：io, select/poll/epoll, reactor, http, websocket
  + 网络原理：百万并发，同步与异步处理， 中间件， posix api , 网络协议栈
+ 基础组件设计
  + 池式组件：线程池，内存池（内存泄露与定位），异步请求池（流程，kings式四元组（create,commit,callback,destory））,mysql 连接池
  + 高性能组件：原子操作CAS（自旋锁），无锁消息队列（内存屏障），定时器方案（红黑树，时间轮，最小堆），死锁坚持，内存泄露检测，分布式锁‘’
  + 开源组件：libevent/libev, 异步日志方案log4cpp, 应用层协议protobuf/thrift
+ 中间件开发
  + mysql: 体系结构，sql 执行流程，索引原理与优化，事务ACID，缓存策略
  + TiDB,redis,rocksDB
  + nginx: 反向代理
  + mongoDB
  + skynet: 游戏后端开源框架
  + Tars: 分布式RPC框架‘’
  + DPDK 
+ 性能测试：吞吐量，拆链/建链，时延，并发

## 关键字
+ constexper 和 const

## 面向对象
+ 多态是怎样实现的
+ 菱形继承，二义性

## 新特性
+ auto
+ 统一的类成员初始化语法与 std::initializer_list 
+ 左值，右值
+ 引用、万能引用、引用折叠
+ std::move, 完美转发
+ [可调用对象](https://blog.csdn.net/qq_43145072/article/details/103749956)
+ 属性标签（attributes）
+ final/override/=default/=delete 语法
+ Range-based 循环语法
+ 结构化绑定
+ stl 容器新增的实用方法
+ std::thread
+ 线程局部存储 thread_local
+ 线程同步原语 std::mutex、std::condition_variable 等
+ 原子操作类
+ 智能指针类

# 常见问题
+ 指针和引用的概念
+ 指针与内存关系
+ 程序编译过程，静态链接库和动态链接库
+ 面向对象理解, 多态
+ 虚函数与纯虚函数、虚函数实现机制、虚函数表, 继承原理、虚继承、菱形继承
+ new/delete和malloc/free
+ 重载、重写和覆盖
+ 类型转换方式
+ RAII (资源获取即初始化) 与 pimpl 惯用法
+ 内存溢出和内存泄漏
+ STL标准模板库
  + 常用容器特点、用法以及底层实现(vector、list、deque、set、map、unorderedmap)
  + 迭代器、空间配置器理解
+ 左值/右值/std::move/std::forward
+ placement new

## 内存
+ new malloc (类型安全，构造和析构、分配失败处理、大小计算)

# STl

## vector

# 库
+ libevent/libev
+ 异步日志log4cpp
+ 应用层协议：protobug/thrift


# 参考
+ [嵌入式面经111道面试题全解析C/C++可参考](https://blog.csdn.net/feelinghappy/article/details/108087952)
+ [如何能熟练掌握STL？](https://www.zhihu.com/question/37786833/answer/2287955649)