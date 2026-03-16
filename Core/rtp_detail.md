[TOC]

# [流媒体协议之RTP详解20170921](https://www.cnblogs.com/yuweifeng/p/7563265.html)

## 1.RTP介绍

　　实时传输协议RTP（Real-time Transport Protocol）是一个网络传输协议，它是由IETF的多媒体传输工作小组1996年在RFC 1889中公布的，后在RFC3550中进行更新。

　　国际电信联盟ITU-T也发布了自己的RTP文档，作为H.225.0，但是后来当IETF发布了关于它的稳定的标准RFC后就被取消了。它作为因特网标准在 [ RFC 3550 ] 有详细说明.

　　RTP协议详细说明了*在互联网上传递音频和视频的标准数据包格式*。它一开始被设计为一个多播协议，但后来被用在很多单播应用中。RTP协议常用于流媒体系统（配合RTSP协议），

视频会议和一键通（Push toTalk）系统（配合H.323或SIP），使它成为IP电话产业的技术基础。RTP协议和RTP控制协议RTCP一起使用，而且它是建立在用户数据报协议上的（UDP）。

 

## 2.RTP内容

　　RTP标准定义了两个子协议 ，RTP和RTCP

　　数据传输协议RTP，用于实时传输数据。该协议提供的信息包括：时间戳（用于同步）、序列号（用于丢包和重排序检测）、以及负载格式（用于说明数据的编码格式）。

　　控制协议RTCP，用于QoS反馈和同步媒体流。相对于RTP来说，RTCP所占的带宽非常小，通常只有5%。

 

## 3.RTP的使用

### 3.1.为什么要使用RTP

　　一提到流媒体传输、一谈到什么视频监控、视频会议、语音电话（VOIP），都离不开RTP协议的应用，但当大家都根据经验或者别人的应用而选择RTP协议的时候，你可曾想过，为什么我们要使用RTP来进行流媒体的传输呢？为什么我们一定要用RTP？难道TCP、UDP或者其他的网络协议不能达到我们的要求么？

　　像TCP这样的可靠传输协议，通过超时和重传机制来保证传输数据流中的每一个bit的正确性，但这样会使得无论从协议的实现还是传输的过程都变得非常的复杂。而且，当传输过程中有数据丢失的时候，由于对数据丢失的检测（超时检测）和重传，会数据流的传输被迫暂停和延时。

　　或许你会说，我们可以利用客户端构造一个足够大的缓冲区来保证显示的正常，这种方法对于从网络播放音视频来说是可以接受的，但是对于一些需要实时交互的场合（如视频聊天、视频会议等），如果这种缓冲超过了200ms，将会产生难以接受的实时性体验。

 

## 3.2.为什么RTP可以解决上述时延问题

　　RTP协议是一种基于UDP的传输协议，RTP本身并不能为按顺序传送数据包提供可靠的传送机制，也不提供流量控制或拥塞控制，它依靠RTCP提供这些服务。

这样，对于那些丢失的数据包，不存在由于超时检测而带来的延时，同时，对于那些丢弃的包，也可以由上层根据其重要性来选择性的重传。比如，对于I帧、P帧、B帧数据，由于其重要性依次降低，

故在网络状况不好的情况下，可以考虑在B帧丢失甚至P帧丢失的情况下不进行重传，这样，在客户端方面，虽然可能会有短暂的不清晰画面，但却保证了实时性的体验和要求。

 

## 4.RTP的协议层次

### 4.1.传输层的子层

下图给出了流媒体应用中的一个典型的协议体系结构。

 ![img](http://images2017.cnblogs.com/blog/964489/201709/964489-20170920192800603-45869816.png)

从图中可以看出，RTP被划分在传输层，它建立在UDP上。同UDP协议一样，为了实现其实时传输功能，RTP也有固定的封装形式。

RTP用来为端到端的实时传输提供时间信息和流同步，但并不保证服务质量。服务质量由RTCP来提供。

 

### 4.2.应用层的一部分

　　从应用开发者的角度看，RTP 应当是应用层的一部分。在应用的发送端，开发者必须编写用 RTP 封装分组的程序代码，然后把 RTP 分组交给 UDP 插口接口。在接收端，

RTP 分组通过 UDP 插口接口进入应用层后，还要利用开发者编写的程序代码从 RTP 分组中把应用数据块提取出来。

 

## 5.RTP报文

### 5.1.RTP分组

​     RTP分组只包含RTP数据，而控制是由RTCP协议提供。RTP在1025到65535之间选择一个未使用的偶数UDP端口号，而在同一次会话中的RTCP则使用下一个奇数UDP端口号。

端口号5004和5005分别用作RTP和RTCP的默认端口号。RTP分组的首部格式如图2所示，其中前12个字节是必须的。

![img](http://images2017.cnblogs.com/blog/964489/201709/964489-20170920194410493-679604437.png)

 

 

### 5.2.RTP首部

#### 5.2.1.RTP报文头格式（见RFC3550 Page12）：

1).版本号（V）：用来标志使用的RTP版本，占2位，当前协议版本号为2

2).填充位（P）：填充标志，占1位，如果P=1，则该RTP包的尾部就包含附加的填充字节，在该报文的尾部填充一个或多个额外的八位组，它们不是有效载荷的一部分。

