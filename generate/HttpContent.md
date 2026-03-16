下面是对 `HttpContext` 的详细分析，从**数据结构设计**、**方法提供**、以及**实现机制（状态机解析过程）**三方面说明。

------

## 一、总体功能定位

`HttpContext` 是一个 **HTTP 请求解析上下文类**，作用是接收原始的 HTTP 报文数据（如从 socket 读取的 `char*` 缓冲区），通过内部的**有限状态机（Finite State Machine, FSM）**逐步解析出：

- 请求行（Method、URL、Protocol、Version）
- URL 参数（key=value）
- 请求头（Header）
- 请求体（Body）

最终生成一个完整的 `HttpRequest` 对象。

------

## 二、引入的数据结构与对象关系

### 1️⃣ 主体类结构

```cpp
class HttpContext {
private:
    std::unique_ptr<HttpRequest> request_;  // 指向被解析的HTTP请求对象
    HttpRequestParaseState state_;          // 当前解析状态（枚举）
};
```

- **`request_`**
   用 `std::unique_ptr` 管理的智能指针，指向一个 `HttpRequest` 对象，表示正在解析的 HTTP 请求。
   ✅ 优点：
  - 避免手动释放内存；
  - 表明 HttpContext 对该请求对象拥有唯一所有权。
- **`state_`**
   保存当前解析阶段。
   类型为 `HttpRequestParaseState`，定义在 `HttpContext.h` 中的枚举类型。
   这个枚举定义了完整的解析流程（START → METHOD → URL → HEADER → BODY → COMPLETE）。

### 2️⃣ 枚举状态机结构

```cpp
enum HttpRequestParaseState {
    START, METHOD, BEFORE_URL, IN_URL,
    BEFORE_URL_PARAM_KEY, URL_PARAM_KEY, BEFORE_URL_PARAM_VALUE, URL_PARAM_VALUE,
    BEFORE_PROTOCOL, PROTOCOL, BEFORE_VERSION, VERSION,
    HEADER_KEY, HEADER_VALUE,
    WHEN_CR, CR_LF, CR_LF_CR, BODY, COMPLETE,
    kINVALID, ...
};
```

这是一个**HTTP 请求解析状态机的所有阶段**。
 其核心思想是：

> 按顺序从请求行 → URL 参数 → 协议 → Header → Body 逐步解析，每读一个字符根据状态转移。

------

## 三、对外提供的方法

| 方法名                                            | 功能                        | 特点                      |
| ------------------------------------------------- | --------------------------- | ------------------------- |
| `bool ParaseRequest(const char* begin, int size)` | 解析 HTTP 报文内容          | 主函数，实现状态机逻辑    |
| `bool GetCompleteRequest()`                       | 判断是否解析完成            | 返回 `state_ == COMPLETE` |
| `HttpRequest* request()`                          | 获取内部 `HttpRequest` 指针 | 提供给上层获取结果        |
| `void ResetContextStatus()`                       | 重置解析状态                | 为下一次请求复用上下文    |

------

## 四、实现机制：有限状态机解析流程

`ParaseRequest()` 是整个类的核心逻辑。

```cpp
bool HttpContext::ParaseRequest(const char *begin, int size)
```

其本质是一个 **字符流驱动的状态机**：

```cpp
while (state_ != kINVALID && state_ != COMPLETE && end - begin <= size) {
    char ch = *end;
    switch (state_) {
        case START: ...
        case METHOD: ...
        case IN_URL: ...
        case HEADER_KEY: ...
        case BODY: ...
    }
    end++;
}
```

逐个字符扫描，根据当前状态和输入字符类型（空格、回车、冒号、?、& 等）切换状态并填充数据。

------

### 🔹 1. 请求行解析

状态序列：

```
START → METHOD → BEFORE_URL → IN_URL → BEFORE_PROTOCOL → PROTOCOL → BEFORE_VERSION → VERSION
```

对应逻辑：

- 解析 HTTP 方法（如 GET、POST）
- 提取 URL（可能含参数）
- 解析协议（HTTP）
- 提取版本号（1.1 / 1.0）

示例：

```
GET /index.html?user=abc HTTP/1.1
```

------

### 🔹 2. URL 参数解析

状态序列：

```
IN_URL → BEFORE_URL_PARAM_KEY → URL_PARAM_KEY → BEFORE_URL_PARAM_VALUE → URL_PARAM_VALUE
```

当检测到 `'?'` 进入参数区，每遇到 `'&'` 视为一个新的 `key=value` 对。
 解析后写入：

```cpp
request_->SetRequestParams(key, value);
```

------

### 🔹 3. Header 解析

状态序列：

```
CR_LF → HEADER_KEY → HEADER_VALUE → WHEN_CR → CR_LF → ...
```

每行形如：

```
Host: localhost
Content-Length: 50
```

解析时：

- `':'` 分隔键和值；
- `\r\n` 表示一行结束；
- 空行 `\r\n\r\n` 表示 Header 结束，进入 Body 阶段。

调用：

```cpp
request_->AddHeader(key, value);
```

------

### 🔹 4. Body 解析

状态序列：

```
CR_LF_CR → BODY → COMPLETE
```

通过 `Content-Length` 判断 Body 长度：

```cpp
if (request_->headers().count("Content-Lenght")) {
    if (atoi(request_->GetHeader("Content-Lenght").c_str()) > 0)
        state_ = BODY;
}
```

最后：

```cpp
request_->SetBody(std::string(start, start + bodylength));
```

------

### 🔹 5. 错误与终止状态

若在任意阶段出现非法字符（例如 URL 中空格、Header 缺少冒号），则：

```cpp
state_ = kINVALID;
```

解析终止，返回 false。

------

## 五、整体数据流与控制流总结图

```text
                ┌──────────────────────┐
                │  HttpContext::Parase │
                └──────────┬───────────┘
                           │
                  +--------▼---------+
                  |  状态机 HttpRequestParaseState |
                  +------------------+
                           │
   ┌────────────┬──────────┼──────────┬──────────────┬────────────┐
   │ START→METHOD │ URL参数 │ 协议版本 │ Header键值对 │ Body内容 │
   └────────────┴──────────┴──────────┴──────────────┴────────────┘
                           │
                ┌──────────▼─────────┐
                │ HttpRequest 填充完成 │
                └────────────────────┘
```

------

## 六、技术要点总结

| 模块                  | 技术点                                    | 说明                     |
| --------------------- | ----------------------------------------- | ------------------------ |
| **智能指针**          | `std::unique_ptr<HttpRequest>`            | 自动内存管理             |
| **有限状态机（FSM）** | 枚举 + switch                             | 解析 HTTP 报文           |
| **字符串切片**        | `std::string(start, end)`                 | 从指针区间提取字符串     |
| **数据结构分层**      | HttpContext（控制） + HttpRequest（存储） | 解耦解析逻辑与存储结构   |
| **健壮性设计**        | 状态检查 + `kINVALID`                     | 避免非法请求破坏解析流程 |

------

## ✅ 总结一句话

`HttpContext` 使用一个**有限状态机（FSM）+ 智能指针管理的 HttpRequest 对象**，
 实现了从原始字节流逐字符解析出完整 HTTP 请求的过程。
 它封装了解析状态与请求对象，使上层 Reactor/Connection 模块在读取数据后，只需调用：

```cpp
if (ctx.ParaseRequest(buf, len) && ctx.GetCompleteRequest()) {
    handle(ctx.request());
}
```

即可得到一个结构化的 `HttpRequest` 对象。