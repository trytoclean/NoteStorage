下面按你点名的方向继续：**RTMP vs RTSP：状态机复杂度对比（工程实现视角）**。我会用同样的“状态机语言”来讲清楚两者的**控制面、数据面、会话模型**差异，并给你一个**落地到代码结构**的对照。

------

# 1. 先给结论：复杂度差异来自哪里

**RTMP 更像“单通道的消息总线”**：

- 连接上所有东西都走一条 TCP（控制 + 音频 + 视频）
- 复杂度集中在：**Chunk 分块/重组 + Header 继承 + 多 csid 上下文**

**RTSP 更像“控制平面 + 独立媒体承载”**：

- 控制走 RTSP（通常 TCP）
- 媒体走 RTP/RTCP（UDP 或 TCP interleaved）
- 复杂度集中在：**会话/事务（CSeq、Session）、SDP、RTP/RTCP、传输协商（Transport）**

一句话：

> RTMP 的难点在“**怎么把字节流还原成语义消息**”；RTSP 的难点在“**怎么把控制与媒体编排成一个可运行的会话**”。

------

# 2. 连接/会话模型对比（决定状态机形状）

## 2.1 RTMP：Connection 为中心

- 一条连接上跑多个逻辑 stream（Message Stream ID）
- “会话”基本就是“连接本身 + stream 状态”

状态机是“**单连接内多子状态**”：

```
ConnState + ProtoState + StreamState + ChunkFSM
```

## 2.2 RTSP：Session 为中心（并且是事务驱动）

- RTSP 是典型的 request/response 协议（类似 HTTP）
- 但它维护服务器端的 **Session（Session: header）**
- 同一 TCP 连接上可以多个 session（实现上需支持）

状态机是“**事务序列 + 会话状态**”：

```
TransactionFSM(CSeq) + SessionFSM(SETUP/PLAY/TEARDOWN) + MediaTransportFSM(RTP/RTCP)
```

------

# 3. 控制面状态机对比（命令序列）

## 3.1 RTMP 控制面 FSM（推流/拉流）

典型序列（你之前已经有了）：

- connect → createStream → publish / play

控制面状态大致 3~4 个：

```
WAIT_CONNECT -> WAIT_CREATE_STREAM -> READY -> (PUBLISHING/PLAYING)
```

## 3.2 RTSP 控制面 FSM（典型点播）

最小可用序列通常是：

- OPTIONS（可选）
- DESCRIBE（拿 SDP）
- SETUP（为每条 track 建立传输）
- PLAY（开始发 RTP）
- TEARDOWN（结束）

注意：**SETUP 是按 track 重复的**（音频一个、视频一个）

会话 FSM 更细：

```
INIT
  -> DESCRIBED (拿到 SDP)
  -> SETUPPING (track1 SETUP, track2 SETUP, ...)
  -> READY
  -> PLAYING
  -> TEARDOWN/CLOSED
```

你会看到：RTSP 的控制面状态比 RTMP 更“长”，并且带循环（多 track）。

------

# 4. 数据面状态机对比（真正难的部分）

## 4.1 RTMP 数据面：ChunkFSM 是主战场

数据面难点是**协议本身定义的字节组织方式**：

- fmt 0/1/2/3 header 继承
- csid 上下文
- chunk size 动态变化
- message 跨 chunk 重组

所以你需要一个“流式解析器”状态机：

```
READ_BASIC_HEADER
READ_MESSAGE_HEADER
READ_EXT_TIMESTAMP?
READ_CHUNK_PAYLOAD
REASSEMBLE_MESSAGE
DISPATCH_MESSAGE(Audio/Video/Command)
```

**这是 RTMP 复杂度的核心贡献者**。

## 4.2 RTSP 数据面：RTP/RTCP 是主战场（而且是另一套协议族）

RTSP 自己的数据面很薄：就是 RTSP 文本报文解析（类似 HTTP）。
真正的音视频数据在 RTP：