3).扩展位（X）：扩展标志，占1位，如果X=1，则在RTP固定头部后面就跟有一个扩展头部

4).CSRC计数器（CC）：CSRC计数器，占4位，指示固定头部后面跟着的CSRC标识符的个数

5).标记位（M）：标记，占1位，该位的解释由配置文档（Profile）来承担，不同的有效载荷有不同的含义**，一般而言，对于视频，标记一帧的结束；对于音频，标记会话的开始**。  // (分包 )

6).载荷类型（PayloadType）： 有效荷载类型，占7位，用于说明RTP报文中有效载荷的类型，如GSM音频、JPEM图像等,在流媒体中大部分是用来区分音频流和视频流的，这样便于客户端进行解析。

7).序列号（SN）：占16位，用于标识发送者所发送的RTP报文的序列号，**每发送一个报文，序列号增1。接收端可以据此检测丢包和重建包序列**，当下层的承载协议用UDP的时候，网络状况不好的时候可以用来检查丢包，同时出现网络抖动的情况可以用来对数据进行重新排序，序列号的初始值是随机的，同时音频包和视频包的sequence是分别记数的。

8).时间戳(Timestamp): 占32位，记录了该包中数据的第一个字节的采样时刻。在一次会话开始时，时间戳初始化成一个初始值。即使在没有信号发送时，时间戳的数值也要随时间而不断地增加（时间在流逝嘛）。时钟频率依赖于负载数据格式，并在描述文件（profile）中进行描述。一般而言，必须使用90 kHz 时钟频率。接收者使用时戳来计算延迟和延迟抖动，并进行同步控制。 （增加依据时钟频率，而频率依据具体payload type）

9).同步源标识符(SSRC)：占32位，用于标识同步信源，同步源就是指RTP包流的来源。在同一个RTP会话中不能有两个相同的SSRC值。该标识符是随机选取的，RFC1889推荐了MD5随机算法。    (??)

10).贡献源列表（CSRC List）：可以有0～15个，每个CSRC标识符占32位，用来标志对一个RTP混合器产生的新包有贡献的所有RTP包的源。由混合器将这些有贡献的SSRC标识符插入表中。SSRC标识符都被列出来，以便接收端能正确指出交谈双方的身份。

 

#### 5.2.2.取一段码流如下：

80 e0 00 1e 00 00 d2 f0 00 00 00 00 41 9b 6b 49 €?....??....A?kI

e1 0f 26 53 02 1a ff06 59 97 1d d2 2e 8c 50 01 ?.&S....Y?.?.?P.

cc 13 ec 52 77 4e e50e 7b fd 16 11 66 27 7c b4 ?.?RwN?.{?..f'|?

f6 e1 29 d5 d6 a4 ef3e 12 d8 fd 6c 97 51 e7 e9 ??)????>.??l?Q??

cfc7 5e c8 a9 51 f6 82 65 d6 48 5a 86 b0 e0 8c ??^??Q??e?HZ????

其中，

80        是V_P_X_CC

e0        是M_PT

00 1e     是SequenceNum

00 00 d2 f0 是Timestamp

00 00 00 00是SSRC

把前两字节换成二进制如下

1000 0000 1110 0000

按顺序解释如下：

10        是V；

0         是P；

0         是X；

0000      是CC；

1         是M；

110 0000  是PT；

 

