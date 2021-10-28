## Maven版本冲突

```
1 第一声明者优先原则 在pom文件定义依赖，先声明的依赖为准。
	在pom.xml配置文件中，如果有两个名称相同版本不同的依赖声明，那么先写的会生效。
	先声明自己要用的版本的jar包即可。
	
2 路径近者优先
	直接依赖优先于传递依赖，如果传递依赖的jar包版本冲突了，那么可以自己声明一个指定版本的依赖jar，即可解决冲突。

3 排出原则
	通过添加<exclusion>标签，将不需要的jar的传递依赖中声明排除
    
4 版本锁定原则（最常使用）
	在配置文件pom.xml中先声明要使用哪个版本的相应jar包，声明后其他版本的jar包一律不依赖
```
