下面给你一份真正 **可执行、可落地、工程师友好** 的

# 🔥 六个月「边做项目边补基础」时间表

适合你的目标方向（音视频 / 网络 / 系统层 C++ 工程师）。

这比“照本宣科的课程表”更强，因为它基于你**正在做的项目**来驱动基础学习，而不是先啃厚厚的书。

------

# 🧭 总览

整个计划的总逻辑：

1. **主线项目**：Mini-RTSP/RTP Server（越做越完整）
2. **基础补充**：只补你因为项目卡住的理论
3. **每周循环**：
   - 写代码
   - 遇问题
   - 查理论
   - 修复
   - 深化
4. **六个月完成 3 个可展示的作品**（大厂面试够用）

------

# 🗓 整体时间安排（6 个月）

| 月份 | 主目标               | 项目进度                          | 需补基础            |
| ---- | -------------------- | --------------------------------- | ------------------- |
| 1    | 搭建基础框架         | Mini Echo Server + 单线程 epoll   | OS + 网络基础       |
| 2    | Reactor 框架         | Mini-Muduo 风格                   | IO 模型 + TCP       |
| 3    | RTSP 协议解析        | OPTIONS / DESCRIBE / SETUP / PLAY | HTTP/RTSP/状态机    |
| 4    | RTP/H264 推流        | RTP over UDP + H264 NAL 解析      | RTP/RTCP/H264       |
| 5    | 多线程优化           | 线程池 + 定时器 + 性能优化        | 锁/缓存一致性       |
| 6    | 整合、打磨、面试准备 | 完整可运行 RTSP Server            | 全面复盘 + 系统设计 |

------

# 🎯 第一月（第 1–4 周）

# ⭐ 目标：能跑 + 理解 OS 网络基础

# ⭐ 产出：Mini Echo Server（单线程 epoll）

------

## 📅 Week 1

### 任务（项目）

- 写一个 Blocking TCP Echo Server
- 只用 socket / bind / listen / accept / recv/send
- 不拆分类结构，能跑即可

### 遇到的点 → 需要补基础

- TCP 三次握手
- 阻塞/非阻塞 I/O
- 内核 socket 缓冲区
- 文件描述符是什么
- send/recv 的返回值机制

### 结果

你能把“网络 I/O”作为一个真实系统看待，而不是书本知识。

------

## 📅 Week 2

### 项目

- 把 server 改成 **非阻塞 + epoll (LT)**
- 引入 epoll_create/epoll_ctl/epoll_wait
- 能处理多客户端（简单 echo）

### 需要补的基础

- epoll LT 原理
- 水平触发行为
- 队列模型
- accept 的语义（惊群）

------

## 📅 Week 3

### 项目

- 从 procedural 改成类结构：
  - EventLoop
  - TcpConnection
  - Socket 封装
- 加入 Buffer 类（环形队列 or vector）

### 需要补基础

- C++ 内存管理
- RAII
- 智能指针（shared_ptr 的控制块与引用计数）
- copy/move 语义

------

## 📅 Week 4

### 项目

- 完成“单线程 Reactor”工作流
- Echo Server 可稳定处理并发 1000+
- 初步日志模块（printf → spdlog）

### 需要补基础

- Reactor vs Proactor
- 系统调用成本
- 多路复用与网络模型历代演进

------

# 🎯 第二月（第 5–8 周）

# ⭐ 目标：写出简化版 Muduo（支持多线程）

# ⭐ 产出：Mini-Reactor 框架（支持线程池）

------

## 📅 Week 5

### 项目

- 增加 TimerQueue（小根堆）
- 定时触发 callback（如定时广播）

### 补基础

- 最小堆
- 时间复杂度
- Linux timerfd（可选）
- 时钟漂移

------

## 📅 Week 6

### 项目

- 加入线程池（Task Queue）
- 主线程负责 accept
- 子线程负责 connection I/O
- 使用 eventfd 做线程间唤醒

### 补基础

- 线程调度
- 线程安全
- 锁与条件变量
- eventfd 原理

------

## 📅 Week 7

### 项目

- 引入连接回调（onMessage/onWriteDone）
- 完善 Buffer
- 加入优雅关闭（half close）

### 补基础

- 半关闭（FIN/ACK）
- 粘包拆包
- 状态机解析

------

## 📅 Week 8

### 项目

- 压测（使用 wrk 或自写压力工具）
- 优化（无锁结构、减少系统调用次数）

### 补基础

