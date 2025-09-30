# 特点

将会在后面逐一介绍这些特点, 此处只是列出来作为参考.

* 高性能
  * 内存数据库
  * 单线程异步IO(避免多线程竞争和阻塞)
* 多数据结构 &rarr; String, Hash, List, Zset, Sorted Set, etc
* 持久化
  * RDB snapshot &rarr; 内存数据二进制格式写进磁盘
  * AOF(append-only file) &rarr; 追加指令
* 常见用法
  * 消息队列
  * 分布式缓存
  * 集群
  * 哨兵机制
  * 分布式锁
* 支持事务
* 支持lua脚本, 原子操作

---

# 对比

## vs memcached

## vs ElasticSearch

## vs MongoDB