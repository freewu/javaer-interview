## BeanFactory和FactoryBean的区别

```
 BeanFactory
    BeanFactory定义了IOC容器的最基本形式，并提供了IOC容器应遵守的的最基本的接口，也就是Spring IOC所遵守的最底层和最基本的编程规范。在Spring代码中，BeanFactory只是个接口，并不是IOC容器的具体实现，但是Spring容器给出了很多种实现，如 DefaultListableBeanFactory、XmlBeanFactory、ApplicationContext等，都是附加了某种功能的实现

FactoryBean
	Spring通过反射机制利用<bean>的class属性指定实现类实例化Bean，在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在<bean>中提供大量的配置信息。配置方式的灵活性是受限的，这时采用编码的方式可能会得到一个简单的方案。Spring为此提供了一个org.springframework.bean.factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑。
FactoryBean接口对于Spring框架来说占用重要的地位，Spring自身就提供了70多个FactoryBean的实现。它们隐藏了实例化一些复杂Bean的细节，给上层应用带来了便利。从Spring3.0开始，FactoryBean开始支持泛型，即接口声明改为FactoryBean<T>的形式

区别
	BeanFactory是个Factory，也就是IOC容器或对象工厂，FactoryBean是个Bean。在Spring中，所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的。但对FactoryBean而言，这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂模式和修饰器模式类似。
```

## SpringBoot自动装配原理

```
1 SpringBoot通过main方法启动SpringApplication类的静态方法run()来启动项目
2 run方法从一个使用了默认配置的指定资源启动一个SpringApplication并返回ApplicationContext对象
3 SpringApplication类的 @SpringBootApplication注解(复合注解) 指定默认配置 
	@SpringBootConfiguration：
        这个注解的底层是一个 @Configuration 注解，
        意思被 @Configuration 注解修饰的类是一个 IOC 容器，支持 JavaConfig 的方式来进行配置；
	@ComponentScan：
		默认扫描当前类所在的包及其子包下包含的注解，
		将 @Controller / @Service / @Component / @Repository 等注解加载到 IOC 容器中；
	@EnableAutoConfiguration：
		这个注解表明启动自动装配，里面包含连个比较重要的注解@AutoConfigurationPackage和@Import
		@AutoConfigurationPackage
			和@ComponentScan一样，也是将主配置类所在的包及其子包里面的组件扫描到IOC容器中，
			@AutoConfigurationPackage 扫描 @Enitity、@MapperScan 等第三方依赖的注
		@Import(AutoConfigurationImportSelector.class)
			自动装配的核心注解
			AutoConfigurationImportSelector.class中有个selectImports方法
			selectImports方法还调用了getCandidateConfigurations方法
			getCandidateConfigurations 来找META-INF/spring.factories文件的
			spring.factories文件是一组组的key=value的形式，包含了key为EnableAutoConfiguration的全类名，value是一个AutoConfiguration类名的列表，以逗号分隔
```

# SpringMVC执行流程及工作原理

```
SpringMVC执行流程:
    1.用户发送请求至前端控制器 DispatcherServlet
    2.DispatcherServlet 收到请求调用处理器映射器 HandlerMapping。
    3.处理器映射器根据请求url找到具体的处理器，生成处理器执行链 HandlerExecutionChain(包括处理器对象和处理器拦截器)一并返回给DispatcherServlet。
    4.DispatcherServlet 根据处理器 Handler 获取处理器适配器 HandlerAdapter 执行 HandlerAdapter 处理一系列的操作，如：参数封装，数据格式转换，数据验证等操作
    5.执行处理器Handler(Controller，也叫页面控制器)。
    6.Handler执行完成返回ModelAndView
    7.HandlerAdapter将Handler执行结果ModelAndView返回到DispatcherServlet
    8.DispatcherServlet将ModelAndView传给ViewReslover视图解析器
    9.ViewReslover解析后返回具体View
    10.DispatcherServlet对View进行渲染视图（即将模型数据model填充至视图中）。
    11.DispatcherServlet响应用户。

组件说明:
    1.DispatcherServlet：前端控制器。用户请求到达前端控制器，它就相当于mvc模式中的c，dispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求，dispatcherServlet的存在降低了组件之间的耦合性,系统扩展性提高。由框架实现
    2.HandlerMapping：处理器映射器。HandlerMapping负责根据用户请求的url找到Handler即处理器，springmvc提供了不同的映射器实现不同的映射方式，根据一定的规则去查找,例如：xml配置方式，实现接口方式，注解方式等。由框架实现
    3.Handler：处理器。Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。由于Handler涉及到具体的用户业务请求，所以一般情况需要程序员根据业务需求开发Handler。
    4.HandlAdapter：处理器适配器。通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。由框架实现。
    5.ModelAndView是springmvc的封装对象，将model和view封装在一起。
    6.ViewResolver：视图解析器。ViewResolver负责将处理结果生成View视图，ViewResolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。
    7View:是springmvc的封装对象，是一个接口, springmvc框架提供了很多的View视图类型，包括：jspview，pdfview,jstlView、freemarkerView、pdfView等。一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开发具体的页面。
```