- 零拷贝 sendfile
- cache line false sharing
- 内存对齐与 cache 友好性

------

# 🎯 第三月（第 9–12 周）

# ⭐ 目标：完整 RTSP 协议解析

# ⭐ 产出：Mini RTSP Parser + Session 管理

------

## 📅 Week 9

### 项目

- 实现 OPTIONS
- 实现 DESCRIBE（返回 sdp）
- 用字符串手写 parser（后面可优化）

### 补基础

- HTTP 文本协议结构
- RTSP 状态机
- SDP 格式

------

## 📅 Week 10

### 项目

- 完成 SETUP（UDP/TCP Interleaved）
- 管理 RTP Port 分配（UDP）

### 补基础

- TCP 交错传输
- NAT/防火墙的 UDP 孔洞
- UDP socket sendto 行为

------

## 📅 Week 11

### 项目

- 实现 PLAY/PAUSE/TEARDOWN
- 引入 Session 类
- 分配 SSRC

### 补基础

- SSRC 冲突
- RTSP 底层状态机
- RTP Session 概念

------

## 📅 Week 12

### 项目

- 实现完整 RTSP Parser（带 header 与字段）
- 整理成可复用模块

### 补基础

- 正则状态机
- 无锁结构（读多写少）

------

# 🎯 第四月（第 13–16 周）

# ⭐ 目标：实现 RTP over UDP 推流

# ⭐ 产出：H264 解析 + RTP Packetizer

------

## 📅 Week 13

### 项目

- 读取 H264 文件
- 解析 NALU，判断类型（SPS/PPS/IDR）

### 补基础

- Annex-B 格式
- NALU Header
- start code 探测

------

## 📅 Week 14

### 项目

- RTP Header 构造
- FU-A 切片
- 时间戳生成（90kHz）

### 补基础

- RTP 协议栈
- 大端小端（htonl）

------

## 📅 Week 15

### 项目

- RTP over UDP 发包
- Wireshark 抓包校验

### 补基础

- MTU
- 分片重组
- UDP 拥塞

------

## 📅 Week 16

### 项目

- RTCP Receiver Report + Sender Report
- 校验同步

### 补基础

- RTCP 作用
- 抖动计算公式
- NTP 时间戳

------

# 🎯 第五月（第 17–20 周）

# ⭐ 目标：多线程优化 + 高性能

# ⭐ 产出：更接近工业级的传输模块

------

## 📅 Week 17–18

### 项目

- 线程池增强
- 生产者消费者队列
- RingBuffer 做 jitter buffer

### 补基础

- CAS
- 内存模型
- ABA 问题（可选）

------

## 📅 Week 19–20

### 项目

- Zero Copy（减少 memcpy）
- 使用 mmap 做文件读取
- 改善延迟抖动

### 补基础

- 内核页缓存
- copy-on-write
- 边界对齐

------

# 🎯 第六月（第 21–24 周）

# ⭐ 目标：整合、优化、准备面试

# ⭐ 产出：一个真正可用的 RTSP Server（可展示作品集）

------

## 📅 Week 21

### 项目

- 项目重构
- 清晰的模块目录结构
- README + 流程图 + API 文档

------

## 📅 Week 22

### 项目

- 性能压测
- 加日志、指标监控
- 可用性测试（丢包、网络抖动）

------

## 📅 Week 23

### 面试准备

- 整理知识体系：
  - C++
  - epoll
  - TCP
  - RTP/RTCP
  - H264
- 刷高频题
- 系统设计：如何做一个直播推流器？如何做 RTP jitter buffer？

------

## 📅 Week 24

### 项目最终版

- 上传 GitHub
- 配置 Docker
- 准备用于面试 demo
- 自我 Mock 面试（我可以帮你模拟）

------

# 🎉 最终结果

六个月后你将拥有：

- 🔧 可运行的底层网络框架
- 🎥 RTP/H264 推流引擎
- 📡 完整 RTSP Server
- 🔨 对网络、系统、C++ 底层的深刻理解
- 📚 面试必备理论
- 🧰 一套大厂级作品集
- 🔥 拿得出手的面试经验

大厂音视频/网络/系统岗位的面试官会非常认可这样的成长路径。

------

# 想继续下一步吗？

我可以帮你继续：

### ✔ 每个月拆成“每周执行 checklist”

### ✔ 给你项目目录结构模板

### ✔ 给你 RTSP Server 第一阶段的代码骨架

### ✔ 给你面试题库

### ✔ 给你每周一次的复盘模板

你想先看哪一个？