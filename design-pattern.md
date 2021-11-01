## 单例模式 Double Check 的原理

````
public class LazyDoubleCheckSingleton {
	// 在Java里面，可以通过volatile关键字来保证一定的“有序性”
	// 一旦一个共享变量（类的成员变量、类的静态成员变量）被volatile修饰之后，那么就具备了两层语义：
　　 // 1）保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。
　　 // 2）禁止进行指令重排序。
    private volatile static LazyDoubleCheckSingleton instance = null;
    private LazyDoubleCheckSingleton() { 
    }

    public static LazyDoubleCheckSingleton getInstance() {
    	// 第一次检查，有声明过直接返回
        if(instance == null) {
        	// 加上类锁
            synchronized (LazyDoubleCheckSingleton.class) {
            	// 第二次检查
            	// 可能会有多个线程进入到这理
                if(instance == null){
                    instance = new LazyDoubleCheckSingleton();
                }
            }
        }
        return instance;
    }
}
````

## 使用 enum 实现懒加载的单例

```
public class EnumSingleton {
	private EnumSingleton() {
	}
	public static EnumSingleton getInstance() {
		return Holder.HOLDER.instance;
	}
	// Enum类内部会有一个构造函数，该构造函数只能有编译器调用，我们是无法手动操作的
	// jvm保证枚举的构造方法只被调用一次
	private enum Holder {
		HOLDER;
		private final EnumSingleton instance;
		Holder() {
			instance = new EnumSingleton();
		}
	}
}
```

