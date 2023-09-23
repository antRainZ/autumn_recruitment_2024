# 简介
1、OSI7层网络模型：应用层、表示层、会话层、运输层、网络层、链路层、物理层
2、TCP/IP四层网络模型：应用层、运输层、网际层、接口层综合OSI与TCP/IP模型，学习五层网络模型：从上向下架构：应用层、运输层、网络层、链路层、物理层

链路层：
3、MTU
4、MAC地址

网络层：
5、地址解析协议
6、为啥有IP地址还需要MAC地址？同理，为啥有了MAC地址还需要IP地址？
7、网络层转发数据报的流程
8、子网划分、子网掩码
9、网络控制报文协议ICMP
10、ICMP应用举例：PING、traceroute运输层：
11、TCP与UDP的区别及应用场景
12、TCP首部报文格式（SYN、ACK、FIN、RST必须知道）
13、TCP滑动窗口原理
14、TCP超时重传时间选择
15、TCP流程控制
16、TCP拥塞控制（一定要弄清楚与流量控制的区别）
17、TCP三次握手及状态变化。为啥不是两次握手？
18、TCP四次挥手及状态变化。为啥不是三次挥手？
19、TCP连接释放中TIME_WAIT状态的作用
20、SYN泛洪攻击。如何解决？
21、TCP粘包
22、TCP心跳包
23、路由器与交换机的区别
24、UDP如何实现可靠传输

应用层：
25、DNS域名系统。采用TCP还是UDP协议？为什么？
26、FTP协议（了解）
27、HTTP请求报文与响应报文首部结构
28、HTTP1.0、HTTP1.1、HTTP2.0对比
29、HTTP与HTTPS对比
30、HTTPS加密流程
31、方法：GET、HEAD、POST、PUT、DELETE
32、状态码：1、2、3、4、5**33、cookie与session区别
34、输入一个URL到显示页面的流程（越详细越好，搞明白这个，网络这块就差不多了）


## 网络编程
1、IO多路复用：select、poll、epoll的区别（非常重要，几乎必问，回答得越底层越好，要会使用）
2、手撕一个最简单的server端服务器（socket、bind、listen、accept这四个API一定要非常熟练）
3、线程池
4、基于事件驱动的reactor模式
5、边沿触发与水平触发的区别
6、非阻塞IO与阻塞IO区别

# TCP
+ TCP的可靠传输
+ 粘包、拆包
+ 拥塞控制
+ 状态变化

# HTTPS
+ https使用什么机制来保证数据传输的安全性

# 网络编程
+ 网络IO
+ 多路复用：select,poll,epoll
+ reactor的原理与实现
+ http/https 服务器的实现
+ websocket协议与服务器实现
+ 协程框架NtyTcp
+ io_uring

## 网络原理
+ 服务器百万并发实现
+ redis,memcached,nignx 网络组件
+ quic