#### 5.2.3.RTP扩展头结构

 ![img](http://images2017.cnblogs.com/blog/964489/201709/964489-20170920193031415-2095007069.jpg)

​                     图 Rtp扩展头

 

​     若 RTP 固定头中的扩展比特位置1（注意：如果有CSRC列表，则在CSRC列表之后），

则一个长度可变的头扩展部分被加到 RTP 固定头之后。头扩展包含 16 比特的长度域，指示扩展项中 32 比特字的个数，不包括 4 个字节扩展头(因此零是有效值)。

　　RTP 固定头之后只允许有一个头扩展。为允许多个互操作实现独立生成不同的头扩展，或某种特定实现有多种不同的头扩展,扩展项的前 16 比特用以识别标识符或参数。**这 16 比特的格式由具体实现的上层协议定义。基本的 RTP 说明并不定义任何头扩展本身。**

 

### 5.3.RTP数据部分

#### 5.3.1.RTP荷载H264码流

##### 1).码流结构

RFC3984是H.264的baseline码流在RTP方式下传输的规范，

![img](http://images2017.cnblogs.com/blog/964489/201709/964489-20170920193127165-466495381.png)

​                                

##### 2).荷载结构

 ![img](http://images2017.cnblogs.com/blog/964489/201709/964489-20170920193145525-139321495.png)

荷载格式定义三个不同的基本荷载结构，接收者可以通过RTP荷载的第一个字节后5位（如上图）识别荷载结构。

I、 单个NAL单元包：荷载中只包含一个NAL单元。NAL头类型域等于原始 NAL单元类型,即在范围1到23之间

II、 聚合包：本类型用于聚合多个NAL单元到单个RTP荷载中。本包有四种版本,单时间聚合包类型A (STAP-A)，单时间聚合包类型B (STAP-B)，多时间聚合包类型(MTAP)16位位移(MTAP16), 多时间聚合包类型(MTAP)24位位移(MTAP24)。赋予STAP-A, STAP-B, MTAP16, MTAP24的NAL单元类型号分别是 24,25, 26, 27

III、分片单元：用于分片单个NAL单元到多个RTP包。现存两个版本FU-A，FU-B,用NAL单元类型 28，29标识

对三种荷载结构的原文描述如下：

Common Structure of the RTP Payload Format：

  The payload format defines three different basic payload structures.

  A receiver can identify the payload structure by the first byte of

  the RTP payload, which co-serves as the RTP payload header and, in

  some cases, as the first byte of the payload.  This byte is always

  structured as a NAL unit header.  The NAL unit type field indicates

  which structure is present.  The possible structures are as follows:

 

  Single NAL Unit Packet: Contains only a single NAL unit in the

  payload.  The NAL header type field will be equal to the original NAL

  unit type; i.e., in the range of 1 to 23, inclusive.  Specified in

  section 5.6.

 

  Aggregation packet: Packet type used to aggregate multiple NAL units

  into a single RTP payload.  This packet exists in four versions, the

  Single-Time Aggregation Packet type A (STAP-A), the Single-Time

  Aggregation Packet type B (STAP-B), Multi-Time Aggregation Packet

  (MTAP) with 16-bit offset (MTAP16), and Multi-Time Aggregation Packet

  (MTAP) with 24-bit offset (MTAP24).  The NAL unit type numbers

  assigned for STAP-A, STAP-B, MTAP16, and MTAP24 are 24, 25, 26, and

  27, respectively.  Specified in section 5.7.

 

  Fragmentation unit: Used to fragment a single NAL unit over multiple

  RTP packets.  Exists with two versions, FU-A and FU-B, identified

  with the NAL unit type numbers 28 and 29, respectively.  Specified in

  section 5.8.

  Table 1.  Summary of NAL unit types and their payload structures

 

   Type  Packet   Type name             Section

   \---------------------------------------------------------

   0    undefined                   -

   1-23  NAL unit  Single NAL unit packet per H.264  5.6

   24   STAP-A   Single-time aggregation packet   5.7.1

   25   STAP-B   Single-time aggregation packet   5.7.1

   26   MTAP16   Multi-time aggregation packet    5.7.2

   27   MTAP24   Multi-time aggregation packet    5.7.2

   28   FU-A    Fragmentation unit         5.8

   29   FU-B    Fragmentation unit         5.8

   30-31  undefined                   -

同时又有三种封包模式，每种封包模式支持不同的荷载结构(具体使用哪种封包模式可以充SDP会话中取得)，原文：

Packetization Modes

  This memo specifies three cases of packetization modes:

   o Single NAL unit mode

   o Non-interleaved mode

   o Interleaved mode

  The single NAL unit mode is targeted for conversational systems that

  comply with ITU-T Recommendation H.241 [15] (see section 12.1).  The

  non-interleaved mode is targeted for conversational systems that may

  not comply with ITU-T Recommendation H.241.  In the non-interleaved

  mode, NAL units are transmitted in NAL unit decoding order.  The

  interleaved mode is targeted for systems that do not require very low

  end-to-end latency.  The interleaved mode allows transmission of NAL

  units out of NAL unit decoding order.

 

  The packetization mode in use MAY be signaled by the value of the

  OPTIONAL packetization-mode MIME parameter or by external means.  The

  used packetization mode governs which NAL unit types are allowed in

  RTP payloads.  Table 3 summarizes the allowed NAL unit types for each

  packetization mode.  Some NAL unit type values (indicated as

  undefined in Table 3) are reserved for future extensions.  NAL units

  of those types SHOULD NOT be sent by a sender and MUST be ignored by

  a receiver.  For example, the Types 1-23, with the associated packet

  type "NAL unit", are allowed in "Single NAL Unit Mode" and in "Non-

  Interleaved Mode", but disallowed in "Interleaved Mode".

  Packetization modes are explained in more detail in section 6.

 

 

 

 

RFC 3984      RTP Payload Format for H.264 Video    February 2005

  Table 3.  Summary of allowed NAL unit types for each packetization

  mode (yes = allowed, no = disallowed, ig = ignore)

 

   Type  Packet   Single NAL   Non-Interleaved   Interleaved

​            Unit Mode      Mode       Mode

   \-------------------------------------------------------------

 

   0    undefined   ig        ig        ig

   1-23  NAL unit   yes        yes        no

   24   STAP-A     no        yes        no

   25   STAP-B     no        no        yes

   26   MTAP16     no        no        yes

   27   MTAP24     no        no        yes

   28   FU-A      no        yes        yes

   29   FU-B      no        no        yes

   30-31  undefined   ig        ig        ig

 

*常用的打包时的分包规则是：如果小于MTU采用单个NAL单元包，如果大于MTU就采用FUs分片方式。*

*因为常用的打包方式就是单个NAL包和FU-A方式*

 

##### 3).单个NAL包单元

定义在此的NAL单元包必须只包含一个。这意味聚合包和分片单元不可以用在单个NAL 单元包中。并且RTP序号必须符合NAL单元的解码顺序。NAL单元的第一字节和RTP荷载头第一个字节重合。

打包H264码流时，只需在帧前面加上12字节的RTP头即可，具体来说：

 

12字节的RTP头后面的就是音视频数据。一个封装单个NAL单元包到RTP的NAL单元流的RTP序号必须符合NAL单元的解码顺序。

对于 NALU 的长度小于 MTU 大小的包, 一般采用单一 NAL 单元模式.

对于一个原始的 H.264 NALU 单元常由[Start Code] [NALU Header] [NALU Payload]三部分组成, 其中 Start Code 用于标示这是一个 NALU 单元的开始, 必须是 "00 00 00 01" 或 "00 00 01", NALU 头仅一个字节, 其后都是 NALU 单元内容.

打包时去除 "00 00 01" 或 "00 00 00 01" 的开始码, 把其他数据封包的 RTP 包即可.

 

  0          1          2          3

  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1

 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

 |F|NRI| type  |                        |

 +-+-+-+-+-+-+-+-+                        |

 |                                |

 |        Bytes 2..n of a Single NAL unit         |

 |                                |

 |                +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

 |                :...OPTIONAL RTP padding    |

 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

如有一个 H.264 的 NALU 是这样的:

[00 00 00 01 67 42 A0 1E 23 56 0E 2F ... ]

 

这是一个序列参数集 NAL 单元. [00 00 00 01] 是四个字节的开始码, 67 是 NALU 头, 42 开始的数据是 NALU 内容.

封装成 RTP 包将如下:

[ RTP Header ] [ 67 42 A0 1E 23 56 0E 2F ]

即只要去掉 4 个字节的开始码就可以了.

 

##### 4).分片单元（FU-A）

数据比较大的H264视频包，被RTP分片发送。12字节的RTP头后面跟随的就是FU-A分片：

当 NALU 的长度超过 MTU 时, 就必须对 NALU 单元进行分片封包. 也称为 Fragmentation Units (FUs).

 

  0          1          2          3

  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1

 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

 | FU indicator |  FU header |                |

 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                |

 |                                |

 |             FU payload              |

 |                                |

 |                +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

 |                :...OPTIONAL RTP padding    |

 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

 Figure 14. RTP payload format for FU-A

#### I、FU indicator有以下格式：

 

 +---------------+

 |0|1|2|3|4|5|6|7|

 +-+-+-+-+-+-+-+-+

 |F|NRI| Type  |

 +---------------+

FU指示字节的类型域Type=28表示FU-A。。NRI域的值必须根据分片NAL单元的NRI域的值设置。

NAL单元荷载类型定义见下表

表1. 单元类型以及荷载结构总结

 

 .Type  Packet   Type name            

 \---------------------------------------------------------

 0   undefined                  -

 1-23  NAL unit  Single NAL unit packet per H.264 

 24   STAP-A   Single-time aggregation packet  

 25   STAP-B   Single-time aggregation packet  

 26   MTAP16  Multi-time aggregation packet  

 27   MTAP24  Multi-time aggregation packet  

 28   FU-A   Fragmentation unit        

 29   FU-B   Fragmentation unit        

 30-31 undefined     

​           

#### II、FU header的格式如下：

 

 +---------------+

 |0|1|2|3|4|5|6|7|

 +-+-+-+-+-+-+-+-+

 |S|E|R| Type  |

 +---------------+

S: 1 bit 当设置成1,开始位指示分片NAL单元的开始。当跟随的FU荷载不是分片NAL单元荷载的开始，开始位设为0。

E: 1 bit 当设置成1, 结束位指示分片NAL单元的结束，即, 荷载的最后字节也是分片NAL单元的最后一个字节。 当跟随的FU荷载不是分片NAL单元的最后分片,结束位设置为0。

R: 1 bit 保留位必须设置为0，接收者必须忽略该位。

Type: 5 bits

 

#### III、打包和解包

打包：当编码器在编码时需要将原有一个NAL按照FU-A进行分片，原有的NAL的单元头与分片后的FU-A的单元头有如下关系：

原始的NAL头的前三位为FU indicator的前三位，原始的NAL头的后五位为FU header的后五位，

FU indicator与FU header的剩余位数根据实际情况决定。

 

解包：当接收端收到FU-A的分片数据，需要将所有的分片包组合还原成原始的NAl包时，FU-A的单元头与还原后的NAL的关系如下：

还原后的NAL头的八位是由FU indicator的前三位加FU header的后五位组成，即：

nal_unit_type = (fu_indicator & 0xe0) | (fu_header & 0x1f)

 

#### IV、取一段码流分析如下：

80 60 01 0f 00 0e 10 00 00 0000 00 7c 85 88 82 €`..........|???

00 0a 7f ca 94 05 3b7f 3e 7f fe 14 2b 27 26 f8 ...??.;.>.?.+'&?

89 88 dd 85 62 e1 6dfc 33 01 38 1a 10 35 f2 14 ????b?m?3.8..5?.

84 6e 21 24 8f 72 62f0 51 7e 10 5f 0d 42 71 12 ?n!$?rb?Q~._.Bq.

17 65 62 a1 f1 44 dc df 4b 4a 38 aa 96 b7 dd 24 .eb??D??KJ8????$

前12字节是RTP Header

7c是FU indicator

85是FU Header

FU indicator（0x7C）和FU Header（0x85）换成二进制如下

0111 1100 1000 0101

按顺序解析如下：

0              是F

11             是NRI

11100       是FU Type，这里是28，即FU-A

1              是S，Start，说明是分片的第一包

0              是E，End，如果是分片的最后一包，设置为1，这里不是

0              是R，Remain，保留位，总是0

00101      是NAl Type，这里是5，说明是关键帧（不知道为什么是关键帧请自行谷歌）

 

打包时，FUindicator的F、NRI是NAL Header中的F、NRI，Type是28；FU Header的S、E、R分别按照分片起始位置设置，Type是NAL Header中的Type。

解包时，取FU indicator的前三位和FU Header的后五位，即0110 0101（0x65）为NAL类型。

 

#### V、注意

分片只定义于单个NAL单元不用于任何聚合包。NAL单元的一个分片由整数个连续NAL单元字节组成。每个NAL单元字节必须正好是该NAL单元一个分片的一部分。

相同NAL单元的分片必须使用递增的RTP序号连续顺序发送(第一和最后分片之间没有其他的RTP包）。相似，NAL单元必须按照RTP顺序号的顺序装配。

  当一个NAL单元被分片运送在分片单元(FUs)中时，被引用为分片NAL单元。STAPs,MTAPs不可以被分片。 FUs不可以嵌套。

即, 一个FU 不可以包含另一个FU。运送FU的RTP时戳被设置成分片NAL单元的NALU时刻。

 

5).组合封包模式

当 NALU 的长度特别小时, 可以把几个 NALU 单元封在一个 RTP 包中.



##### 上述总结，by gpt

 **RTP 传输 H.264 的基本思路是：**
 **小 NALU 直接发，大 NALU 拆成多个 FU-A 包发送。**
 **每个 FU-A 包在 RTP 层保持顺序，接收端根据 S/E 标志重组出完整的 H.264 帧。**

注: 另外一篇生成的关于rtp负载h264的md在本地

#### 5.3.2.RTP荷载PS流

针对H264 做如下PS 封装：每个IDR NALU 前一般都会包含SPS、PPS 等NALU，因此将SPS、PPS、IDR 的NALU 封装为一个PS 包，包括ps 头，然后加上PS system header，PS system map，PES header+h264 raw data。

所以一个IDR NALU PS 包由外到内顺序是：PSheader| PS system header | PS system Map | PES header | h264 raw data。对于其它非关键帧的PS 包，就简单多了，直接加上PS头和PES 头就可以了。

顺序为：PS header | PES header | h264raw data。以上是对只有视频video 的情况，如果要把音频Audio也打包进PS 封装，也可以。

当有音频数据时，将数据加上PES header 放到视频PES 后就可以了。顺序如下：PS 包=PS头|PES(video)|PES(audio)，再用RTP 封装发送就可以了。

​    

GB28181 对RTP 传输的数据负载类型有规定（参考GB28181 附录B），负载类型中96-127，RFC2250 建议96 表示PS 封装，建议97 为MPEG-4，建议98 为H264

即我们接收到的RTP 包首先需要判断负载类型，若负载类型为96，则采用PS 解复用，将音视频分开解码。若负载类型为98，直接按照H264 的解码类型解码。

注：此方法不一定准确，取决于打包格式是否标准

PS 包中的流类型（stream type）的取值如下：

a、    MPEG-4 视频流： 0x10；

b、    H.264 视频流： 0x1B；

c、    SVAC 视频流： 0x80；

d、    G.711 音频流： 0x90；

e、    G.722.1 音频流： 0x92；

f、    G.723.1 音频流： 0x93；

g、    G.729 音频流： 0x99；

h、    SVAC音频流： 0x9B。

##### 1).PS包头

 ![img](http://images2017.cnblogs.com/blog/964489/201709/964489-20170920193337493-762551641.png)

​                        　　　　　　 图

I、    Pack start code：包起始码字段，值为0x000001BA的位串，用来标志一个包的开始。

II、    System clock reference base，system clock reference extenstion：系统时钟参考字段。

III、   Pack stuffing length ：包填充长度字段，3 位整数，规定该字段后填充字节的个数

80 60 53 1f 00 94 89 00 00 0000 00 00 00 01 ba €`S..??........?

7e ff 3e fb 44 01 00 5f 6b f8 00 00 01 e0 14 53 ~.>?D.._k?...?.S

80 80 05 2f bf cf bed1 1c 42 56 7b 13 58 0a 1e €€./????.BV{.X..

08 b1 4f 33 69 35 0453 6d 33 a8 04 15 58 d9 21 .?O3i5.Sm3?..X?!

9741 b9 f1 75 3d 94 2b 1f bc 0b b2 b4 97 bf 93 ?A??u=?+.?.?????

前12位是RTP Header，这里不再赘述；

000001ba是包头起始码；

接下来的9位包括了SCR，SCRE，MUXRate，具体看上图

最后一位是保留位（0xf8），定义了是否有扩展，二进制如下

1111 1000

前5位跳过，后3位指示了扩展长度，这里是0.

##### 2).系统标题

 ![img](http://images2017.cnblogs.com/blog/964489/201709/964489-20170920193354900-464070450.png)

   　　　                   　　　　　　　　图

Systemheader当且仅当pack是第一个数据包时才存在，即PS包头之后就是系统标题。取值0x000001BB的位串，指出系统标题的开始，暂时不需要处理，读取Header Length直接跳过即可。

##### 3).节目映射流

Systemheader当且仅当pack是第一个数据包时才存在，即系统标题之后就是节目流映射。取值0x000001BC的位串，指出节目流映射的开始，暂时不需要处理，读取Header Length直接跳过即可。前5字节的结构同系统标题，见上图。

 

取一段码流分析系统标题和节目映射流

00 00 01 ba 45 a9 d4 5c 34 0100 5f 6b f8 00 00 ...?E??\4.._k?..

01 bb 00 0c 80 cc f5 04 e1 7f e0 e0 e8 c0 c0 20 .?..€??.?.?????

00 00 01 bc 00 1e e1 ff00 00 00 18 1b e0 00 0c ...?..?......?..

2a 0a 7f ff 00 00 0708 1f fe a0 5a 90 c0 00 00 *........??Z??..

00 00 00 00 00 00 01 e0 7f e0 80 80 0521 6a 75 .......?.?€€.!ju

前14个字节是PS包头（注意，没有扩展）；

接下来的00 00 01 bb是系统标题起始码；

接下来的00 0c说明了系统标题的长度（不包括起始码和长度字节本身）；

接下来的12个字节是系统标题的具体内容，这里不做解析；

继续看到00 00 01 bc，这是节目映射流起始码；

紧接着的00 1e同样代表长度；

跳过e1 ff，基本没用；

接下来是00 18，代表基本流长度，说明了后面还有24个字节；

接下来的1b，意思是H264编码格式；

下一个字节e0，意思是视频流；

接下里00 0c，同样代表接下的长度12个字节；

跳过这12个字节，看到90，这是G.711音频格式；

下一个字节是c0，代表音频流；

接下来的00 00同样代表长度，这里是0；

接下来4个字节是CRC，循环冗余校验。

到这里节目映射流解析完毕。

 

##### 4).PES分组头部

 ![img](http://images2017.cnblogs.com/blog/964489/201709/964489-20170920193427540-1891958457.png)

​                     　　　　　　　 图

别被这么长的图吓到，其实原理相同，但是，你必须处理其中的每一位。

1) Packet start code prefix：值为0x000001的位串，它和后面的stream id 构成了标识分组开始的分组起始码，用来标志一个包的开始。
2) Stream id：在节目流中，它规定了基本流的号码和类型。0x(C0~DF)指音频，0x(E0~EF)为视频
3) PES packet length：16 位字段，指出了PES 分组中跟在该字段后的字节数目。值为0 表示PES 分组长度要么没有规定要么没有限制。这种情况只允许出现在有效负载包含来源于传输流分组中某个视频基本流的字节的PES 分组中。
4) PTS_DTS：2 位字段。当值为'10'时，PTS 字段应出现在PES 分组标题中；当值为'11'时，PTS 字段和DTS 字段都应出现在PES 分组标题中；当值为'00'时，PTS 字段和DTS 字段都不出现在PES分组标题中。值'01'是不允许的。
5) ESCR：1位。置'1'时表示ESCR 基础和扩展字段出现在PES 分组标题中；值为'0'表示没有ESCR 字段。
6) ESrate：1 位。置'1'时表示ES rate 字段出现在PES 分组标题中；值为'0'表示没有ES rate 字段。
7) DSMtrick mode：1 位。置'1'时表示有8 位特技方式字段；值为'0'表示没有该字段。
8) Additionalinfo：1 位。附加版权信息标志字段。置'1'时表示有附加拷贝信息字段；值为'0'表示没有该字段。
9) CRC：1 位。置'1'时表示CRC 字段出现在PES 分组标题中；值为'0'表示没有该字段。
10) Extensionflag：1 位标志。置'1'时表示PES 分组标题中有扩展字段；值为'0'表示没有该字段。

