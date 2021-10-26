## CAP定理

```
分布式系统有三个指标。

    Consistency(一致性)
    Availability(可用性)
    Partition tolerance(分区容错性)
    
它们的第一个字母分别是 C、A、P。
Eric Brewer 说，这三个指标不可能同时做到。这个结论就叫做 CAP 定理
```

## BASE理论

```
BASE理论是由eBay架构师提出的。BASE是对CAP中一致性和可用性权衡的结果，其来源于对大规模互联网分布式系统实践的总结，是基于CAP定律逐步演化而来。其核心思想是即使无法做到强一致性，但每个应用都可以根据自身业务特点，才用适当的方式来使系统打到最终一致性。

BASE理论是Basically Available(基本可用)，Soft State（软状态）和Eventually Consistent（最终一致性）三个短语的缩写。

核心思想:
	既是无法做到强一致性（Strong consistency），但每个应用都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性（Eventual consistency）。
```

## 2PC提交协议

```
2PC是一种能够保证原子性和持久性的提交协议, 参与事务的组件有多个, 每个组件分别记录各自操作日志, 操作日志是分离式而非集中式.

2PC没有言及如何对事务做并发控制(不涉及并发控制协议), 仅仅是有别于中心化WAL日志的一种机制. 不考虑性能的情况下, 完全可以通过中心化的TM记录日志, TM通过RPC向其他事务参与者, 发起redo和undo的操作.

分布式事务, 涉及多个局部事务. 网络断开和进程崩溃等故障会导致部分局部事务提交, 部分失败. 要保证Atomicity和Durability, 需要用到提交协议, 2PC和其变种是一种广泛使用的提交协议.

在2PC中, 由一个coordinator和多个participant对分布式事务的提交进行协调. coordinator和participant写各自WAL日志, 便于故障重启后, 对事务进行恢复. coordinator是2阶段提交的发起者. 事务T的提交过程为:

	prepare阶段: 
		coordinator追加<prepare T>日志记录, 向事务T所涉及的participant发送消息	
		prepare T； 收到消息后, 
            participant判断事务T是否可以提交(申请锁, 冲突检测). 
                如果可以提交, 则追加日志<ready T>, 并向coordinator发送消息ready T; 
                如果无法提交, 则追加日志<no T>, 向coordinator发现abort T.
	
	commit阶段:
		case 1: 
			coordinator收到全体participant的ready 
			T: coordinator追加<commit T>日志, apply日志; 
			然后, coordinator向全体participant发送commit T消息, 
			收到消息后, participant追加<commit T>日志, apply日志修改状态.
		case 2: 
			coordinator收到至少一个participant的abort T 或者等待超时: 
			coordinator追加<abort T>日志, 向全体participant发送abort T消息; 
			收到信息后, participant追加<abort T>日志.

participant追加 <commit T> 或 <abort T>后, 向coordinator发送acknowledge T消息, coordinator收到所有participant发来的确认后, 写日志<complete T>, T结束.

缺点:
	同步阻塞问题
	单点故障(协调者宕机)
	数据不一致
	发送完 commit 后宕机

处理方法:
	手动补偿
```

## 3PC提交协议

```
三阶段提交升级点（基于二阶段）：

 <1>三阶段提交协议引入了超时机制。

 <2>在第一阶段和第二阶段中，引入了一个准备阶段。保证了在最后提交阶段之前各参与节点的状态是一致的。

 简单讲：就是除了引入超时机制之外，3PC把2PC的准备阶段再次一分为二，这样三阶段提交就有CanCommit、PreCommit、DoCommit三个阶段。
 
第一阶段（CanCommit 阶段）
 类似于2PC的准备（第一）阶段。协调者向参与者发送commit请求，参与者如果可以提交就返回Yes响应，否则返回No响应。
 
第二阶段（PreCommit 阶段）
 协调者根据参与者的反应情况来决定是否可以记性事务的PreCommit操作。根据响应情况，有以下两种可能。

第三阶段（doCommit 阶段）
 该阶段进行真正的事务提交，也可以分为执行提交和中断事务两种情况
```

## TCC解决方案

```
```

## 可靠消息服务方案

```
消息生产者通过业务操作完成数据的操作，在准备发送消息的时候，先将消息存储一份，然后发送给消息中间件集群。
消息消费者监听消息中间件中的消息，消费者消息处理之后处理之后，调用消息生产者接口，进行消息消费确认。
消息生产者接受消息确认之后，删除消息数据。
消息查询服务查询消息在被接收之后没有返回消息消费确认，那么就通过消息恢复能进行消息重新发送。
```

## 最大努力通知方案

```
有一定的消息重复通知机制。因为接收通知方可能没有接收到通知，此时要有一定的机制对消息重复通知。

消息校对机制。如果尽最大努力也没有通知到接收方，或者接收方消费消息后要再次消费，此时可由接收方主动向通知方查询消息信息来满足需求。
```

## 幂等

```
在编程中一个幂等操作的特点是其任意多次执行所产生的影响均与一次执行的影响相同。
幂等函数，或幂等方法，是指可以使用相同参数重复执行，并能获得相同结果的函数。这些函数不会影响系统状态，也不用担心重复执行会对系统造成改变。
例如，“setTrue()”函数就是一个幂等函数,无论多次执行，其结果都是一样的.更复杂的操作幂等保证是利用唯一交易号(流水号)实现。

幂等技术解决方案:
	唯一索引
	Token机制
	TraceID(操作唯一)
```

## 双写一致性问题

```
更新DB，delete 缓存
```



