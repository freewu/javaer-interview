## Redis 做分布式锁

````
加锁: set <key> <value> nx ex 10s
释放锁: delete key
````

## Redis 持久化机制

```
RDB 内存快照
AOF	指令重放 
```

## Redis线程模型

```
```

## Redis过期键的删除策略

```
keys的过期时间使用unix时间戳存储(从 Redis 2.6 开始以毫秒为单位)
被动方式:
	当client 尝试访问 key时，发现并主动过期
主动方式:(redis每10秒执行)
	1 测试随机的20keys进行过期检测
	2 删除所有已过期的keys
	3 如果有多于25%的keys过期,重复步骤1
```

## Redis缓存回收策略 

```
noeviction 返回错误,当内存限制达到并且client尝试执行写入指令
allkeys-lru 尝试回收最少的使用 key（LRU）,使得新添加的数据有空间
volatile-lru 尝试回收最少的使用 key（LRU）, 仅限于在过期集合的key
allkeys-random 回收随机的key，使得新添加的数据有存放空间
volatile-random 回收随机的key，使得新添加的数据有存放空间，仅限于在过期集合的key
volatile-ttl 回收过期集合的key，优先回收存活时间(TTL)较短的key
volatile-lfu 从所有配置过期的key中回收使用频率最少的key
allkeys-lfu 从所有key中回收使用频率最少的key
```

## Redis 回收进程工作过程

```
client 运行新增命令
redis 检查内存使用情况,如果大于 maxmemory的限制,根据设定好的策略进行回收
新增命令被执行
```

## Redis集群方案

```
哨兵模式(主从复制集群)
Cluster模式(分片集群)
客户端实现路由分片集群
使用中间件代理层的分片集群 Codis
```

## Redis事务的实现原理

```reR
MULTI EXEC DISCARD WATCH 是 Redis事务相关的命令,事务可以一次执行多个命令
事务是一个单独的隔离操作，事务的所的命令都会序列化,按顺序执行
事务的执行过程中，不会被其它客户端发送来的命令请求打断
事务是一个原子操作,事务中的命令要么全部执行,要么全部不执行
```

## Redis 主从复制

```
 当 master实例和 slave 实例连接正常时,master会发送一连串的命令流来保持对slave的更新
 当 master实例和 slave 实例断开后(网络问题,连接超时)
 	slave 重新连上 master 并尝试进行部分重同步,尝试只获取断开连接期间内丢失的命令流
 	当无法进行部分重同步时，slave会请求全量重同步
```



