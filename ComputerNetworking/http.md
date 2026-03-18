[TOC]



# HTTP 超文本传输协议

## 介绍

>   - 协议的属性和特征（可扩展、无状态、文本协议、请求-响应模型……）
>   - 历史（0.9 → 1.0 → 1.1 → 2 → 3 + HTTPS 普及时间线）
>   - 在 TCP/IP 协议栈中的位置 & 客户端-服务器架构

属性

​	无状态：服务器不维持连接状态，为的是客户端可以方便的从各个主机请求资源

​	可扩展：

​		从协议来看：（extensiablity

​	这是指 HTTP 协议允许在不修改核心协议的情况下，通过添加新的功能来满足不断变化的需求。

- **HTTP Header（报文首部）**：这是最直接的体现。开发者可以自定义 `Header` 字段来传递额外信息，而不会破坏原有的通信逻辑。
- **新方法与状态码**：HTTP 允许引入新的请求方法（如 WebDAV 引入的 `PROPFIND`）和新的状态码，使得协议能适应不同的应用场景。
- **中间组件支持**：由于其简单的请求-响应模式，可以在客户端和服务器之间轻松加入代理（Proxy）、缓存（Cache）或网关，从而在不改变端到端逻辑的情况下扩展功能。

​		从架构来看：（scalability

这是指基于 HTTP 的 Web 系统（如网站或 API 服务）在用户量增加时，通过增加资源来维持性能的能力

- **无状态特性 (Stateless)**：HTTP 的每一个请求都是独立的，服务器不需要保留上一个请求的状态。这使得负载均衡器可以将请求分发到集群中的任何一台服务器上，实现**水平扩展**（增加更多服务器）。
- **缓存机制**：HTTP 内置了强大的缓存控制协议。通过在边缘节点（如 CDN）缓存响应，可以极大地减轻源服务器的负担，提升系统处理海量并发请求的能

​	请求响应模型：客户端发送请求，服务端响应请求，返回某种资源

​	超文本协议: 带有HTML（超文本标记语言）标签的文本，可以被浏览器渲染

历史：

​	每一代http协议的进步。

​	比如说 1.1添加了keep-alive 字段，解决了1.0一次请求一次相应的，相应结束后断开的问题，极大地提高了效率。、

​	通过引入cookie的方式，使得如电商这种需要用户状态（信息）的业务得以展开

​	2.0 相比与前一代，将http报文改成二进制，提高了计算机处理的效率。

​	流水线pipeline : 支持并发输出，加快页面渲染。

​	引入TLS，安全性得以保障

​	3.0 基于UDP传输，并发性再次增强。

​		

## 核心概念与术语

>   - 资源、URI/URL
>   - 超文本与媒体类型
>   - HTTP 方法及其语义（幂等、安全）
>   - 状态码（已写，可再细分常见码场景）
>   - 连接管理（短连接/长连接/管道化/HoL）
>

URL，统一资源定位符 地址

 URI：统一资源指示符 仅指代资源。

​	格式都是 协议://www.domain.../xxx.type

浏览器通过URL就可以获得网站的页面（HTML）通过URI：获得页面的渲染

请求类型：

GET：请求数据。如返回一个页面

POST：提交数据。提交个人信息

PUT：更新数据。

PATCH：更新部分数据

DELETE ：删除数据

特殊方法：

​	HEAD(GET) 仅获取报文头部信息，用于了解资源的大小等等。

​	OPTION：获得服务器所支持的方法



## 报文结构

>   - 请求报文（Start-line + Headers + Body）
>   - 响应报文（Start-line + Headers + Body）
>   - 首部字段分类与常用 Header

请求报文：

​	request line: 请求方法+URL+版本

​	request header ：key-value 

​	\r\n

​	body : data

​	如果是 GET 则为空

​	如果是POST则需要添加按格式（如JSON）添加数据			

```
POST /login HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 27

user=admin&password=123456
```

