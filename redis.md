## Redis 做分布式锁

````
加锁: set <key> <value> nx ex 10s
释放锁: delete key
````

