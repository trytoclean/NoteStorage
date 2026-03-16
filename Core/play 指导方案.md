好，那我们先完全**不谈代码**，只谈“结构”和“逻辑”，把 PLAY 阶段当成一个“系统设计题”。

------

## 一、先从 10,000 米高度看：PLAY = 控制流 + 数据流

在 RTSP 里，PLAY 其实就两条主线：

1. **控制流（Control Plane）**
   - 客户端发一个 `PLAY` 请求
   - 服务器解析、检查、更新会话状态，返回 `200 OK`
2. **数据流（Data Plane）**
   - 一旦允许播放，对应的会话开始“按时间节奏”把媒体数据变成 RTP 包，发给客户端
   - 直到被 `PAUSE` / `TEARDOWN` / 超时打断

所以从结构上看，你要解决的就是：

> **如何把“控制 PLAY 的那一条线”，和“真正发 RTP 的那一条线”，清晰地分开，并且用“Session”把两条线连起来。**

------

## 二、第一层拆分：PLAY 涉及的 4 大子系统

从大的方向看，可以先切成这 4 块（名字只是方便讨论，不要求完全照抄）：

1. **RTSP 控制处理子系统**（收/发 RTSP 报文）
2. **Session 与状态管理子系统**（知道谁在播、播到哪一步）
3. **媒体源与时间轴子系统**（从哪儿拿数据，什么时间点拿什么帧）
4. **RTP 发送调度子系统**（按照时间节奏把 RTP 包扔出去）

你现在问的是 PLAY 阶段，但实际上它是这四块的 “交汇点”。

------

## 三、第二层拆分：每个子系统的结构 & 逻辑

### 1. RTSP 控制处理子系统（PLAY 只是其中一种 Method）

**目标：**
 把“网络上的字符串请求”变成“服务器内部的结构化操作”，例如：

- `handlePlay(request, connection)`

**内部可以再拆：**

- **连接级别**（Connection）
   负责：收 TCP 数据、按行拆分、交给解析器、把响应写回去。
   PLAY 时：
  - 从客户端读到一行行文本 → 交给 Parser，得到一个 `RtspRequest` 对象。
- **解析器（Parser）**
   负责：
  - 把文本转换成：Method、URI、Headers、Body 等结构化信息。
     PLAY 时：
  - 得到：`method = PLAY`、`CSeq`、`Session`、可选的 `Range` 等。
- **路由 / 分发（Dispatcher）**
   负责：
  - 根据 Method 分发到不同 handler，例如：`PlayHandler`。
     PLAY 时：
  - 调用：`PlayHandler::onPlay(request, connection)`。

> 在这个子系统里，PLAY 的本质就是：
>  **识别 PLAY 请求 → 调用“播放逻辑”模块 → 把结果封装成 RTSP 响应发回去。**
>  它不关心“怎么发 RTP”。

------

### 2. Session 与状态管理子系统（PLAY 关注“这个会话是否允许播放”）

PLAY 不会凭空出现，它一定基于一个已经存在的 Session（通常是 SETUP 阶段创建的）。

**核心对象：Session**

一个 Session 逻辑上要包含（不用代码，讲结构）：

- Session 标识：`session_id`
- 当前状态：
  - `INIT / READY / PLAYING / PAUSED / TEARDOWN`
- 传输参数：
  - 使用 UDP 还是 TCP interleaved
  - 客户端地址、端口
  - 服务端用于发 RTP/RTCP 的“通道”（socket 或 interleaved channel）
- 媒体信息：
  - 绑定了哪个媒体源（文件/摄像头/推流）
  - 有哪些 track（video/audio）
- RTP 运行时信息：
  - 当前 seq、timestamp、SSRC
- 发送任务状态：
  - 目前有没有一个“正在跑的发送循环/线程”
  - 是否处于“暂停”状态
- 生命周期信息：
  - 最后一次活动时间（用来做超时回收）

**还有一个管理者：SessionManager**

- 按 `session_id` 查找 Session
- 新建、删除 Session
- 定期检查超时 Session

