[TOC]




## 缓存的使用场景



+ 什么时候用缓存
	+ 不需要实时更新但是又极其消耗数据库的数据
	+ 需要实时更新，但是更新频率不高的数据
	+ 在某个时刻访问量极大而且更新也很频繁的数据

+ 什么时候不用缓存
	+ 涉及到钱、密钥、业务关键性核心数据等
	+ 大部分数据都是可以缓存的80%,二八原则

+ 缓存的问题：
	+ 多线程并发控制
	+ 缓存数据和真实数据的同步问题
	+ 缓存的老化
	+ redis没有事务的概念，没有ACID






###Redis

redis放弃传统的sql语句和ACID保证
+ Redis 将数据储存在内存里面，读写数据的时候都不受硬盘 I/O 速度的限制
+ 即使做了持久化设置，持久化开启子线程
	+ 比如默认开启dump，写磁盘是合并的
	+ 在没有开启AOF的情况下，每一次都写，性能上不是很好，不过其实redis说到底是用来做缓存的，个人认为没必要保证缓存数据的可用性，通过热备来做缓存，会引起设计上的混乱。分布式缓存和优秀的缓存路由算法可以解决这个问题
	+ 一致性hash算法

+ [多路复用](https://www.zhihu.com/question/28594409)
	+ 复用：复用同一线程
	+ 多路：多个socket
多路I/O复用模型是利用 select、poll、epoll 可以同时监察多个流的 I/O 事件的能力，在空闲的时候，会把当前线程阻塞掉，当有一个或多个流有 I/O 事件时，就从阻塞态中唤醒，于是程序就会轮询一遍所有的流（epoll 是只轮询那些真正发出了事件的流），并且只依次顺序的处理就绪的流，这种做法就避免了大量的无用操作。



#### redis 为什么那么快？

+ 绝大多数的请求都是内存操作，数据基于内存存储
+ 数据在内存中，类似于hash，查找时间复杂度为O(1)
+ 数据结构简单，操作也简单，摒弃了传统sql的表连接等操作
+ 单线程避免了上下文切换,锁竞争,不需要考虑并发关注的锁，共享等问题
+ 使用多路IO复用m模型，非阻塞IO
+ 减少了切换内核态的操作，Redis底层自己做了优化，具体不是很了解



使用底层模型不同，它们之间底层实现方式以及与客户端之间通信的应用协议不一样，Redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求；

	+ 数据
	+ 单进程单线程模型的 KV 数据库，
	+ C语言编写
	+ 每秒内查询：
		+ 最高: $10^5QPS$,
		+ 高连接:$4*10^4 QPS$



- [redis为什么是单线程](https://blog.csdn.net/chenyao1994/article/details/79491337)

这里的单线程指的是处理数据是单线程，dump操作等其他操作是在子线程完成的。

官方的解释是，单线程已经足够快，并且这种情况下瓶颈主要在于memory or network.
官方的建议是开始多个实例做配置
听说在4.0 后有改进，不是很了解

单线程，所以尽量不要进行耗时操作，比如union操作
单线程，所以Redis更钟情于大Cache高速CPU


1. 单进程多线程模型：MySQL、Memcached、Oracle（Windows版本）
2. 多进程模型：Oracle（Linux版本）；
3. Nginx有两类进程，一类称为Master进程(相当于管理进程)，另一类称为Worker进程（实际工作进程）。启动方式有两种：

	+ 单进程启动：此时系统中仅有一个进程，该进程既充当Master进程的角色，也充当Worker进程的角色。
	+ 多进程启动：此时系统有且仅有一个Master进程，至少有一个Worker进程工作。
	+ Master进程主要进行一些全局性的初始化工作和管理Worker的工作；事件处理是在Worker中进行的。
- redis单线程有什么缺点，如果用多线程有什么 优缺点

- aof，rdb，优点，区别




### redis订阅与发布：


```bash

# A 端
SUBSCRIBE msg


# B 端
PUBLISH msg "hello world"
```

```c
struct redisServer
{

    /* Pubsub */
    // 字典，键为频道，值为链表
    // 链表中保存了所有订阅某个频道的客户端
    // 新客户端总是被添加到链表的表尾
    dict *pubsub_channels; /* Map channels to list of subscribed clients */

    // 这个链表记录了客户端订阅的所有模式的名字
    list *pubsub_patterns; /* A list of pubsub_patterns */
    ...

}



/* Publish a message 
 *
 * 将 message 发送到所有订阅频道 channel 的客户端，
 * 以及所有订阅了和 channel 频道匹配的模式的客户端。
 */
int pubsubPublishMessage(robj *channel, robj *message) {
    ...
    /* Send to clients listening for that channel */
    // 取出包含所有订阅频道 channel 的客户端的链表
    // 并将消息发送给它们
    de = dictFind(server.pubsub_channels,channel);
    if (de) {
        list *list = dictGetVal(de);
        listNode *ln;
        listIter li;

        // 遍历客户端链表，将 message 发送给它们
        listRewind(list,&li);
        while ((ln = listNext(&li)) != NULL) {
            redisClient *c = ln->value;

            // 回复客户端。
            // 示例：
            // 1) "message"
            // 2) "xxx"
            // 3) "hello"
            addReply(c,shared.mbulkhdr[3]);
            // "message" 字符串
            addReply(c,shared.messagebulk);
            // 消息的来源频道
            addReplyBulk(c,channel);
            // 消息内容
            addReplyBulk(c,message);

            // 接收客户端计数
            receivers++;
        }
    }

    /* Send to clients listening to matching channels */
    ...
    // 返回计数
    return receivers;
}

```

### Redis启动
- 进入到Redis文件所在文件，以终端状态打开
- redis-server.exe redis.conf   ------- //以conf文件中的信息开启Redis的服务器
 redis.conf文件中配置了许多信息，若需要修改，可以复制一份，然后再修改所需要修改的信息
- redis-cli.exe -h 127.0.0.1   -----//开启接口为127.0.0.1的redis客户端
    端口号在conf文件中设置了，故直接连接接口即可
