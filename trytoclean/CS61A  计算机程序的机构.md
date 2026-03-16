# CS61A  计算机程序的机构 



### day one

**函数的绑定，调用，执行**

​	首先在python中，不应该有赋值这一说法，应该叫绑定，

例如 

``````python
x=1
a,b=3,4

from operator import mul:
def square(x):
      return mul(x,x)
square(-2)

``````

其中，a,b=3,4 蕴含一个拆包，其代码等价于：

```python
arr[1]={3,4}
a=arr[0]
b=arr[1]
# from operator import mul:
```

注释一行是导包，导入mul这个函数。

---

**纯函数vs非纯函数：**

纯函数，只是返回值；非纯函数，具有side effects

l例如 abs(2) 返回2的绝对值， print(abs(2)) 打印2到控制台上，返回值是None。 且 side effect 不是一个值，他是一切调用函数的结果导致的行为。再比如：

``````python
from operator import mul;
print(print(1),print(2))
1
2
None None
'''
1
2
None None
		解释:这是因为 print() 函数本身返回 None，但会首先把它的参数打印到标准输出。所以先看到 1 和 2 分别从内部的 print() 输出，然后是最外层的 print(None, None)
		函数调用设计一个入栈出栈的过程。
'''

``````

**环境**

​	每当函数被调用时，解释器都会创建一个新的本地栈帧（local frame），用于存储该函数的参数和局部变量。这个新栈帧会与其所属的上层环境（通常是调用它的环境）建立连接。
 在变量查找时，解释器会首先在当前栈帧中寻找对应的名称；如果未找到，则会沿着父环境链向上传递，直到查找到全局栈帧（global frame）或抛出错误。
 	整个函数调用过程可以理解为“入栈”与“出栈”的操作：调用函数时，新的栈帧被压入调用栈中；当函数执行完毕并返回结果时，该栈帧被弹出，控制权回到调用点。

一些python的操作:

``````python
# 1. python cli的基本使用: python3 profile.py   执行
# 2. python3 -i ... 以命令行的形式使用，特点是定义的函数加入到环境中，可以方便测试与交互
# 3. python3 -m doctest -v profile.py  
# 使用 doctest 模块，将解析注释里的参数，然后调用函数并自动测试，返回执行情况。-v 选项是将正确测试也会打印 如：
'''
>>>q,rdouble_usage_func(2013,10）
>>>q
201
>>>r
3
'''
from operator import mod,floordiv
def double_usage_func(a,b):
		return mod(a,b),floor(a,b)

``````

记住：取模10获取最后一位，除并向下取整10获取除了最后一位的其他位

```
>>> floor(2013,10)
>>>3
>>> mod(2013,10)
>>>201
```

函数的形式：

```
def function(a,b):
		return value
```

​	function(a,b)统称函数签名（function signature）中间部分叫函数体（function body）return vaule称作返回值。

```
<header>:						<- |从header
		<statement>     	 |<-一行称作语句（statement）
		<statement>				 |<- statement部分称作suite
		...								 ]到此处为从句（clause）
<seprating header>
		<statement>
		<statement>
		...
...									<- 整个函数称作
解释：statement，interpreter执行并采取行动。在没有控制语句的情况下，会从上到下采取顺序执行，suite，判断header的真假值，为真，顺序执行。
```

if, while

```python
# e.g.
def my_abs(a):
  if(a<0)
  	return -a
  if (a>0):
    return a
  if(a=0):
    return 0
#只会执行一个suite
i，sum=0,0
while(i<5):
  i=i+1
  sum=sum*i
# 知道不满条件才会停下，示例会改变全部变量
```

```python
# eg
def real_sqrt(a):
  
```

