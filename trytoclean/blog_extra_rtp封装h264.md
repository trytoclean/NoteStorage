这篇博客主要讲了 **H.264 视频数据在 RTP 协议中是怎样被封装、分片并传输的全过程**，核心内容可以分为 **传输结构 → 打包模式 → 单个 NALU 传输 → 大帧分片传输（FU-A）** 四个阶段。以下是总结与解释：

------

## 一、整体概览：RTP 如何传输 H.264

H.264 编码输出的是一系列 **NALU（Network Abstraction Layer Unit）单元**。每个 NALU 代表视频数据的一部分（如 SPS、PPS、I 帧、P 帧等）。

在 RTP 传输中：

```
RTP 包 = [RTP Header(12字节)] + [RTP Payload(H.264数据)]
```

而 **RTP Payload** 的组织方式由 RFC3984 规范定义，有三种：

1. **单个 NAL 单元包**（Single NAL Unit）
    → 一个 RTP 包只装一个 NALU，最简单。
2. **聚合包（STAP/MTAP）**
    → 多个小 NALU 合并进一个 RTP 包中。
3. **分片单元包（FU-A / FU-B）**
    → 一个大 NALU 拆分成多个 RTP 包。

------

## 二、RTP 载荷结构（Payload Structure）

RFC3984 定义的类型号如下：

| 类型号 | 含义          | 名称            | 用途            |
| ------ | ------------- | --------------- | --------------- |
| 1–23   | 单个 NAL 单元 | Single NAL      | 小于 MTU 的情况 |
| 24     | 聚合包        | STAP-A          | 多 NAL 合并     |
| 25     | 聚合包        | STAP-B          |                 |
| 26–27  | 多时间聚合包  | MTAP16 / MTAP24 |                 |
| 28     | 分片单元      | FU-A            | 大帧分片        |
| 29     | 分片单元      | FU-B            |                 |
| 30–31  | 未定义        | —               | 保留            |

**常见规则：**

- 若 `NALU size < MTU` → 使用单 NALU 包。
- 若 `NALU size > MTU` → 使用 FU-A 分片方式。

------

## 三、单个 NAL 单元的 RTP 封装

### 1. 原始 NALU 格式：

```
[Start Code][NAL Header][NAL Payload]
Start Code = 00 00 01 或 00 00 00 01
```

### 2. 打包步骤：

- 去掉 Start Code；
- 在前面加上 12 字节 RTP Header；
- NAL Header（1字节）直接作为 RTP Payload 的首字节。

### 3. 示例：

原始 H.264：

```
00 00 00 01 67 42 A0 1E 23 56 0E ...
```

去掉开始码 →

```
[RTP Header][67 42 A0 1E 23 56 0E ...]
```

这种包直接携带一个完整 NALU，不需要分片。

------

## 四、大 NALU 的分片传输：FU-A 机制

当 NALU 过大超过 MTU（如 1400 字节）时，采用 **FU-A** 分片机制。
 每个 RTP 包携带该 NALU 的一部分。

------

### 1. RTP Payload 结构：

```
| RTP Header(12B) | FU Indicator(1B) | FU Header(1B) | FU Payload |
```

------

### 2. FU Indicator 字节（1字节）

| 位     | 名称 | 含义                   |
| ------ | ---- | ---------------------- |
| bit7   | F    | 禁止位                 |
| bit6-5 | NRI  | 重要性等级             |
| bit4-0 | Type | 固定为 28（表示 FU-A） |

示例：`0x7C = 0111 1100`
 → F=0, NRI=11, Type=11100(28)

------

### 3. FU Header 字节（1字节）

| 位     | 名称 | 含义                          |
| ------ | ---- | ----------------------------- |
| bit7   | S    | 起始位（1表示第一个分片）     |
| bit6   | E    | 结束位（1表示最后一个分片）   |
| bit5   | R    | 保留位，恒为0                 |
| bit4-0 | Type | 原始 NALU 的类型（如5=IDR帧） |

示例：`0x85 = 1000 0101`
 → S=1, E=0, R=0, Type=5（关键帧）

------

### 4. 打包与解包逻辑

**打包：**

- 原 NAL 头的前三位 → FU Indicator 的前三位；
- 原 NAL 头的后五位 → FU Header 的后五位；
- Type=28 表示 FU-A；
- 每个分片都按顺序递增 RTP sequence number。

**解包：**

- 接收端收到所有分片后，

  ```
  NAL Header = (FU_indicator & 0xE0) | (FU_header & 0x1F)
  ```

- 然后按序拼接所有 FU Payload。

------

### 5. 分片规则与限制

- 同一 NALU 的分片必须连续编号；
- 不允许嵌套分片（FU 不可包含 FU）；
- STAP/MTAP 不可再分片；
- RTP 时间戳（timestamp）相同，表示来自同一帧。

------

## 五、总结：H.264 → RTP 封装流程

| 阶段           | 数据结构             | 描述                                            |
| -------------- | -------------------- | ----------------------------------------------- |
| **编码输出**   | H.264 NALUs          | 由编码器生成的若干 NAL 单元                     |
| **传输判断**   | 比较 NALU 长度与 MTU | 小则单包，大则分片                              |
| **打包方式1**  | 单 NAL 单元包        | RTP Header + NAL Header + Payload               |
| **打包方式2**  | FU-A 分片包          | RTP Header + FU Indicator + FU Header + Payload |
| **接收端重组** | 组合 FU-A → NAL      | 根据 S/E 标志恢复完整 NALU                      |
| **解码**       | H.264 解码器         | 还原视频帧数据                                  |

------

### 🔹可视化理解

```
原始大帧（NALU）：
┌────────────────────────────────────────────┐
│ NAL Header │  NAL Payload(>MTU)           │
└────────────────────────────────────────────┘

→ 分片为多个 RTP 包：

Packet1: RTP + [FU-A Start] + 第一段数据(S=1, E=0)
Packet2: RTP + [FU-A Middle] + 中间数据(S=0, E=0)
Packet3: RTP + [FU-A End] + 最后一段数据(S=0, E=1)
```

接收端：

```
合并所有 FU Payload → 重组 NALU → 交给解码器
```

------

✅ **总结一句话：**

> RTP 传输 H.264 的基本思路是：
>  小 NALU 直接发，大 NALU 拆成多个 FU-A 包发送。
>  每个 FU-A 包在 RTP 层保持顺序，接收端根据 S/E 标志重组出完整的 H.264 帧。