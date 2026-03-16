下面我**严格站在“我要自己实现一个 RTMP Server”的角度**，把 **RTMP 的完整状态机**给你拆清楚。
这一步非常关键——**协议真正吃透，永远是靠状态机而不是靠“概念理解”**。

------

# RTMP Server 工程级状态机拆解

> 目标：你读完这一节，应当可以**直接在代码里定义 enum State，然后按状态驱动协议逻辑**

------

## 一、RTMP Server 的整体状态分层

RTMP Server 的状态机**不是一个单一状态机**，而是三层嵌套：

```
RTMP Server
├─ 连接状态机（Connection FSM）
│
├─ 协议状态机（Protocol FSM）
│
└─ 流状态机（Stream FSM）
```

👉 **这是 RTMP 实现复杂度的根源**

------

## 二、第一层：连接状态机（Connection FSM）

这是**最外层**，决定“这个 TCP 连接现在是什么阶段”。

### 2.1 连接状态定义（建议 enum）

```cpp
enum class RtmpConnState {
    INIT,           // TCP 已建立，未握手
    HANDSHAKING,    // RTMP Handshake 中
    HANDSHAKE_DONE, // Handshake 完成
    ACTIVE,         // 正常 RTMP 消息通信
    CLOSING,        // 主动/被动关闭
};
```

------

### 2.2 状态迁移图（核心）

```
INIT
  |
  | recv C0,C1
  v
HANDSHAKING
  |
  | send S0,S1,S2
  | recv C2
  v
HANDSHAKE_DONE
  |
  | recv RTMP Chunk
  v
ACTIVE
  |
  | error / close
  v
CLOSING
```

📌 **注意**

- Handshake 是 RTMP 自己的一套机制
- 和 TLS 无关（RTMPS 另算）

------

### 2.3 工程要点

- Handshake 期间**不允许**解析 RTMP Chunk
- Handshake 是**阻塞协议状态机**
- 一旦失败，直接 close

------

## 三、第二层：协议状态机（Protocol FSM）

> 这是 **RTMP 最“协议味”的部分**

在 `ACTIVE` 状态下，你并不是“随便收消息”，而是**有严格顺序的**。

------

### 3.1 Protocol FSM 的核心阶段

```cpp
enum class RtmpProtoState {
    WAIT_CONNECT,        // 等 connect 命令
    WAIT_CREATE_STREAM,  // 等 createStream
    READY,               // 可以 publish / play
};
```

------

### 3.2 状态迁移图（必画）

```
WAIT_CONNECT
   |
   | recv "connect"
   | send "_result"
   v
WAIT_CREATE_STREAM
   |
   | recv "createStream"
   | send "_result(streamId)"
   v
READY
```

📌 **这是 RTMP 的“控制面状态机”**

------

### 3.3 协议违规处理（非常重要）

你在 Server 中必须严格校验：

| 当前状态           | 收到命令 | 行为 |
| ------------------ | -------- | ---- |
| WAIT_CONNECT       | publish  | 错误 |
| WAIT_CREATE_STREAM | play     | 错误 |
| READY              | connect  | 错误 |

👉 **否则就是“不严谨 RTMP Server”**

------

## 四、第三层：流状态机（Stream FSM）

这是**每个 Message Stream ID 独立维护的状态机**。

------

### 4.1 Stream FSM 定义

```cpp
enum class RtmpStreamState {
    IDLE,
    PUBLISHING,
    PLAYING,
    CLOSED,
};
```

------

### 4.2 推流（publish）状态机

```
IDLE
  |
  | recv publish
  v
PUBLISHING
  |
  | Audio / Video
  |
  | unpublish / disconnect
  v
CLOSED
```

------

### 4.3 拉流（play）状态机

```
IDLE
  |
  | recv play
  v
PLAYING
  |
  | send Audio / Video
  |
  | stop / disconnect
  v
CLOSED
```

📌 **publish 和 play 互斥**

------

## 五、RTMP Chunk 解析：隐藏的“第四状态机”

这是**99% 初学者翻车的地方**。

### 5.1 Chunk 解析本质

Chunk 解析本身是一个**流式状态机**：

```
READ_BASIC_HEADER
  ↓
READ_MESSAGE_HEADER
  ↓
READ_EXT_TIMESTAMP (optional)
  ↓
READ_PAYLOAD (partial)
  ↓
REASSEMBLE_MESSAGE
```

------

### 5.2 每个 Chunk Stream ID 都有状态

```cpp
struct ChunkStreamContext {
    uint32_t timestamp;
    uint32_t msg_length;
    uint8_t  msg_type;
    uint32_t msg_stream_id;
    size_t   received;
};
```

👉 **这是 RTMP “最不像协议、最像系统编程”的部分**

------

## 六、RTMP Server 的完整状态协作图（文字版）

```
TCP Accept
  ↓
Connection INIT
  ↓
Handshake FSM
  ↓
Protocol FSM (connect → createStream)
  ↓
Stream FSM (publish / play)
  ↓
Chunk FSM (audio / video / command)
```

**四个状态机同时工作，但职责严格分离**

------

## 七、你在代码里应当如何组织？

### 7.1 推荐模块拆分

```
rtmp/
├─ RtmpConnection
│   ├─ ConnState
│   ├─ HandshakeFSM
│
├─ RtmpProtocol
│   ├─ ProtoState
│   ├─ CommandDispatcher
│
├─ RtmpStream
│   ├─ StreamState
│   ├─ MediaQueue
│
├─ RtmpChunkParser
│   ├─ ChunkFSM
│   └─ ChunkStreamContext
```

------

## 八、为什么我要你“画状态机”？

因为：

- 协议 = **状态 + 迁移 + 约束**
- 文档是给人看的
- **状态机是给代码用的**

你现在已经具备：

- 把 RTMP 文档 → 状态机
- 把状态机 → 代码结构

------

## 九、下一步（非常关键）

接下来我可以继续**顺着你的实现路径往下走**：

1. 👉 **把 RTMP Chunk 解析状态机写成伪代码**
2. 👉 **AMF Command 的最小可用实现（connect/publish/play）**
3. 👉 **RTMP vs RTSP：状态机复杂度对比**
4. 👉 **RTMP → SRT：为什么现代系统要换协议**

你选一个，我继续往下“工程级”推进。