### 关于connection

Connection 是“对一个建立后的 TCP 连接的抽象封装”，用于承载该连接上的所有 I/O 状态、缓冲区、协议状态机，并向 epoll 注册其 I/O 事件。



class Connection {
public:
    void onReadable();
    void onWritable();
    void close();
private:
    int fd;
    Buffer input;
    Buffer output;
};

epoll 不“处理” fd，它只是“通知你事件”，
 真正的行为发生在 Connection 类的回调里：
 `onReadable()`
 `onWritable()`

为什么要 buffer？

因为 TCP 是流（没有消息边界），一次 read() 不一定刚好读完一个 RTSP 请求。

所以 input buffer 就是 “累积请求数据，到足够解析为止”。

output buffer 是 “write() 写不完时继续写”。

一句话总结：

> **Connection 是“事件驱动框架”与“协议状态机”之间的桥梁。**

epoll 不知道你的协议是什么。
 Parser 不知道你的 I/O 如何调度。
 Dispatcher 不知道你的 socket 状态如何管理。

Connection 把这些粘合起来。



1. 管 fd 的生命周期
2. 管输入输出缓冲
3. 在 epoll 事件驱动下调度 onReadable/onWritable
4. 驱动协议解析、分发
5. 管理连接状态、超时、RTP 通道等（未来）