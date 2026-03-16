## so so important 

### 项目流程：

```
┌─────────────────────────────────────────────────────┐
│                     Server.start()                  │
└──────────────┬──────────────────────────────────────┘
               │ 创建 listen fd
               │ 设置非阻塞
               ▼
        ┌────────────┐
        │ epoll_ctl   │  ← 注册 listen_fd / EPOLLIN
        └──────┬─────┘
               ▼
       ┌────────────────┐
       │ epoll_wait     │  ← 阻塞等待事件
       └───────┬────────┘
               ▼
   ┌─────────────────────────┐
   │ event = new connection? │ listen_fd?
   └───────────────┬────────┘
                   │ yes
                   ▼
        accept() 新连接
        创建 Connection(fd)
        epoll_ctl(ADD, fd, EPOLLIN | ...)

                   │ no
                   ▼
   ┌─────────────────────────────────────┐
   │ event for some connection fd        │
   └──────────┬─────────┬──────────────┘
              │         │
        EPOLLIN     EPOLLOUT
              │         │
              ▼         ▼
  conn->onReadable()   conn->onWritable()
              │         │
              ▼         ▼
   read 数据 → input buffer  
              │
              ▼
       Parser 解析  
              │
              ▼
         Dispatcher 分发
              │
              ▼
         Handler 生成响应
              │
              ▼
      Builder 构造 RTSP 响应
              │
              ▼
     写入 output buffer  
              │
              ▼
 epoll_ctl(MOD, fd, EPOLLOUT) —— 让 epoll 回调 onWritable()
              │
              ▼
 在 onWritable() 中写出数据  
 若 output 写空 → epoll_ctl(MOD, fd, EPOLLIN)

IO 事件由 epoll 触发 → Connection 执行读写 → 解析 → 分发 → 构造响应 → 写回 → epoll 再次调度下一轮。

```



```
        [创建阶段]
Server.accept() → new Connection(fd)
Connection.set_nonblocking()
epoll_ctl(ADD)

            │
            ▼

        [工作阶段]
   多次 onReadable/onWritable
   Buffer 多次读写
   Parser 多次解析
   Dispatcher 多次调用逻辑
   Handler 多次生成响应

            │
            ▼

        [结束条件触发]
   - 客户端关闭（read=0）
   - read/write 错误
   - 服务器主动关闭
   - RTSP TEARDOWN 指令

            │
            ▼

        [关闭阶段]
epoll_ctl(DEL)
::close(fd)
释放 Connection 对象


connection 对象的生命周期
```



### 对象职责：

#### net模块

**connection**

解耦的关键，connection通过回调去处理逻辑（比如写入）。结果是上层rtsp看不到下层net，反过来也是。如onReadable onWriteable



​	需要buffer的原因是，tcp不维护报文边界，是流的协议，所以会出现，read 和 write 不完全，会有多余的数据在操作系统的tcp buffer里，结果就是 读到的数据不完整，写入的数据没写完。	

**epoll** 通知事件，不处理fd

> 真正的行为发生在 Connection 类的回调里：
> `onReadable()`
>`onWritable()`

所以，Connection 是“事件驱动框架”与“协议状态机”之间的桥梁。

协议状态机：RTSP state machine 

事件驱动框架： 