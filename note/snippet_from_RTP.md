## snippet_for_RTP



## 经过一轮“调查”之后，应该能回答，协议的应用场景，定义，报文字段及其含义，最终能基本实现语义





```
provides end-to-end delivery services for data with real-time
   characteristics, such as interactive audio and video.
```

```
Those services
   include payload type identification, sequence numbering, timestamping
   and delivery monitoring. 
```



```
Applications typically run RTP on top of
   UDP to make use of its multiplexing and checksum service
```



```
RTP supports data transfer to
   multiple destinations using multicast distribution if provided by the
   underlying network.
```



```
Note that RTP itself does not provide any mechanism to ensure timely
   delivery or provide other quality-of-service guarantees, but relies
   on lower-layer services to do so.
```

```
but sequence numbers might also be used to
   determine the proper location of a packet, for example in video
   decoding, without necessarily decoding packets in sequence.
```

```
the RTP control protocol (RTCP), to monitor the quality of service
      and to convey information about the participants in an on-going
      session.
```



```
a profile specification document, which defines a set of payload
      type codes and their mapping to payload formats (e.g., media
      encodings).  
```

   In   these examples, RTP is carried on top of IP and UDP, and follows the
   conventions established by the **profile** for audio and video specified
   in the companion RFC 3551.

scenarios(略) 

​	只需知道RTP 不可靠传输，RTCP负责反馈通信的QOS即可

“我只管你已经知道往哪发、怎么发实时媒体。”

保证播放连续性，而不是保证字节完整性

恢复“时间体验”，而不是恢复“原始数据顺序”

一些定义：(我个人暂时理解为，RTP的*组成*)

RTP media type: An RTP media type is the collection of payload
      types which can be carried within a single RTP session.  **The RTP  Profile** assigns RTP media types to RTP payload types.

**RTP session**：指一组使用 RTP 进行通信的参与者之间形成的一种关联关系。一个参与者可以同时参与多个 RTP 会话。

参与者通过接收使用**不同目的传输地址对**的会话来区分多个 RTP 会话。这里的一个传输地址对由：

- 一个网络地址
- 一对端口号（一个用于 RTP，一个用于 RTCP）
   组成。

RTP 会话的一个**关键区分特征**在于：
 **每一个 RTP 会话都维护一个完全独立的 SSRC 标识符空间**（见下文定义）。

一个 RTP 会话所包含的参与者集合，是指那些**能够接收到该会话中任意一个参与者发送的 SSRC 标识符**的参与者。这个 SSRC 标识符可以：

- 出现在 RTP 包中的 SSRC 字段
- 出现在 RTP 包中的 CSRC 字段（下文定义）
- 或出现在 RTCP 包中

**同步源（SSRC）**：指 RTP 数据包流的来源，通过一个 **32 位的数值型 SSRC 标识符**来标识，该标识符携带在 RTP 头部中，从而避免依赖网络地址。

SSRC 标识符是**随机选择的数值**，其目标是在某一个特定的 RTP 会话内保持**全局唯一性**

rule:  如果一个参与者在同一个 RTP 会话中生成了**多个媒体流**（例如来自多个摄像头的视频流），那么**每一个流都必须使用不同的 SSRC 标识符**进行标识。

the mixerMixer（混音器 / 混合器）

> **Mixer**：一种中间系统，它从一个或多个源接收 RTP 数据包，可能会改变数据格式，以某种方式将这些数据包进行组合，然后再转发生成的新的 RTP 数据包。

------

> 由于来自多个输入源的时间关系通常并不同步，混音器会在各个流之间进行**时间调整**，并为合成后的数据流**生成自己的时间基准**。

------

> 因此，所有由混音器产生的数据包，都会被标识为**以混音器自身作为同步源（SSRC）**。

End system: An application that generates the content to be sent
      in RTP packets and/or consumes the content of received RTP
      packets.  An end system can act as one or more synchronization
      sources in a particular RTP session, but typically only one.





贡献源（CSRC）**：指对某一个 RTP 混音器生成的**合成数据流**做出贡献的 RTP 数据包流来源。

extra:  为了提供一个**可用的服务**，在 RTP 之外**可能还需要的协议和机制**。

> 特别是在**多媒体会议**场景中，可能需要一个**控制协议**来分发组播地址和用于加密的密钥，协商所使用的*加密算法*，并为那些**没有预定义负载类型值**的格式，定义 **RTP 负载类型值与其所代表的负载格式之间的动态映射关系**。
>
> eg SDP

