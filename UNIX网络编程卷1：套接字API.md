## UNIX网络编程卷1：套接字API

pre-operation: C API : fgets snprintf (or slprintf) 

字节序转换：htons, xx

网络地址转换： inet_addr inet_aton  inet_ntoa   inet_pton 



原因，相较于以前的API，后者可能会要求传入内存空间长度，有效避免缓存空间（内核）溢出。

额外掌握，format string。

connect 调用，

积极采用errno的形式，验证信息



about unix deamon