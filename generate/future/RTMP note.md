下面我**严格复用上一条给你的“协议通用模板”**，用**工程实现视角**完整拆解 **RTMP（Real-Time Messaging Protocol）**。
目标不是“知道 RTMP 是什么”，而是：**你将来真的能写、能读、能调一个 RTMP server/client**。

------

# RTMP 协议工程化拆解笔记（通用模板版）

------

## 0️⃣ 一句话总览（终极复习用）

> **RTMP** 是一种位于 **应用层**、基于 **持久 TCP 连接** 的实时流媒体协议，通过 **二进制分块消息机制** 实现 **低延迟音视频与控制消息的双向传输**，适合推流场景，但对弱网和现代浏览器支持较差。

------

## 1️⃣ Why：RTMP 解决什么问题？

### 1.1 背景问题

HTTP 的问题：

- 请求 / 响应模型
- 高延迟
- 不适合连续音视频数据

RTSP 的问题：

- 控制复杂
- NAT / 防火墙穿透差
- 客户端实现成本高

### 1.2 RTMP 的目标

RTMP 的核心设计目标是：

- **低延迟**
- **长连接**
- **服务端主动推送**
- **音视频 + 控制消息统一通道**

### 1.3 RTMP 的典型使用场景

- 直播推流（OBS → CDN）
- 音视频会议（早期）
- 实时互动应用（Flash 时代）

------

## 2️⃣ Where：RTMP 在协议栈中的位置

```
Application ── RTMP
Transport   ── TCP
Network     ── IP
Link        ── Ethernet / WiFi
```

### 2.1 关键依赖关系

- 强依赖 TCP
- 自己实现：
  - 消息边界
  - 流复用
  - 时序控制

### 2.2 RTMP 的变体

| 变体  | 说明                       |
| ----- | -------------------------- |
| RTMP  | TCP 明文                   |
| RTMPS | RTMP over TLS              |
| RTMPT | RTMP over HTTP（穿防火墙） |

------

## 3️⃣ What：RTMP 数据长什么样？（最重要）

RTMP 是**二进制协议**，核心由 **Chunk + Message** 构成。

------

### 3.1 RTMP 的分层数据模型（非常关键）

```
RTMP Connection
├─ Chunk Stream
│  ├─ Message
│  │  ├─ Audio
│  │  ├─ Video
│  │  └─ Command
```

**不是一包 TCP = 一个消息**

------

### 3.2 Chunk（分块机制）

RTMP 为了解决：

- TCP 粘包
- 大包阻塞
- 多流公平性

引入 **Chunk**。

#### Chunk 结构（简化）

```
+------------------+
| Basic Header     |
+------------------+
| Message Header   |
+------------------+
| Extended TS(opt) |
+------------------+
| Chunk Data       |
+------------------+
```

#### Chunk Header 类型（fmt）

| fmt  | 含义           |
| ---- | -------------- |
| 0    | 完整 header    |
| 1    | 去掉 stream id |
| 2    | 只剩 timestamp |
| 3    | 全部复用       |

📌 **这是 RTMP 最难、也最工程化的部分**

------

### 3.3 Message（逻辑消息）

Chunk 只是传输单位，**Message 才是语义单位**。

#### Message 结构

```
Message
├─ Timestamp
├─ Message Length
├─ Message Type ID
├─ Message Stream ID
├─ Payload
```

------

### 3.4 常见 Message Type

| Type ID | 含义           |
| ------- | -------------- |
| 1       | Set Chunk Size |
| 8       | Audio          |
| 9       | Video          |
| 20      | Command AMF0   |
| 18      | Data AMF0      |

------

## 4️⃣ How：RTMP 如何通信？（流程 & 状态机）

------

### 4.1 RTMP 建立流程（必背）

```
Client                Server
  | --- C0,C1 ------> |
  | <--- S0,S1,S2 --- |
  | --- C2 --------> |
```

这是 RTMP Handshake（非 TLS）

#### Handshake 的本质

- 同步时间
- 确认协议版本
- 防止垃圾连接

------

### 4.2 连接建立后的典型流程（推流）

```
connect
  ↓
createStream
  ↓
publish
  ↓
Audio / Video Message
```

拉流流程：

```
connect
  ↓
createStream
  ↓
play
```

------

### 4.3 Command Message（控制信令）

RTMP 的“信令”靠 **AMF 编码的 Command Message**。

例如：

```
connect(app, tcUrl, ...)
createStream()
publish(streamName)
play(streamName)
```

📌 **RTMP = 二进制协议 + AMF**

------

## 5️⃣ Guarantee：RTMP 提供哪些保证？

### 5.1 提供的保证

- 基于 TCP 的可靠传输
- 消息顺序性
- 低延迟（相对 HTTP）

### 5.2 不提供的保证

- 不保证实时性（TCP 本质）
- 不保证弱网下可用
- 不适合大规模多终端播放

------

## 6️⃣ Trade-off：RTMP 的代价与边界

### 6.1 优点

- 实现简单（相比 WebRTC）
- 延迟低
- 推流稳定

### 6.2 致命缺点

- 强依赖 TCP
- Flash 死亡
- 浏览器原生不支持
- 高延迟 CDN 播放

📌 **结论：RTMP 适合“推”，不适合“播”**

------

## 7️⃣ 工程实现视角（你真正该记的）

### 7.1 实现一个 RTMP Server，你要解决什么？

#### 网络层

- 长连接
- 高并发 TCP
- 粘包 / 拆包

#### 协议层

- Chunk 解析状态机
- Message 重组
- Header 复用逻辑

#### 控制层

- AMF 编解码
- Command 路由
- Stream 管理

#### 媒体层

- H.264 / AAC
- 时间戳对齐
- GOP 管理

------

### 7.2 RTMP 的“坑点清单”

- Chunk Size 动态变化
- fmt=3 的 header 继承
- Timestamp 回绕
- Message 跨多个 chunk
- 音视频交错顺序

------

## 8️⃣ 用模板总结 RTMP（标准收尾）

> RTMP 是一种基于 TCP 的应用层实时流媒体协议，通过 Chunk + Message 机制解决长连接中多路音视频与控制消息的公平传输问题，适合低延迟推流场景，但在弱网、浏览器和大规模分发方面存在明显局限。

------

## 九、下一步建议（非常重要）

你现在已经有了 **HTTP → RTMP** 的统一协议理解方式。
接下来我**强烈建议**按这个顺序继续：

1. **RTSP（对照 RTMP）**
2. **SRT（现代弱网方案）**
3. **QUIC（传输层革命）**
4. **WebRTC（终极实时）**

如果你愿意，我可以直接下一步帮你做其中之一：

- 👉 **用同一模板拆 RTSP（和 RTMP 正面对比）**
- 👉 **站在“自己写 RTMP Server”的角度画状态机**
- 👉 **RTMP vs RTSP vs SRT 工程级对照表**

你只需要选一个。