​	响应报文

```	
HTTP/1.1 200 OK
Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
Content-Length: 0
Content-Type: text/html
Pragma: no-cache
Server: bfe
Date: Wed, 18 Mar 2026 07:19:51 GMT

```



## 状态与会话管理

>   - Cookie 机制
>   - 跨域与 CORS
>   - Token 认证简述

​	cookie 服务器向浏览器发送的小的数据，每次客户端请求时，都会发送cookie ，用来识别客户端，个性化（如定制广告），服务端本身不会存储任何信息。

​	跨域（Cross-Origin）是指浏览器出于安全考虑，限制从一个源（协议+域名+端口），意味着这三个条件必须同时满足，才算同一个源。当加载的脚本访问另一个源的资源，源不同即触发限制。

​	CORS（跨域资源共享）是HTML5标准解决方案，通过服务器配置特定的 HTTP 头，授权不同源的资源访问，从而安全地实现跨域数据传输。

​	典型的header 如

| 响应头                           | 含义                                | 常见值示例                             |
| -------------------------------- | ----------------------------------- | -------------------------------------- |
| Access-Control-Allow-Origin      | 允许哪些源跨域访问我                | `*` 或 `https://frontend.com`          |
| Access-Control-Allow-Methods     | 允许哪些 HTTP 方法                  | `GET, POST, PUT, DELETE`               |
| Access-Control-Allow-Headers     | 允许携带哪些请求头                  | `Content-Type, Authorization, X-Token` |
| Access-Control-Allow-Credentials | 是否允许携带 cookie / Authorization | `true`                                 |
| Access-Control-Max-Age           | 预检请求结果缓存多久（秒）          | `86400` (24小时)                       |

浏览器把跨域请求分成两类，处理方式完全不同：

如果是简单请求，客户端就可以直接（方法是：**GET、HEAD、POST** 三种之一

Content-Type 只能是下面三种：

- application/x-www-form-urlencoded
- multipart/form-data
- text/plain）

这种请求浏览器直接发，服务器只要在响应里带上正确的 Access-Control-Allow-Origin 就行。



非简单请求，除了以上内容都是非简单请求。

机制是：

浏览器先发一个 **OPTIONS** 请求（预检请求）

服务器回复允许（返回上面那些 Allow-* 头）

浏览器收到允许 → 才发出真正的请求（GET/POST/PUT...）

## 缓存机制

>   - 缓存控制与验证
>   - Cache-Control / ETag / Last-Modified



## 版本对比与演进

>   - HTTP/1.0 vs HTTP/1.1（持久连接、Host、管道化）
>   - HTTP/2.0（二进制、多路复用、头部压缩、Server Push）
>   - HTTP/3.0（QUIC、0-RTT、连接迁移、无 HoL）