**在 PLAY 逻辑里，与 Session 的典型交互：**

当 `PlayHandler` 收到一个 PLAY 请求时，它会按顺序做：

1. 从请求头中读出 `Session` 字段
2. 用 `SessionManager` 找到对应的 Session
3. 检查这个 Session 当前状态：
   - 是否存在？
   - 是否处于 `READY` 或 `PAUSED`（允许 PLAY 的状态）？
4. 更新状态为 `PLAYING`
5. 标记/触发对应发送任务开始工作（交给“RTP 发送调度子系统”）

> 所以从结构角度说：
>  **PlayHandler 并不直接发包，而是改 Session 的“控制字段”，然后通知专门管“发数据”的那一块。**

------

### 3. 媒体源与时间轴子系统（PLAY 关心“从哪里开始播”）

PLAY 的语义是：“从某个时间点开始播放到某个时间点”。要实现这个语义，你必须有一个统一的“媒体时间轴”概念。

**核心抽象：MediaSource（媒体源）**

它代表：
 “我可以按时间顺序给你提供一帧一帧的数据”。

内部可以再裂变为几类实现：

- `FileMediaSource`：从 mp4 文件读取
- `LiveMediaSource`：从实时采集（摄像头、推流）获取
- `TestMediaSource`：用于测试的假数据源（方便你阶段性验证）

MediaSource 一般要提供逻辑能力（不是代码，只说能力）：

- 初始化：打开文件/设备、读入头部、建立索引
- 按时间读帧：
  - 给定当前播放时间 → 返回下一帧的内容 + 时间戳信息
- 可选的随机跳转（seek）：
  - PLAY 的 `Range: npt=10-` 就需要此能力

**时间轴逻辑：**

- 媒体源内部自己维护“当前读到的时间位置”
- PLAY 请求如果没有 Range：
  - 从当前时间位置继续播放
- PLAY 请求如果带 Range（未来版本）：
  - 先请求媒体源 seek 到指定时间，再开始向外给帧

> 从结构上看：
>  **MediaSource 对“控制层”隐藏了文件格式、解复用等复杂度，只暴露“按时间拿帧”的接口。**
>  Session 持有一个 MediaSource 的“句柄”，PLAY 只是决定“是否开始从 MediaSource 取数据”。

------

### 4. RTP 发送调度子系统（PLAY = 为 Session 启动/恢复一个发送任务）

这是 PLAY 阶段的“数据平面大脑”：
 **怎么把“一个个媒体帧”，按时间节奏、打成 RTP 包、通过网络发出去。**

可以按责任，拆成几块逻辑：

#### 4.1 Sender / Scheduler（调度器）

负责：

- 对每个 Session，维护一个“发送任务”的生命周期：
  - 启动（PLAY）
  - 暂停（PAUSE）
  - 停止（TEARDOWN/超时）
- 控制节奏：
  - 例如“每隔 40ms 取一帧送出去”
  - 或更精确：根据每帧的 PTS、系统时钟对齐

你可以有一个概念上的组件，比如：

- `RtpSendController` 或 `StreamScheduler`

它与 Session 的关系：

- Session 请求“开始播放” → 调度器为该 Session 创建/激活一个“发送任务”
- Session 请求“暂停” → 调度器标记这个任务暂停或结束
- Session 销毁 → 调度器彻底回收对应任务资源

#### 4.2 Packetizer（打包器）

负责：

- 把 MediaSource 提供的一帧帧数据，转换为一个或多个 RTP 包：
  - 设置 RTP Header：版本号、PT、seq、timestamp、SSRC
  - 处理 NAL 分片（H264 时一个帧可能拆成多个 RTP 包）
- 把 Packetizer 和“序列号/时间戳”状态绑定在 Session 上

从逻辑上讲，一次发送循环可能是这样：

1. 向 MediaSource 请求一个“下一帧”
2. 把这一帧喂给 Packetizer → 得到若干 RTP 数据块
3. 对每个块，更新 seq / timestamp，设置 M 标志等
4. 把这些块交给 NetworkSender 去发

