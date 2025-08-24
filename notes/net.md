# Computer Network

## 小林coding

### TCP/IP 网络模型

TCP/IP 四层：应用层、传输层、网络层、网络接口层

**应用层**在 OS 的用户态；传输层、网络层、网络接口层在内核态

**应用层** case： HTTP、FTP、Telnet、DNS、SMTP

**传输层** 有两个传输协议：TCP 和 UDP

- **TCP**  全称 传输控制协议，HTTP 使用的就是 TCP
- **TCP** 特性：流量控制、超时重传、拥塞控制
- **UDP** 只负责发送，不保证抵达；实施性更好、传输效率更高。

- 分块：TCP 段（TCP Segment）；端口号

**网络层** 常用 IP 协议（Internet Protocol），负责寻址和路由

**网络接口层** 传输数据帧，处理硬件地址（如 MAC 地址）

### 键入网址到网页显示，发生了什么

**输入网址**，例如 `https://www.example.com/index.html`

**URL 解析**

- 协议：`https`
- 域名：`www.example.com`
- 端口：HTTPS 默认是 443，HTTP 是 80
- 路径：`/index.html`
- 查询：例如 `?id=123`

**DNS 域名解析**

- 检查浏览器缓存
- 检查操作系统缓存：`hosts` 文件或系统 DNS 缓存
- 检查路由器缓存
  - 本地 DNS 服务器
  - 递归查询：根域名服务器 ➡ 顶级域名服务器（.com）➡ 权威域名服务器，最终获得 IP

**TCP 连接**

- 三次握手
- HTTPS 还需进行 TLS/SSL 握手
- Linux 可以 ` netstat -napt` 查看 TCP 连接状态

![TCP 连接状态查看](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/%E9%94%AE%E5%85%A5%E7%BD%91%E5%9D%80%E8%BF%87%E7%A8%8B/10.jpg)

**发送 HTTP 请求**

TCP 连接建立后，浏览器发送 HTTP 请求报文，例：

```http
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 ...
Accept: text/html, */*
...
```

**服务器处理请求，返回响应**

例如

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 1234
...