- RTP packet 解析
- 序列号/时间戳处理
- 抖动缓存（Jitter buffer，客户端更重）
- 丢包、乱序（UDP）
- RTCP SR/RR（统计、同步、反馈）

如果 RTP over TCP interleaved，还要加一个“帧封装”：

```
$ <channel-id> <len-hi> <len-lo> <RTP/RTCP bytes>
```

所以 RTSP 的数据面复杂度，来自“**你不得不实现 RTP/RTCP 或至少正确打包发送**”，而不是 RTSP 本身。

------

# 5. 复杂度量化对比（工程实现拆项）

下面是一个务实的“实现项清单”对比：

## 5.1 RTMP 主要实现项

- Handshake（C0C1/S0S1S2/C2）
- Chunk 解析/组包（最难）
- AMF0 编解码（命令、元数据）
- publish/play 状态与 stream 管理
- 音视频 payload（H264/AAC 常见）封装为 RTMP message

## 5.2 RTSP 主要实现项

- RTSP 报文解析（请求行/头/体）
- 方法处理：OPTIONS/DESCRIBE/SETUP/PLAY/TEARDOWN
- 生成 SDP（媒体描述、sprop-parameter-sets 等）
- Transport 协商（UDP 端口对、TCP interleaved 通道）
- RTP 打包（payload format：H264/AAC）
- RTCP（至少 SR，很多实现先简化）

结论：

- **RTMP：协议解析难（chunk/amf），媒体发送相对直线**
- **RTSP：协议解析不难（类似 HTTP），但媒体承载体系更复杂（RTP/RTCP/Transport/SDP）**

------

# 6. 对照的“状态机数量与耦合度”

| 维度       | RTMP                               | RTSP                                   |
| ---------- | ---------------------------------- | -------------------------------------- |
| 控制面 FSM | 短（3~4 状态）                     | 长（含多 track SETUP 循环）            |
| 数据面 FSM | ChunkFSM 重                        | RTP/RTCP FSM 重                        |
| 会话模型   | 连接中心（connection-centric）     | session-centric（Session header）      |
| 复用方式   | chunk stream + message stream      | track + RTP SSRC/channel               |
| 最易踩坑   | fmt=3 继承、chunk 重组、chunk size | Transport 协商、SDP 正确性、RTP 时间戳 |

------

# 7. 代码结构建议：两套协议怎么落到架构里

## 7.1 RTMP（解析器中心）

```
RtmpConnection
  - HandshakeFSM
  - ChunkParser (per csid context)
  - AmfCommandDispatcher
  - StreamManager (publish/play)
  - MediaSender (audio/video message)
```

## 7.2 RTSP（会话/track 中心）

```
RtspConnection
  - RtspParser (text)
  - RtspTransactionManager (CSeq)
  - RtspSessionManager (Session)
  - SdpBuilder
  - TransportNegotiator (UDP/TCP interleaved)
  - RtpPacketizer (per track)
  - Rtcp (optional/phase2)
```

------

# 8. 选择实现路线：如果你的目标是“能跑起来”

给你一个非常实用的路线建议（按你想做 server 的目标）：

- 想最快看到播放成功（工程 MVP）：**RTSP（TCP interleaved）通常更直接**
  因为：控制报文直观、Wireshark 好调、客户端（ffplay/VLC）成熟。
- 想练“协议解析硬功”：**RTMP 更练字节流状态机**
  因为：chunk 继承 + 重组是典型难点。

------

# 9. 你要我下一步继续什么？

我可以顺着你当前方向，继续往下推进两条路线中的一条（不需要你补材料也能开始）：

1. **给出 RTSP 的最小可用状态机 + handler 列表（OPTIONS/DESCRIBE/SETUP/PLAY/TEARDOWN）**
2. **给出 RTMP ChunkFSM 的可直接编码的伪代码（含 csid 上下文、fmt 继承、重组）**
3. **做一张“RTMP vs RTSP vs SRT”三者工程对照表（适用场景 + 复杂度 + 关键坑）**

你只回复一个编号即可，我继续。