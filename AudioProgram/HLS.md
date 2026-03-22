[TOC]

# HLS 

## checklists

- [ ] Playlist format
- [ ] EXT-X tag 规范
- [ ] Segment rules
- [ ] Encryption
- [ ]  Low Latency HLS
- [ ]  coding mp4 转码 h264  and split into ts+m3u8



## overview

HLS基于文件分发运作

break into 切片？ 每个切片包含 几秒到十几秒的视频。

使用 m3u8管理不同的分辨率信息

使用TS (transport stream) 文件打包视频片段，这些视频文件通常使用h264 AAC编码

客户端通过缓存部分ts文件，应对网络环境

通常使用CDN分发这些文件，提升用户体验

HLS 支持这两种使用场景。对于直播，播放列表文件会随着新片段的发布而持续更新。对于点播视频，播放列表会预先包含所有片段，观众可以跳转到任意位置观看。底层技术相同，**区别**在于播放列表的生成和更新方式。

## pros and cons

- 跨设备兼容性
- 支持多种视频格式，ts mp4 编码格式 H264 AAC
- 更好的用户体验。自适应码率
- 扩展性，基于http运行，
- 安全性：HLS 支持对视频片段进行 AES-128 加密并可结合基于令牌的身份验证来控制访问权限

cons:

优先靠考虑流的可靠性，而不是延迟

## use case

- 直播，实时通信

- 点播

## structure

Master Playlist (.m3u8)
        ↓
Variant Playlist
        ↓
Media Segments (.ts / fmp4)

## format

Master Playlist

```
#EXTM3U
#EXT-X-STREAM-INF:BANDWIDTH=800000
low/index.m3u8

#EXT-X-STREAM-INF:BANDWIDTH=1500000
mid/index.m3u8
```



Media Playlist

```
#EXTM3U
#EXT-X-TARGETDURATION:6

#EXTINF:6.0
segment1.ts

#EXTINF:6.0
segment2.ts
```