## SpringMVC的九大组件

```
1.HandlerMapping(处理器映射器)
　　HandlerMapping的主要作用是根据请求的资源uri来查找对应的handler
　　handler代表实际可以处理请求的方法或者类,例如被注解@ResquestMapping所标记的方法,就可以看作一个Handler

2. HandlerAdapter(处理器适配器)
	在我们找到对应的Handler时,我们则要开始处理Handler,
	因为Handler的实现多种多样,所以对于Handler不同的内部结构需要进行一定的处理

	容器在初始化的时候会自动帮我们注入 (也可以自己配置)RequestMappingHandlerAdapter 、HttpRequestHandlerAdapter 和 impleControllerHandlerAdapter 这三个配置器。
       
    supports方法:是判断该适配器是否支持这个HandlerMethod，
    就是当得到一个handler时，该接口子类该方法做判断（就是类似handler instanceof Controller的判断方式），用来得到适配这个handler的适配器子类。
	handle方法:用来执行控制器处理函数，获取ModelAndView 。就是根据该适配器调用规则执行handler方法
	getLastModified方法:处理响应请求,控制客户端的GET请求(如post请求不受影响）是否被真实响应还是直接响应为不修改(304）,servlet的service方法根据getLastModified的返回值
	
    HttpRequestHandlerAdapter可以执行 HttpRequestHandler 类型的 handler
    SimpleControllerHandlerAdapter可以执行 Servlet 类型的 handler
    RequestMappingHandlerAdapter可以执行 Controller 类型的 handler
　　
3. HandlerExceptionResolver(异常处理器)
　　当我们在寻找和处理Handler时难免会出现一些问题(异常),这个时候就需要一个专门来处理异常的角色
　　
4. ViewResolver(页面渲染处理器)
　　View是用来渲染页面的,而ViewResolver所要做的就是找到渲染所用的模板和技术(页面类型)

5. RequestToViewNameTranslator(视图名称翻译器)
	有的Handler处理完后并没有设置View也没有设置ViewName时，需要从request获取ViewName,而如何从request中获取ViewName就是RequestToViewNameTranslator要做的事情了.

　　当 ModelAndView 对象不为null，但是它的 View 对象为null，则需要通过 RequestToViewNameTranslator 组件根据请求解析出一个默认的视图名称。

6. LocaleResolver(当前环境处理器)
　　View的解析需要两个参数,一个是视图名,另一个是Locale(语言环境).视图名是处理器返回的,而Locale是由LocaleResolver从request中解析得到.
　　这就相当于配置数据库的方言一样，有了这个就可以对不同区域的用户显示不同的结果。
　　SpringMVC主要有两个地方用到了Locale：
        一是ViewResolver视图解析的时候；
        二是用到国际化资源或者主题的时候。
 
7.ThemeResolver(主题处理器)
　　SpringMVC中跟主题相关的类有ThemeResolver、ThemeSource和Theme。主题是通过一系列资源来具体体现的，要得到一个主题的资源，首先要得到资源的名称，这是ThemeResolver的工作。然后通过主题名称找到对应的主题（可以理解为一个配置）文件，这是ThemeSource的工作。最后从主题中获取资源就可以了。


8.MultipartResolver(文件处理器)
　　用于处理上传请求。处理方法是将普通的request包装成MultipartHttpServletRequest，后者可以直接调用getFile方法获取File，如果上传多个文件，还可以调用getFileMap得到FileName->File结构的Map。此组件中一共有三个方法，作用分别是判断是不是上传请求，将request包装成MultipartHttpServletRequest、处理完后清理上传过程中产生的临时资源。

9.FlashMapManager(参数传递管理器)
　　后端有请求转发和请求重定向两种方式，请求转发的时候Request是同一个，所以可以在转发后拿到转发前的所有信息；但是重定向后 Request是新的，如果需要在重定向前设置一些信息，重定向后获取使用应该怎么办法呢？
　　这就是 FlashMap存在的意义，FlashMap 借助 session 重定向前通过 FlashMapManager将信息放入FlashMap,重定向后 再借助 FlashMapManager 从 session中找到重定向后需要的 FalshMap。
```

