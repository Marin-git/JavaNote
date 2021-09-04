

[TOP]

## 数据结构与对象
- 简单动态字符串（SDS）
- 链表
- 字典
- 跳跃表
- 整数集合
- 压缩列表
- 对象

### 简单动态字符串
*字符串、AOF缓冲区、客户端状态中的缓冲区* 

#### 1、结构 
```c
struct sdshdr{
    // 保存字符串的长度
    int len;
    // buf中未使用字节的数量
    int free;
    // 保存字符串
    char buf[];
}

``` 
#### 2、作用
- len
  1. 获取buf长度时，不需要遍历 
  2. 拼接字符串时，根据len判断空间是否足够，不够就扩容 
  3. 减少修改字符串时带来的内存重分配次数
   
- free   
  分配规则：  
    len < 1M : free长度与len长度一致  
    len >= 1M : free长度为1M 


### 链表 
*发布订阅、慢查询、监视器、保存客户端状态信息* 
 
#### 1、结构 
```c
typedef struct listNode{
    // 前一个节点
    struct listNode * prev;
    // 后一个节点
    struct listNode * next;
    // 节点的值
    void * value;
}
```