PES header data length： 8 位。PES 标题数据长度字段。指出包含在PES 分组标题中的可选字段和任何填充字节所占用的总字节数。该字段之前的字节指出了有无可选字段。

 

老规矩，上码流：

00 00 01 e0 21 33 80 80 05 2b 5f df 5c 95 71 84 ...?!3€€.+_?\?q?

aa e4 e9 e9 ec 40 cc17 e0 68 7b 23 f6 89 df 90 ?????@?.?h{#????

a9d4 be 74 b9 67 ad 34 6d f0 92 0d 5a 48 dd 13 ???t?g?4m??.ZH?.

 

00 00 01是起始码；

e0是视频流；

21 33 是帧长度；

接下来的两个80 80见下面的二进制解析；

下一个字节05指出了可选字段的长度，前一字节指出了有无可选字段；

接下来的5字节是PTS；

第7、8字节的二进制如下：

1000 0000 1000 0000

按顺序解析：

第7个字节：

10             是标志位，必须是10；

00             是加扰控制字段，‘00’表示没有加密，剩下的01,10,11由用户自定义；

0              是优先级，1为高，0为低；

0              是数据对齐指示字段；

0              是版权字段；

0              是原始或拷贝字段。置'1'时表示相关PES分组有效负载的内容是原始的；'0'表示内容是一份拷贝；

第8个字节：

10             是PTS_DTS字段，这里是10，表示有PTS,没有DTS；

0              是ESCR标志字段，这里为0，表示没有该段；

0              是ES速率标志字段，，这里为0，表示没有该段；

0              是DSM特技方式标志字段，，这里为0，表示没有该段；

0              是附加版权信息标志字段，，这里为0，表示没有该段；

0              是PESCRC标志字段，，这里为0，表示没有该段；

0              是PES扩展标志字段，，这里为0，表示没有该段；

本段码流只有PTS，贴一下解析函数

[cpp] view plain copy

unsigned long parse_time_stamp (const unsigned char *p) 

{ 

  unsigned long b; 

  //共33位，溢出后从0开始 

  unsigned long val; 

 

  //第1个字节的第5、6、7位 

  b = *p++; 

  val = (b & 0x0e) << 29; 

 

  //第2个字节的8位和第3个字节的前7位 

  b = (*(p++)) << 8; 

  b += *(p++); 

  val += ((b & 0xfffe) << 14); 

 

  //第4个字节的8位和第5个字节的前7位 

  b = (*(p++)) << 8; 

  b += *(p++); 

  val += ((b & 0xfffe) >> 1); 

 

  return val; 

} 

其他字段可参考协议解析

ps：

遇到00 00 01 bd的，这个是私有流的标识

 

## 6.RTP的会话过程

当应用程序建立一个RTP会话时，应用程序将确定一对目的传输地址。目的传输地址由一个网络地址和一对端口组成，有两个端口：

一个给RTP包，一个给RTCP包，使得RTP/RTCP数据能够正确发送。RTP数据发向偶数的UDP端口，而对应的控制信号RTCP数据发向相邻的奇数UDP端口（偶数的UDP端口＋1），这样就构成一个UDP端口对。

 

RTP的发送过程如下，接收过程则相反。

  RTP协议从上层接收流媒体信息码流（如H.264），封装成RTP数据包；RTCP从上层接收控制信息，封装成RTCP控制包。

  RTP将RTP 数据包发往UDP端口对中偶数端口；RTCP将RTCP控制包发往UDP端口对中的接收端口，即奇数端口。

 人话：分配偶数端口给RTP （+1）端口给RTCP

## 7.RTP的profile机制

​    RTP为具体的应用提供了非常大的灵活性，它将传输协议与具体的应用环境、具体的控制策略分开，传输协议本身只提供完成实时传输的机制，

开发者可以根据不同的应用环境，自主选择合适的配置环境、以及合适的控制策略。

​    这里所说的控制策略指的是你可以根据自己特定的应用需求，来实现特定的一些RTCP控制算法，比如前面提到的丢包的检测算法、丢包的重传策略、一些视频会议应用中的控制方案等等（这些策略我可能将在后续的文章中进行描述）。

​    对于上面说的合适的配置环境，主要是指RTP的相关配置和负载格式的定义。RTP协议为了广泛地支持各种多媒体格式（如 H.264, MPEG-4, MJPEG, MPEG），没有在协议中体现出具体的应用配置，而是通过profile配置文件以及负载类型格式说明文件的形式来提供。

对于任何一种特定的应用，RTP定义了一个profile文件以及相关的负载格式说明，相关的文件如下所示：

《RTP Profile for Audio and Video Conferences with Minimal Control》（RFC3551）

《RTP Payload Format for H.264 Video》（RFC3984）

《RTP Payload Format for MPEG-4 Audio/Visual Streams》（RFC3016）

等等，想了解更多可以点击这里：http://en.wikipedia.org/wiki/RTP_audio_video_profile

 

说明：如果应用程序不使用专有的方案来提供有效载荷类型(payload type)、顺序号或者时间戳，而是使用标准的RTP协议，应用程序就更容易与其他的网络应用程序配合运行，

这是大家都希望的事情。例如，如果有两个不同的公司都在开发因特网电话软件，他们都把RTP合并到他们的产品中，这样就有希望：使用不同公司电话软件的用户之间能够进行通信。

 

## 8.RTCP的主要功能

服务质量的监视与反馈、媒体间的同步，以及多播组中成员的标识。在RTP会话期 间，各参与者周期性地传送RTCP包。RTCP包中含有已发送的数据包的数量、丢失的数据包的数量等统计资料，

因此，各参与者可以利用这些信息动态地改变传输速率，甚至改变有效载荷类型。RTP和RTCP配合使用，它们能以有效的反馈和最小的开销使传输效率最佳化，因而特别适合传送网上的实时数据。

 

## 9.RTCP的使用

RTCP 控制协议需要与RTP数据协议一起配合使用，当应用程序启动一个RTP会话时将同时占用两个端口，分别供RTP 和RTCP使用。RTP本身并不能为按序传输数据包提供可靠的保证，

也不提供流量控制和拥塞控制，这些都由RTCP来负责完成。通常RTCP会采用与 RTP相同的分发机制，向会话中的所有成员周期性地发送控制信息，应用程序通过接收这些数据，

从中获取会话参与者的相关资料，以及网络状况、分组丢失概率等反馈信息，从而能够对服务质量进行控制或者对网络状况进行诊断。

 

## 10.RTCP报文

RTCP也是用UDP来传送的，但RTCP封装的仅仅是一些控制信息，因而分组很短，所以可以将多个RTCP分组封装在一个UDP包中。RTCP有如下五种分组类型。

| **类型** | **缩写表示**                     | **用途**   |
| -------- | -------------------------------- | ---------- |
| 200      | SR（Sender Report）              | 发送端报告 |
| 201      | RR（Receiver Report）            | 接收端报告 |
| 202      | SDES（Source Description Items） | 源点描述   |
| 203      | BYE                              | 结束传输   |
| 204      | . APP                            | 特定应用   |

 

上述五种分组的封装大同小异，下面只讲述SR类型，而其它类型请参考RFC3550。

 

​    发送端报告分组SR（Sender Report）用来使发送端以多播方式向所有接收端报告发送情况。SR分组的主要内容有：相应的RTP流的SSRC，RTP流中最新产生的RTP分组的时间戳和NTP，RTP流包含的分组数，RTP流包含的字节数。

SR包的封装如下图所示。

 ![img](http://images2017.cnblogs.com/blog/964489/201709/964489-20170920193526962-827123352.png)

版本（V）：同RTP包头域。

填充（P）：同RTP包头域。

接收报告计数器（RC）：5比特，该SR包中的接收报告块的数目，可以为零。

包类型（PT）：8比特，SR包是200。

长度域（Length）：16比特，其中存放的是该SR包以32比特为单位的总长度减一。

同步源（SSRC）：SR包发送者的同步源标识符。与对应RTP包中的SSRC一样。

NTP Timestamp（Network time protocol）：SR包发送时的绝对时间值。NTP的作用是同步不同的RTP媒体流。

RTP Timestamp：与NTP时间戳对应，与RTP数据包中的RTP时间戳具有相同的单位和随机初始值。

Sender’s packet count：从开始发送包到产生这个SR包这段时间里，发送者发送的RTP数据包的总数. SSRC改变时，这个域清零。

Sender`s octet count：从开始发送包到产生这个SR包这段时间里，发送者发送的净荷数据的总字节数（不包括头部和填充）。发送者改变其SSRC时，这个域要清零。

同步源n的SSRC标识符：该报告块中包含的是从该源接收到的包的统计信息。

丢失率（Fraction Lost）：表明从上一个SR或RR包发出以来从同步源n(SSRC_n)来的RTP数据包的丢失率。

累计的包丢失数目：从开始接收到SSRC_n的包到发送SR,从SSRC_n传过来的RTP数据包的丢失总数。

收到的扩展最大序列号：从SSRC_n收到的RTP数据包中最大的序列号，

接收抖动（Interarrival jitter）：RTP数据包接受时间的统计方差估计 上次SR时间戳（Last SR,LSR）：取最近从SSRC_n收到的SR包中的NTP时间戳的中间32比特。如果目前还没收到SR包，则该域清零。

上次SR以来的延时（Delay since last SR,DLSR）：上次从SSRC_n收到SR包到发送本报告的延时。

 

RTCP数据报携带有服务质量监控的必要信息，能够对服务质量进行动态的调整，并能够对网络拥塞进行有效的控制。由于RTCP数据报采用的是多播方式，

因此会话中的所有成员都可以通过RTCP数据报返回的控制信息，来了解其他参与者的当前情况。

 

在一个典型的应用场合下，发送媒体流的应用程序将周期性地产生发送端报告SR，该RTCP数据报含有不同媒体流间的同步信息，以及已经发送的数据报和字节的计数，

接收端根据这些信息可以估计出实际的数据传输速率。另一方面，接收端会向所有已知的发送端发送接收端报告RR，

该RTCP数据报含有已接收数据报的最大序列号、丢失的数据报数目、延时抖动和时间戳等重要信息，发送端应用根据这些信息可以估计出往返时延，

并且可以根据数据报丢失概率和时延抖动情况动态调整发送速率，以改善网络拥塞状况，或者根据网络状况平滑地调整应用程序的服务质量。

 

## 11.代码描述

(  具体看gpt生成的