## Spring的事务传播机制

```
Spring的事务传播机制有7种，在枚举Propagation中有定义

# REQUIRED
	PROPAGATION_REQUIRED：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的默认设置。

    @Transactional(propagation= Propagation.REQUIRED)
    public void methodA(){
        methodB();
        // do something
    }

    @Transactional(propagation= Propagation.REQUIRED)
    public void methodB(){
        // do something
    }
    调用methdoA，如果methodB发生异常，触发事务回滚，也会methodA中的也会回滚。

# SUPPORTS
	PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。
	
	@Transactional(propagation= Propagation.REQUIRED)
    public void methodA(){
        methodB();
        // do something
    }

    @Transactional(propagation= Propagation.SUPPORTS)
    public void methodB(){
        // do something
    }
	如果调用methodA，再调用methodB，MehtodB会加入到MethodA的开启的当前事务中。
	如果直接调用methodB，当前没有事务，就以非事务执行。
	
# MANDATORY
	PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。
	@Transactional(propagation= Propagation.REQUIRED)
    public void methodA(){
        methodB();
        // do something
    }

    @Transactional(propagation= Propagation.MANDATORY)
    public void methodB(){
        // do something
    }
    如果调用methodA，再调用methodB，MehtodB会加入到MethodA的开启的当前事务中。
	如果直接调用methodB，当前没有事务，就会抛出异常。
	
# REQUIRES_NEW
	PROPAGATION_REQUIRES_NEW：创建新事务，无论当前存不存在事务，都创建新事务
	@Transactional(propagation= Propagation.REQUIRED)
    public void methodA(){
        // do something pre
        methodB();
        // do something post
    }

    @Transactional(propagation= Propagation.REQUIRES_NEW)
    public void methodB(){
        // do something
    }
    调用methodA，会先开启事务1，执行A的something pre的代码。
    再调用methodB，methdoB会开启一个事务2，再执行自身的代码。
    最后在执行methodA的something post。
    如果method发生异常回滚，只是methodB中的代码回滚，不影响methodA中的代码。
    如果methodA发生异常回滚，只回滚methodA中的代码，不影响methodB中的代码。
	简言之，不会影响别人，也不会被别人影响。

# NOT_SUPPORTED
	PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
	@Transactional(propagation= Propagation.REQUIRED)
    public void methodA(){
        methodB();
        // do something
    }

    @Transactional(propagation= Propagation.NOT_SUPPORTED)
    public void methodB(){
        // do something
    }
	调用methodA，再调用methodB，methodA开启的事务会被挂起，即在methodB中不齐作用，相当于没有事务，	 methodB内部抛出异常不会回滚。methodA内的代码发生异常会回滚。
	直接调用methodB，不会开启事务。

# NEVER
	PROPAGATION_NEVER：以非事务方式执行操作，如果当前存在事务，则抛出异常。
	@Transactional(propagation= Propagation.REQUIRED)
    public void methodA(){
        methodB();
        // do something
    }

    @Transactional(propagation= Propagation.NEVER)
    public void methodB(){
        // do something
    }

# NESTED
	PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。
	@Transactional(propagation= Propagation.REQUIRED)
    public void methodA(){
        // do something pre
        methodB();
        // do something post
    }

    @Transactional(propagation= Propagation.NESTED)
    public void methodB(){
        // do something
    }
	调用methodA，开启一个事务，执行something pre的代码，
	设置回滚点savepoint,再调用methodB的代码，
	如果methodB里抛出异常，此时回滚到之前的saveponint。
	再然后执行methodA里的something post的代码，最后提交或者回滚事务。
	嵌套事务，外层的事务如果回滚，会导致内层的事务也回滚；
	但是内层的事务如果回滚，仅仅是回滚自己的代码，不影响外层的事务的代码。

```