#### 4.3 NetworkSender（真正发到网络）

负责：

- 通过 UDP 或 TCP interleaved 把 RTP 数据发出去
- 不关心 RTSP 方法，只关心给定 buffer + 目标地址/连接，如何**高效、可靠地 send**

对 PLAY 阶段来说，它就是一个“黑盒”：
 **你给我要发的 RTP 包，我帮你送到对端。**

------

## 四、把一次 PLAY 的执行过程串成一条“时序线”

现在我们不看代码，只看“模块间的调用顺序”：

1. **客户端发送 PLAY 请求**
   - RTSP 连接收到数据 → 交给 Parser
   - Parser 解析为 `RtspRequest(method=PLAY, headers=...)`
2. **Dispatcher 把请求交给 PlayHandler**
   - `PlayHandler.onPlay(request, connection)`
3. **PlayHandler 与 SessionManager 交互**
   - 从请求读出 `Session` 头
   - `SessionManager` 查找对应 Session
   - 检查 Session 状态是否是 `READY` 或 `PAUSED`
   - 如不合法，构造错误响应返回
4. **合法时，PlayHandler 调用发送调度器**
   - 告诉调度器：“这个 Session 要进入 PLAYING 状态了”
   - 调度器为该 Session：
     - 创建或恢复一个“发送任务”
     - 标记 Session 的状态为 `PLAYING`
     - 把 Session 里携带的 MediaSource、Packetizer、NetworkSender 组合起来
5. **PlayHandler 构造 RTSP PLAY 响应**
   - 设置 `CSeq`、`Session`、可选 `RTP-Info` 等
   - 通过 Connection 返回给客户端
6. **调度器开始数据循环（独立于控制流）**
   - 循环：
     1. 从 Session 关联的 MediaSource 拉取下一帧
     2. 调 Packetizer 生成一个或多个 RTP 包
     3. 使用 NetworkSender 发给客户端
     4. 根据帧时间控制发送节奏（sleep / 定时器 / 时钟对齐）
   - 直到：
     - 收到 PAUSE/TEARDOWN → Session 状态变更 → 任务退出
     - 或媒体源数据耗尽 → 自行停止
7. **期间可能再收到新的RTSP请求**
   - 如 OPTIONS/GET_PARAMETER 保活，PAUSE，TEARDOWN 等
   - 控制流再次通过 Session / 调度器调整状态

> 注意：
>  **控制流（PlayHandler 部分）是“短生命周期的一次调用”，
>  数据流（发送任务）是“长生命周期的循环服务”**。
>  PLAY 就是控制流去“启动/恢复”一个数据流服务。

------

## 五、层级总结：从大到小的三层拆法

帮你回顾一下层次结构，方便你脑子里形成树状图：

### 层 1：宏观视角（两个面）

- 控制平面：RTSP 请求/响应
- 数据平面：RTP 包发送

### 层 2：四大子系统

1. RTSP 控制处理：连接、解析、分发、PLAY Handler
2. Session & 状态管理：Session 对象、会话生命周期、状态机
3. 媒体源 & 时间轴：MediaSource 抽象、时间定位、逐帧输出
4. RTP 发送调度：调度器、打包器、网络发送

### 层 3：PLAY 相关关键逻辑

- 播放前：
  - 通过 SETUP 建好 Session，填好传输参数、媒体源
- PLAY 处理：
  - 解析 → 找 Session → 校验状态 → 更新状态 → 通知调度器 → 回应
- 发送循环：
  - 调度器根据 Session 状态 & 媒体时间
  - 从媒体源取帧 → 打包成 RTP → 网络发送 → 控制节奏

------

如果你愿意，下一步我们可以：

- 只拿这套结构，画一张“文字版时序图 / 模块关系图”（比如用 Mermaid 状态图/时序图来表达），
- 或者把四个子系统拆成若干 `.h/.cpp` 文件名 + 类名（仍然保持不写具体实现）。

你可以选：

- 想先看“模块关系图”，
- 还是先看“文件 & 类的划分方案”。