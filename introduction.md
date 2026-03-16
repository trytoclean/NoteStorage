 speification

 （书接上文：

关于重构的一些建议：

循序渐进，将修改计划改成一系列小步骤，例如将冗长代码拆分成辅助函数时，一次只引入一次新函数，测试是否正常工作

暂时注释旧代码，写新代码，与旧的比较

完成后再删掉旧的，提交（commit

长度问题：暂不考虑

）

关于规范：

### introduction

团队合作的基石，是caller 和 callee的规范，有了规范，callee在不通知caller的基础上更改实现，

### 行为等效性

​	不管callee对函数使用哪种实现，caller在不需要知道底层实现下，不同的底层实现应返回相同的结果。

​	使用规范（specification）规定caller和callee的行为。规范必须回答：客户端可以依赖什么行为，caller能获得预期的结果。相反如果没有规范，代码中精确的规范可以帮助你确定责任归属。

eg

```c++
int find(vector<int>& arr, int val) {
    for (int i = 0; i < arr.size(); i++)
        if (arr[i] == val)
            return i;
    return -1;
}

int find(vector<int>& arr, int val) {
    int pos = -1;
    for (int i = 0; i < arr.size(); i++)
        if (arr[i] == val)
            pos = i;
    return pos;
}
```

二者都是访问数组，返回值的下表，但是区别在于，是否考虑到数组有重复的值，以及对有重复值做出什么样的回应：返回第一次出现，还是只要找到就OK。如果caller指明要其中一个，那么另外一个的实现是错误的。而且没有可参考的规范，caller如果在使用实现复杂的函数时将无从下手。

规范对模块的客户端来说非常有用，因为它能帮助用户更轻松地理解模块。

### 前置条件vs后置条件

> 前置条件：函数调用之前必须成立的条件。（调用方保证

eg

```TS
function sqrt(x: number): number
requires x >= 0
// 调用之前必须保证x>=0
```

前置条件可以是参数范围，参数关系，状态条件

> 后置条件 函数执行完成后必须成立的条件。（实现方保证）

```ts
function abs(x: number): number
effects:
    return >= 0
    return == x or return == -x
```

后置条件可以是：返回值关系 异常行为 状态修改（mutation）

### 规范结构

一般来说，一个函数的规范由几个部分组成，

函数签名（函数名称，参数，返回值）

要求：描述对参数的附加值

效果：描述函数的返回值，异常，



extra： 文档规范，参考Google code style ， 
	另外一个是 Doxcygen。自动生成html文档

### 规范不应包含函数的实现

### 避免空值

在一些场合不要使用空值

从抽象角度来看，空值不包含任何有效信息（空值说明了什么），而且空值会有传递性 ，即程序会在某个位置崩溃，如

```
函数B 忘记检查
函数C dereference
程序崩溃
函数A 返回 null
root cause != crash location
```

从函数角度来看，User findUser(id) 如果允许返回null,状态会变成

```
user not found
database error
permission denied
timeout
uninitialized 
这是一种状态混叠（state conflation)
```

现代软件工程有几种规范和替代方案

**每种状态应该有明确的表示。**

使用安全的处理机制 如 std::optional<T> 要求caller显示处理空值

(c++23) std::expected<T,E> 当失败时错误（可能会有更好的错误处理）

(C++) contract programming



### 测试单元

我们应该独立地为程序的每个模块编写测试。一个好的单元测试应该只关注一个规范。

- 我们编写的某个函数的单元测试不应该因为*另一个*函数不满足其规范而失败。

- 好的*集成测试*使用模块组合的测试可以确保不同函数的规范兼容：

- 不同函数的调用者和实现者传递和返回的值都符合彼此的预期。

- 集成测试不能取代系统设计的单元测试。

###  所有测试必须遵循规范

我们无法测试违反前提条件时的行为：例如，我们无法检查函数在遇到非法输入时是快速失败还是返回垃圾值，因为**所有测试都必须遵循规范** 。如果前提条件是实现者可以合理检查的，那么通常可以修改规范：移除前提条件，并在后置条件中指定行为。

### 如果规范没有说可以修改输入，那么默认不能修改

```ts
//pure function
function toLowerCase(arr:string[]):string[] {
    return arr.map(x => x.toLowerCase())
}
//caller
a = ["HELLO","WORLD"]
b = toLowerCase(a)
//callee return
a = ["HELLO","WORLD"]
b = ["hello","world"]
//side-effect
function toLowerCase(arr:string[]):string[] {
    for(let i=0;i<arr.length;i++){
        arr[i] = arr[i].toLowerCase()
    }
    return arr
}
//caller:
a = ["HELLO","WORLD"]
b = toLowerCase(a)
//callee
a = ["hello","world"]
b = ["hello","world"]

/**
 * Converts strings to lowercase.
 *
 * @param arr array of strings
 * @returns new array with lowercase strings
 */
```

在这个例子中，a的状态被悄悄修改，这会导致，隐藏状态发生变化。这样会导致程序推理的复杂度变高，调试苦难，并发问题等等。

所以如果mutation是函数功能的一部分，就应该在specification中指出（例子如上）

最终实现 Make side effects explicit

（我认为exception 处理可以单独在开一章，甚至在项目中再次应用）

