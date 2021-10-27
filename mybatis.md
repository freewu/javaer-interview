## mybatis和Hibernate的区别

```
 相同点:
 	都可以通过 SessionFactoryBuilder 由 XML 配置文件生成 SessionFactory,然后由 SessionFactory 生成 Session 来开启执行事务和 SQL语句
 	SessionFactoryBuilder SessionFactory Session 的生命周期差不多
 	都支持 JDBC 和 事务处理
 
 区别:
 	Hibernate 完全通过对象的关系模型实现对数据库的操作，拥有完整的 JavaBean 对象与数据库的映射结构来自动生成sql，mybatis 仅有基本的字段映射,对象数据以及对象实际关系仍需要通过手写 sql 来实现和管理
 	Hibernate 通过强大的映射结构和 hql 降低了对象与数据库的耦合,mybatis 需要手写 sql 耦合性直接取决程序员写 sql 的方法
 	Hibernate 日志系统健全 sql 记录，关系异常,优化告警,缓存提示,脏数据告警,mybatis除了基本的记录功能外，功能薄弱很多
 	Hibernate 配置比 mybatis 复杂,学习成本高
 	Sql直接优化上 mybatis 要比 Hibernate 方便
 	缓存机制上 Hibernate 比 mybatis 要好
```

## #{} 和 ${} 的区别

```
#{}是预编译处理 ${} 是字符串替换
mybatis 在处理 #{} 时,会将 sql 中 #{} 替换为 ? 调用 PreparedStatement 的 set 方法来赋值
mybatis 在处理 ${} 时, 就是把 ${}替换成 变更的值
#{} 可以有效防止 SQL 注入,提高系统安全性
```



