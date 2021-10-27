## ArrayList 和 LinkedList的区别

```
ArrayList 底层基于 数组，可以以 O(1)的时间复杂度对元素进行随机访问,插入，添加 ，删除需要O(n)的时间复杂度
LinkedList 底层基于 链表，查找元素时间复杂度为O(N) 插入，添加 ，删除 使用O(1)的时间复杂度操作 
```

## 高并发中的集合类

```
# 第一代线程安全类
	Vector Hashtable
	使用 synchronized 修饰方法
	效率低下
	
# 第二代线程非安全集合类
 	ArrayList HashMap
 	线程不安全,性能好,用来替代 Vector Hashtable
 	必须使用 Collections.synchronizedList(list) Collections.synchronizedMap(map) 保证线程安全
 	底层使用 synchronized 代码块锁 

# 第三代线程安全集合类
	java.util.concuccent.*
	ConcurrentHashMap CopyOnWriteArrayList CopyOnWriteArraySet
	底层使用 Lock 保证安全(1.8 ConcurrentHashMap 不使用 Lock 锁)
```

## HashMap和Hashtable的区别

```
Hashtable 线程同步,HashMap 非线程同步
Hashtable 不允许<key,value>有空值,HashMap 允许 <key,value>有空值
Hashtable 使用 Enumeration,HashMap 使用 Iterator
Hashtable 的 hash 数组的默认大小是11 增加方式是 old*2+1 HashMap中的数组的默认大小是16,增长是2的指数
Hashtable 继承 Dictionary类, HashMap 继承自 AbstractMap
```

## HashMap有哪些线程安全的方式

```
1 通过 Collections.synchronizedMap() 返回一个新的 Map
  新的Map就是线程安全的，返回的并不是 HashMap,而是 Map 接口的一个实现

2 使用 java.util.concurrent.ConcurrentHashMap
```

## HashMap扩容

```
HashMap(jdk1.8以后)借助2倍扩容机制，元素不需要进行重新计算位置
元素要们在原来的位置,要么在原来位置再移动2次幂的位置
```



