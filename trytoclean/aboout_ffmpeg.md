```cp
/*
    说明：
    DS： AvFormatContext
   包含大量信息，贯彻始终。（额外注意，应该将这些提供的结构都设定为指针。）
    fmt包含需要的 streams <-视频的流信息，数组
                    codecpar  <-视频的编码器信息，
    Method：
    处理流自身：
    打开：avformat_open_input
    关闭: avformat_close_input
    处理流：
        1）读取：avformat_find_stream_info
   读取一部分媒体数据包推断每个流的真实参数
        2）调试：av_dump_format 打印详细信息，用于调试用，支持input or output
        3)
*/

/*
为什么不在 avformat_open_input() 里完成？
因为：
很多容器没有把所有 metadata 写在 header
有些编码格式参数必须在 bitstream（Packet）里解析，比如 H264 SPS/PPS
有些文件 duration 只能通过数帧数来推断
因此需要第二阶段扫描。
*/
```

