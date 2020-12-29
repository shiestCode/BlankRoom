### Spring IoC容器和bean的介绍

`IoC也被称为依赖注入(DI)`

在此过程中对象可以通过一下方式创建出来:

1. 构造函数
2. 工厂方法
3. 在构造函数或者工厂方法返回的的对象实例上设置的属性来定义其依赖项(与他们一起使用的其他对象)

```java
//举一个控制反转的例子
定义一个CacheService的接口 作为缓存接口
    //实现Redis的缓存
    RedisCacheServiceImpl implements CacheService
    //实现的本地缓存
    LocalCacheServiceImpl implements CacheService
//如果没有Spring的IoC我们的代码会是这样的：
    CacheService cache = new RedisCacheServiceImpl();
	cache.set();
	...
 //如果需要替换缓存方式的话需要大替换
    CacheService cache = new LocalCacheServiceImpl();
//这样就是面向接口编程,写代码主动创建对象来引用不同实现
//再看看IoC是怎么使用的
	@Service("localCache")
	LocalCacheService implements CacheService{}
	@Service("redisCache")
	RedisCacheService implements CacheService{}
    @Autowired
	@Qualifier("redisCacher")
	CacheService cache;     
//可以看到new关键字没了,多了一些注解
//@Service是把被标记了的服务告诉IoC容器
//@Autowired是告诉IoC容器我想引用什么服务
//当接口有多个实现时,我们需要@Qualifier来指定注入对应名称的服务 
//整个过程就是：注解将想要引用的服务告诉IoC容器,也就是我们所依赖对象的获取方式变成通过IoC获取了
//这就是控制反转
```

**org.springframework.beans**包和**org.springframework.context**包是IoC的基础

**BeanFactory**接口提供了一种高级的装配机制,能够管理任何类型的对象.

**ApplicationContex**t是BeanFactory的超集,ApplicationContext是一个大而全的类,只要获取到容器中的ApplicationContext的对象就可以直接获取到对象,发布消息。

### Bean的定义:

​	**构成应用程序主干**并由**IoC容器管理的对象**成为Bean.

​	Bean是**由IoC容器实例化,组装,管理**的对象.

​	Bean之间的**依赖关系**反映在容器使用的**配置元数据**中.

`注册为Bean的对象才可以用Spring的注解来注入他的依赖项`

### 容器概念:

​	org.springframework.context.**ApplicationContex**t接口就代表**SpringIoC容器**,并负责**实例化,配置和组装Bean**.

​	**容器**通过**读取元数据**配置来获取相关要**实例化,配置和组装**哪些对象的**指令**

#### **配置元数据**

​	**配置元数据**以**XML**,**注解**或者**Java代码**形式表示.他是用来表示**组成应用程序的对象**以及这些**对象之间的丰富相互依赖的关系**

```java
//几种配置元数据的方式
1.XML
    <bean id="..." class="...">
2.注解
    @Service @Component等
3.Java Code
    @Configuration
    public class CommonConfig{
        @Bean
        public Validator validator(){}
    }
```

Spring主要实现两个工作：

​	1.**找到**所有**需要被注册的bean**，对他们进行**解析**

​	2.进行**实例化解决bean之间的依赖**

ApplicationContext各种实现类的区别就在于第一步：查找和解析bean的方式有区别

比如又从XML中解析的,从Configuration中解析的

Spring2.5 引入了基于注解的配置元数据的支持

Spring3.0 引入了Spring JavaConfig[**@Configuration, @Bean, @Import, @DependsOn**]

#### 使用容器：

ApplicationContext是一个维护bean定义和相互依赖关系的注册表的高级工厂接口

```java
//里面有这么一个接口,可以用来获取实例
T getBean(String name, Class reqiredType);
//但是我们一般不会用的，我们都是直接@Autowired
```

