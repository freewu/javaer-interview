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