**Byte Order, Alignment, and Time Format:**

network byte order ： big endian

allgnment: 32

walltime: real time 

> A program might run for 6 minutes (wall time), but only use 0.04 seconds of CPU time, meaning it spent most of its duration waiting for something else, say a network response. 

绝对时间（即真实世界中的日期和时间）使用 **网络时间协议（NTP）** 的时间戳格式来表示，该时间戳以 **1900 年 1 月 1 日 0 点（UTC）** 作为起点，用秒数进行计数。完整精度的 NTP 时间戳是一个 **64 位无符号定点数**，其中：

- 前 32 位表示**整数部分（秒）**
- 后 32 位表示**小数部分（秒的分数）**

> RTP 用 NTP 的时间戳格式来表示“绝对时间”，但它并不强制依赖 NTP 协议；在实际使用中，RTP 关心的不是“几点几分”，而是“两个时间点相差多久”，因此时间回绕问题在可控范围内是无关紧要的。







Q&A

**为什么 RTP 选择 NTP 作为绝对时间表示？**

不是因为 RTP 要“做时间同步协议”，而是因为：

- NTP 是一个**成熟、标准化、跨平台**的时间表示

- 64 位固定格式，解析简单

- 能表达绝对时间，也能表达高精度差值

**那什么时候“绝对时间”才重要**?**跨主机同步时**，例如：

- 音频来自主机 A
- 视频来自主机 B
- 要求口型同步（lip-sync）

此时：

- RTP timestamp 本身不够
- 需要通过 **RTCP SR** 把：
  - RTP timestamp
  - NTP wallclock time
     绑定在一起

---

RTP 报文：

header:

RTP Fixed Header Fields

The interpretation of the marker is defined by a profile.  It is
      intended to allow significant events such as frame boundaries to
      be marked in the packet stream.   语义由 profile定义， 允许定义在一个流中帧的边界

An RTP source MAY change the payload type during a session, but this
field SHOULD NOT be used for multiplexing separate media streams

```
 A receiver MUST ignore packets with payload types that it does not
      understand.
     识别不了的类型属于丢弃行为
     
```
seq number: 
  作用and may be used by the receiver to detect packet loss and to
      restore packet sequence.

机制： The sequence number increments by one for each RTP data packet sent. 

2）The initial value of the sequence number  SHOULD be random (unpredictable)

SSRC:

机制：SHOULD be chosen randomly

```
规定
two
      synchronization sources within the same RTP session will have the
      same SSRC identifier.


If a source changes its source transport
      address, it must also choose a new SSRC identifier to avoid being
      interpreted as a looped source
```

CSRC list: 0 to 15 items, 32 bits each

he CSRC list identifies the contributing sources for the payload
      contained in this packet

CSRC identifiers are inserted by
      mixers

he number of identifiers is given by
      the CC field.



总结

version:

paddling:

marker:

CSRC_number:

Ex:

payload_type:

seq:

SSRC:

(一个机制：proflle 它定义了负载类型，等等。 不知道CSRC items是什么)

RTP会话复用：

ach medium
   SHOULD be carried in a separate RTP session with its own destination
   transport address.

 Separate audio and video streams SHOULD NOT be carried in a single
   RTP session and demultiplexed based on the payload type or SSRC
   fields. 

关于extension:

貌似是为了支持，没有在payload type 定义好的，需要ex来说明的，

```
This RTP specification does
   not define any header extensions itself.
   文档没有定义extension
```

```
If the X bit in the RTP header is one, a variable-length header
   extension MUST be appended to the RTP header, following the CSRC list
   if present. 
```



----

### 临时总结

文档

​	作用：用于实时数据的传输

​	文档定义：RTP自身，RTCP（用于检测反馈服务质量）

​	header: 字段<=>定义 



---

RTP 实现

```cpp
// 结构描述
//$对齐操作
//足以应对 h264和AAC的各种情况
struct header{
  int VPXCC; //version 2bit, paddling1,  externsion1 ,  CCounter 4bits
  int MPT; // Marker bit1 bit , payload type7 bit
  int seq_num; // 12bit 每次发送自增一
  int timestamp;// 32bit NTP （chrono or POSIX API
  int SSRC ;//32 bit 
  int CSRC; //optional,if CC is above zero
};
//问题一：payload type 依赖于profile ,那么profile从哪来？怎样应用？
// helper function:
int get_npt_time();//npt: since 1900-1-1 => timestamp
int SSRC(); // get from profile or get from random?
//what the hell is CSRC ,if needed , how to acquire?

```