| 特性                                              | HTTP/1.0                                         | HTTP/1.1                                                     | 实现方式 / 关键变化                                          | 主要价值 / 解决的问题                                        |
| ------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **持久连接 (Persistent Connection / Keep-Alive)** | 非默认，必须显式声明 Connection: Keep-Alive      | **默认就是持久连接** 想关闭才写 Connection: close            | 客户端/服务器只要不写 `Connection: close`，TCP 连接就默认复用 RFC 2616 明确规定持久连接为默认行为 | 减少 TCP 三次握手 + 四次挥手开销 减少慢启动 (slow-start) 影响 网页多资源加载速度显著提升 |
| **Host 头**                                       | 不要求（规范里没有强制） 很多服务器/代理实际会看 | **强制要求**（规范明确必须有） 缺少 Host 会返回 400 Bad Request | 每个请求必须携带 `Host: example.com` 服务器根据 Host 值决定响应哪个虚拟主机 | 支持**一台服务器 + 一个 IP 跑多个域名**（虚拟主机） 解决了共享主机时代的大问题 |
| **管道化 (Pipelining)**                           | 不支持                                           | 支持（但可选）                                               | 客户端在同一个持久连接上**连续发送多个请求**，不等前一个响应回来 服务器必须**按请求顺序**逐个返回响应 | 进一步减少 RTT（往返时延）等待 理论上能明显提速              |
| Chunked Transfer Encoding                         | 不支持                                           | 支持                                                         | Transfer-Encoding: chunked 响应体分块发送，每块前面写长度，最后一块写 0 | 服务器可以**边生成边发送**（不需要先知道全部内容长度） 适合动态页面、流式输出 |
| 更完善的缓存机制                                  | 只有 If-Modified-Since、Expires 等简单字段       | 新增 ETag、If-None-Match、If-Match、Cache-Control 等         | Cache-Control 头成为主流控制方式                             | 更精确的缓存验证与失效控制，节省带宽                         |
| 范围请求 (Range Request)                          | 不规范支持                                       | 规范支持                                                     | Range: bytes=0-499 206 Partial Content 状态码                | 支持**断点续传**、视频拖拽                                   |
| 新增方法                                          | GET、HEAD、POST                                  | 增加 PUT、DELETE、OPTIONS、TRACE、CONNECT、PATCH             | 方法名字直接写在请求行                                       | 更符合 RESTful 风格，为后续 WebDAV 等铺路                    |

关于管道化：由于要求必须顺序返回响应，若出现头一个响应生成的慢，会导致后面的响应会被阻塞（对头阻塞）。一般浏览器默然关闭http1.0的Pipeline 选项。



## 安全与现代扩展（可选）

>   - HTTPS 与 TLS
>   - 内容压缩
>   - WebSocket / SSE
>   - 优先级与性能优化要点

https的工作机制: http + SSL (secure shell layer)

TLS的机制

​	**Core Components of TLS Authentication**

- **Digital Certificates:** Issued by trusted Certificate Authorities (CAs), these contain the public key, domain name, and owner information.  数字签名
- **[Asymmetric Cryptography](https://www.google.com/search?sca_esv=a314a699d70fed9c&sxsrf=ANbL-n7AYdRDvrtJ2VzlFCBRKd5dz-S_tQ%3A1773822009833&q=Asymmetric+Cryptography&sa=X&ved=2ahUKEwjf6uj4gamTAxUskq8BHeEWKVYQgK4QegQIAxAC&biw=1197&bih=653&dpr=2.5):** Used during the handshake to establish trust. The server uses a private key to decrypt data encrypted by the client with the public key. 非对称加密
- **[TLS Handshake](https://www.google.com/search?sca_esv=a314a699d70fed9c&sxsrf=ANbL-n7AYdRDvrtJ2VzlFCBRKd5dz-S_tQ%3A1773822009833&q=TLS+Handshake&sa=X&ved=2ahUKEwjf6uj4gamTAxUskq8BHeEWKVYQgK4QegQIAxAE&biw=1197&bih=653&dpr=2.5):** The initial process where parties verify certificates and agree on a shared secret key for symmetric encryption.  TLS握手

具体流程：

1. **Client Hello:** Client initiates connection, listing supported TLS versions and cipher suites.
2. **Server Hello/Certificate:** Server sends its certificate and public key.
3. **Validation:** Client checks the server's certificate against a trusted CA.
4. **Key Exchange:** Client sends a secret key encrypted with the server's public key.
5. **Handshake Finished:** Both parties confirm they have the correct keys and encryption is established. 





简述WebSocket / SSE

SSE：server-send-event 是一种由服务端发送单向数据的机制。使用http

WebSocket是双向，实现比SSE更复杂，但比SSE支持更多内容，如文本或二进制。

ws 使用自己的协议



###  实现

简单提一句关于http2+http1.1 双栈形式的服务器的架构

顶部：protocol handler： 通过一种叫使用 ALPN（Application-Layer Protocol Negotiation）的方法，让用户去协商使用哪个版本的http协议。

