### Bean总览

Spring IoC容器管理着一个或多个的bean,bean是由配置元数据创建的

bean在容器内被表示为BeanDefinition对象,描述一下这些属性吧:

| 属性           | 解释             |
| -------------- | ---------------- |
| 类             | 实例化bean       |
| 名称           | bean名称         |
| 生命周期       | bean生命周期     |
| 构造函数参数   | 依赖注入         |
| 属性           | 依赖注入         |
| 自动注入模式   | 自动注入的合作者 |
| 延迟初始化模式 | 懒初始化bean     |
| 初始化方法     | 初始化回调       |
| 销毁方式       | 销毁回调         |

ApplicationContext还允许注册在容器外的对象注册入容器内,可以通过ApplicationContext.BeanFactory的getBeanFactory()来完成,这个方法时返回BeanFactory的DefaultListableBeanFactory的registerSingle()和reigsterBeanDefinition()支持注册.

```java
//registerBean方法传入的class和构造函数
AnnotationConfigApplicationContext context = new AnnotationConfigApplication();
context.registerBean(A.class,"");
conetxt.refresh();
A a = (A)context.getBean("a");
```

像以上bean注册需要尽早注册,以便容器在自动装配和其他自省步骤中正确的推理他们

### bean的命名

每个bean具有一个或多个标志符,这些标志符在容器中必须是唯一的

基于XML配置的bean 可以使用 id属性 name属性 或者两者来指定bean标志符

```
bean命名约定
bean名称以小写字母开头,并使用驼峰
```

基于Java Configuration的话,可以基于@Bean注解提供别名

### 实例化bean

创建bean时,容器会查看已命名bean的配方,并使用该bean的配置元数据创建实际对象