<!DOCTYPE html>
<html>...</html>
```

**浏览器解析渲染**

**关闭连接**：四次挥手

### HTTP 概念

**HTTP**：超文本传输协议

**状态码**:  `200` 表示成功，`301/302` 表示重定向，`404` 表示未找到

**常见字段**

```http
Host: www.A.com
Content-Length: 1000
Connection: Keep-Alive
Content-Type: text/html; Charset=utf-8
Accept: text/html, */*
Content-Encoding: gzip
Accept-Encoding: gzip, deflate
```

- `Content-Length` ：本次回应的长度
- `Content-Length: Keep-Alive`：长连接，任意一端没有明确提出断开连接，则保持 TCP 连接状态
- `Content-Type`：数据格式。`text/html` ：网页； `Charset=utf-8`：编码是UTF-8
- `Accept`：可以接收的数据格式。`*/*` 表示任意格式
- `Content-Encoding`：数据的压缩方法
- `Accept-Encoding`：可接受的数据压缩方法

**GET 与 POST**

- GET：请求获取指定的资源。是安全、幂等、可被缓存的

- POST：根据报文body对指定的资源做出处理。是不安全、不幂等、（大部分时候）不可缓存的

**HTTP 缓存**：包括强制缓存和协商缓存

- **强制缓存**：

  - 如下图：`from disk cache` 就是使用了强制缓存

  ![img](https://cdn.xiaolincoding.com//mysql/other/1cb6bc37597e4af8adfef412bfc57a42.png)

  - HTTP 头的 Cache-Control 和 Expires 字段表示资源在客户端缓存的有效期

- **协商缓存**：客户端与服务端协商，通过协商结果来判断是否使用本地缓存

- 只有在未能命中强制缓存的时候，才能发起带有协商缓存字段的请求

### HTTP vs HTTPS

**HTTP**（HyperText Transfer Protocol）

**HTTPS**（HyperText Transfer Protocol Secure）

HTTPS 在 HTTP 基础上加入了 **SSL/TLS** 加密层，数据在传输前会被加密

> SSL（Secure Sockets Layer）
>
> TLS（Transport Layer Security）

| 特性         | HTTP               | HTTPS              |
| ------------ | ------------------ | ------------------ |
| 安全性       | 不安全（明文传输） | 安全（加密传输）   |
| 加密         | 无                 | SSL/TLS 加密       |
| 默认端口     | 80                 | 443                |
| 是否需要证书 | 否                 | 是（SSL/TLS 证书） |

### RPC

**RPC（Remote Procedure Call，远程过程调用）：** 技术思想，允许程序像调用本地方法一样调用远程服务器

TCP 特点：**面向连接**、**可靠**、**基于字节流**。

裸 TCP 是个无边界的数据流，需要上层协议（HTTP、各类 RPC 协议）定义**消息格式**用于定义**消息边界**

> RPC 本质上不算是协议，而是一种调用方式,而像 gRPC 和 Thrift 这样的具体实现才是协议

RPC有多种实现方式，不一定基于 TCP

HTTP 主要用于 B/S Browser/Server 架构，而 RPC 更多用于 C/S Client/Server 架构

### TCP vs UDP

| 对比维度       | TCP（Transmission Control Protocol） | UDP（User Datagram Protocol）    |
| -------------- | ------------------------------------ | -------------------------------- |
| **连接性**     | 面向连接（需三次握手建立连接）       | 无连接（直接发送）               |
| **可靠性**     | 可靠传输（确认机制、重传、校验）     | 不可靠（不保证送达）             |
| **有序性**     | 保证数据按序到达                     | 不保证顺序                       |
| **传输方式**   | 字节流（Stream）                     | 数据报（Datagram，一个个独立包） |
| **速度与开销** | 较慢，头部大（20-60字节），开销大    | 快，头部小（8字节），开销小      |
| **流量控制**   | 有（滑动窗口）                       | 无                               |
| **拥塞控制**   | 有（慢启动、拥塞避免等）             | 无                               |
| **适用场景**   | 文件传输、网页、邮件、SSH 等         | 视频会议、直播、DNS、游戏        |

### TCP 三次握手与四次挥手面试题

#### 三次握手

名词

- **SYN**：Synchronize，同步标志位，表示建立连接请求
- **ACK**：Acknowledge Flag，确认标志位，表示确认收到数据
- **seq**：Sequence Number，序列号，发送的数据块在整个数据流中的“位置编号”
- **ack**：Acknowledgment Number，期望收到的下一个字节的序列号

整个过程：

![TCP 三次握手](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4/%E7%BD%91%E7%BB%9C/TCP%E4%B8%89%E6%AC%A1%E6%8F%A1%E6%89%8B.drawio.png)

- Client 和 Server 都处于 `CLOSE` 状态
- Client 主动发起连接；Server 监听端口，等待连接
- 【第一次握手】：SYN（Client → Server）
  - Client 发送 SYN=1，seq=x
    - 这里 seq = client_isn，是 Client 随机初始化的
  - Client 状态变为：SYN_SENT
- 【第二次握手】：SYN + ACK（Server → Client）
  - Server 回复 SYN=1，ACK=1，seq=y，ack=x+1
    - 这里 seq = server_isn，是 Server 随机初始化的
    - ack=x+1 表示“收到了 Client 的 seq=x，期待下次收到 x+1”
  - Server 状态变为： SYN-RCVD
- 【第三次握手】：ACK（Client → Server）
  - Client 发送：ACK=1，seq=x+1，ack=y+1
  - C/S 状态均为：ESTABLISHED 

**三次握手的作用**：阻止历史连接

#### 四次挥手

名词

- **FIN**：Finish Flag，结束标志位

整个过程：

![客户端主动关闭连接 —— TCP 四次挥手](https://cdn.xiaolincoding.com//mysql/other/format,png-20230309230614791.png)

- Client 和 Server 都处于 `ESTABLISHED` 状态
- Client 主动关闭连接
- 【第一次挥手】FIN（Client → Server）
  - Client 发送 FYN=1，seq=u
  - Client 状态变为 FIN_WAIT_1
- 【第二次挥手】ACK（Server → Client）
  - Client 发送 ACK=1，seq=v，ack=u+1
    - 含义：“收到关闭请求，但还有一些数据没发，稍等”
  - Server 状态变为 CLOSE_WAIT
  - Client 收到后状态变为 FIN_WAIT_2
- 【第三次挥手】FIN（Server → Client）
  - Server 发送 FIN=1，seq=w，ack=u+1
  - Server 状态变为 LAST_ACK
  - Client 收到后状态变为 TIME_WAIT
- 【第四次挥手】ACK（Client → Server）
  - Client 发送ACK=1，seq=u+1，ack=w+1
  - Client 状态变为 TTIME_WAIT，等待 2MSL 时间后关闭
    - MSL：Maximum Segment Lifetime 报文最大生存时间
  - Server 收到后状态变为 CLOSED


