---
title: 面试复习（计算机网络）
date: 2021-02-23 14:29:54
categories: 学习笔记
tag:
 - 面试准备
---

# OSI 模型
 - 哪七层
     * 应用层：协议与端口
     * 表示层
     * 会话层
     * 传输层：TCP、UDP
     * 网络层：IP、ARP
     * 数据链路层：mac地址
     * 物理层：物理字节流传输

# ARP 协议
 - ARP 协议的作用
     * 将 IP 地址转为下一跳的 mac 地址

# TCP 三次握手
| 序号 | 数据发送内容 | 发送方向 | 客户端状态 | 服务器状态 |
|:-:|:-:|:-:|:-:|:-:|
| 0 | | | CLOSED | LISTEN |
| 1 | SYN=1 seq=x | 客户端 -> 服务器 | 发送后转为 SYN_SENT | 接收后转为 SYN_RCVD |
| 2 | SYN=1 ACK=1 ack=x+1 seq=y | 服务器 -> 客户端 | 接收后转为 ESTABLISHED | SYN_RCVD |
| 3 | ACK=1 ack=y+1 | 客户端 -> 服务器 | ESTABLISHED | 接收后转为 ESTABLISHED |

 - 为什么需要最后一次握手
     * 在最后一次握手前，不能确定服务器本身不能确定自己发送的数据能否被客户端所接收到

 - 如果最后一次的 ACK 包服务器没有接收到，即客户端已经进入了 ESTABLISHED 而服务器仍然没有进入到 ESTABLISHED，此时会发生什么
     * 服务器：由于仍然处于 SYN_RCVD 状态下，当超过一定时间后，服务器会重新发送 SYN+ACK 包，直到达到次数上限后，直接关闭本次连接
     * 客户端：因为已经进入了 ESTABLISHED，所以实际上已经可以发送数据了，当发送数据给服务器时，服务器会发现此连接并未建立，此时服务器将会按照 TCP 规则，发送 RST 包给客户端，当客户端接收到此 RST 包后，关闭连接

# TCP 四次挥手
| 序号 | 数据发送内容 | 发送方向 | 客户端状态 | 服务器状态 |
|:-:|:-:|:-:|:-:|:-:|
| 0 | | | ESTABLISHED | ESTABLISHED |
| 1 | FIN=1 seq=x | 客户端 -> 服务器 | 发送后转为 FIN_WAIT_1 | 接收后转为 CLOSE_WAIT |
| 2 | ACK=1 ack=x+1 | 服务器 -> 客户端 | 接收后转为 FIN_WAIT_2 | CLOSE_WAIT |
| 3 | FIN=1 seq=y | 服务器 -> 客户端 | 接收后转为 TIME_WAIT | 发送后转为 LAST_ACK |
| 4 | ACK=1 ack=y+1 | 客户端 -> 服务器 | 发送后等待 2MSL 后转为 CLOSED | 接收后转为 CLOSED |

 - 四次挥手的理解
     * 四次挥手可以认为是两次+两次，一次是由客户端发起的，客户端表示自己的数据已经发送完毕了，通过 FIN 包通知服务器，而服务器接收到后通过 ACK 包向客户端回复表示收到。另一次是由服务器发起的，因为客户端发送完成数据并不代表服务器发送完成数据了，所以还有服务器单独发起的 FIN 包，此时客户端发送 ACK 包表示确定

# HTTP协议
 - 特点
     * 基于 TCP/IP （从 HTTP 3.0 开始，改为采用 UDP 协议）
     * 由客户端发起请求，服务器进行响应
     * 通过 URL 来区分服务
     * 无状态
     * 无连接
     * 提供了八种方法，其中最常见的两种为 `GET` 和 `POST`
 - 端口
     * HTTP 协议默认为 80 端口，HTTPS 协议为 443 端口
 - 状态码

| 状态码 | 含义 |
|:-:|:-:|
| 1XX | 信息状态码 |
| 2XX | 成功状态码 |
| 3XX | 重定向状态码 |
| 4XX | 客户端错误码 |
| 5XX | 服务器错误码 |

 - 常见的状态码

| 状态码 | 含义 |
|:-:|:-:|
| 200 | 成功 |
| 302 | 重定向 |
| 404 | 客户端的 URL 不存在 |
| 500 | 服务器错误 |

 - HTTP 和 HTTPS 的区别
     * HTTP 采用明文传输的方式，而 HTTPS 采用的是加密的方式传输
     * HTTPS 通常需要申请证书
 - GET 和 POST 的区别
     * GET 通常用于获取数据，GET 的请求参数会附在 URL 后，用 `?` 分割 URL 和参数，用 `&` 分割多个参数，特殊字符进行 base64 转码，以明文的方式显示
     * POST 通常用于更新数据，POST 的请求参数会放在数据包的 Body 部分，相对更加安全，浏览器不会进行保存
 - HTTP 1.0 和 1.1 的区别
     * 持久连接
     * 管道机制：同一个连接中，客户端可以发送多个请求
     * 分块传输：服务器每产生一个数据，就发送一个数据
     * 新增了部分请求方式
 - HTTP 1.1 和 2.0 的区别
     * 完全采用二进制进行传输
     * 完全多路复用
     * 报头压缩

# TCP 的可靠传输
 - TCP 的可靠传输的实现
     * 数据校验
     * 数据的合理分片和排序
     * 滑动窗口机制
         + 发送方保留了已经发送的数据的副本，用于重传
         + 发送方接收到 ACK 确认包后才会丢弃副本
         + 发送方维护一个重传时间，当没有接收到 ACK 确认包时将会进行重传
         + 通过拥塞控制来管理滑动窗口的大小
         + 发送方和接收方商讨发送速度，通过控制发送速度保证接收方有足够的内存空间来接受数据
 - TCP “粘包" 现象
     * 原因：TCP 是面向字节流的，而且在遇到多个连续的数据包时，会合并至一个数据包内发送，此时接收方将难以识别并读取内部的数据
     * 解决办法：在每个数据包前加上数据包的长度，使得每次读取数据时可以获取到数据包的长度而读出足够长的数据再进行解析