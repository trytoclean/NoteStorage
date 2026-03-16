### STL

一些容器

<img src="/Users/tianjiashun/Pictures/截屏/截屏2025-11-06 08.03.00.png" alt="截屏2025-11-06 08.03.00" style="zoom:50%;" />

算法

​	依据底层实现，依据需求（查询还是排序）来理解容器

如 priority_queue 的实现是树，本质是adaptor，std::priority_queue<T, std::vector<T>, std::less<T> less是大顶堆，greater是小顶堆 。 数据存储在 如vector中，类型是T。除了增加删除之外，只能返回top。

map ，set 底层是实现是BST，二叉搜索树，如红黑树，查找、插入、删除都是 O(log n) ，可以自动排序。二叉树的特点是，**不能有重复的元素**（重复的键），如有重复，机制是忽略。

map存储的是键值对，可能叫做entry，访问就是  value=mp[key]

multimap，特点是可以存储重复的键，但是不能根据键来访问值了。 multiset同理

pair的实现应该是std::tuple 有待考证. 访问方法是 a.first a.second

deque是双端可以插入和删除的队列 

至于那个bitset 是怎样“节省内存de 01串数组” 有待考察，课程没有讲

> | 层次   | 节省方式                      | 实现机制           |
> | ------ | ----------------------------- | ------------------ |
> | 存储层 | 每位仅占 1 bit                | 整数按位打包       |
> | 编译层 | 按机器字长对齐（64 位为一组） | 无空洞、无 padding |
> | 算法层 | 用位运算代替条件判断          | 提升局部性与性能   |
> | 系统层 | 连续内存、cache 友好          | 提高访问效率       |
>
> 长度固定、不能动态扩容
>
> 如果是vector<bool>每个元素占一个byte 



