#### 什么是Spring?

 		Spring 框架指的都是 Spring Framework，它是很多模块的集合，使⽤这些模块可以很方便地协助我们进⾏开发。这些模块是：核⼼容器、数据访问/集成,、Web、AOP（⾯向切⾯编程）、⼯ 具、消息和测试模块。⽐如：Core Container 中的 Core 组件是Spring 所有组件的核⼼，Beans 组件 和 Context 组件是实现IOC和依赖注⼊的基础，AOP组件⽤来实现⾯向切⾯编程。

```
核⼼技术 ：依赖注⼊(DI)，AOP，事件(events)，资源，i18n，验证，数据绑定，类型转换，
SpEL。
测试 ：模拟对象，TestContext框架，Spring MVC 测试，WebTestClient。
数据访问 ：事务，DAO⽀持，JDBC，ORM，编组XML。
Web⽀持 : Spring MVC和Spring WebFlux Web框架。
集成 ：远程处理，JMS，JCA，JMX，电⼦邮件，任务，调度，缓存。
语⾔ ：Kotlin，Groovy，动态语⾔。
```

#### Spring模块有哪些？

Spring4.x 版本。⽬前最新的5.x版本中 Web 模块的 Portlet 组件已经被废弃掉，同时 增加了⽤于异步响应式处理的 WebFlux 组件。

![Spring模块](..\..\image\Spring模块.png)

+ Spring Core： 基础,可以说 Spring 其他所有的功能都需要依赖于该类库。主要提供 IoC 依赖 注⼊功能
+  Spring Aspects ： 该模块为与AspectJ的集成提供⽀持
+  Spring AOP ：提供了⾯向切⾯的编程实现
+ Spring JDBC : Java数据库连接
+ Spring JMS ：Java消息服务
+ Spring ORM : ⽤于⽀持Hibernate等ORM⼯具
+ Spring Web : 为创建Web应⽤程序提供⽀持
+ Spring Test : 提供了对 JUnit 和 TestNG 测试的⽀持。

#### @RestController和@Controller

​		单独使⽤ @Controller 不加 @ResponseBody 的话⼀般使⽤在要返回⼀个视图的情况，这种情况 属于⽐较传统的Spring MVC 的应⽤，对应于前后端不分离的情况。

> @ResponseBody 注解的作⽤是将 Controller 的⽅法返回的对象通过适当的转换器转换为指定 的格式之后，写⼊到HTTP 响应(Response)对象的 body 中，通常⽤来返回 JSON 或者 XML 数据， 返回 JSON 数据的情况⽐᫾多。

​		@RestController 只返回对象，对象数据直接以 JSON 或 XML 形式写⼊ HTTP 响应(Response) 中，这种情况属于 RESTful Web服务，这也是⽬前⽇常开发所接触的最常⽤的情况（前后端分离）。

#### bean 的作用域有哪些?

+ singleton : 唯⼀ bean 实例，Spring 中的 bean 默认都是单例的
+ prototype : 每次请求都会创建⼀个新的 bean 实例
+ request : 每⼀次HTTP请求都会产⽣⼀个新的bean，该bean仅在当前HTTP request内有效
+ session : 每⼀次HTTP请求都会产⽣⼀个新的 bean，该bean仅在当前 HTTP session 内有效
+ global-session： 全局session作⽤域，仅仅在基于portlet的web应⽤中才有意义，Spring5已 经没有了。Portlet是能够⽣成语义代码(例如：HTML)⽚段的⼩型Java Web插件。它们基于 portlet容器，可以像servlet⼀样处理HTTP请求。但是，与 servlet 不同，每个 portlet 都有 不同的会话

#### 单例 bean 的线程安全问题

单例 bean 存在线程问 题，主要是因为当多个线程操作同⼀个对象的时候，对这个对象的⾮静态成员变量的写操作会存在线程 安全问题

常见的有两种解决办法： 

1. 在Bean对象中尽量避免定义可变的成员变量（不太现实）
2. 在类中定义⼀个ThreadLocal成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的 ⼀种⽅式）

#### @Component 和 @Bean 的区别是什么？

+ 作⽤对象不同: @Component 注解作⽤于类，⽽ @Bean 注解作⽤于⽅法。 
+ @Component 通常是通过类路径扫描来⾃动侦测以及⾃动装配到Spring容器中（我们可以使⽤ @ComponentScan 注解定义要扫描的路径从中找出标识了需要装配的类⾃动装配到 Spring 的 bean 容器中）。 @Bean 注解通常是我们在标有该注解的⽅法中定义产⽣这个 bean, @Bean 告 诉了Spring这是某个类的示例，当我需要⽤它的时候还给我
+ @Bean 注解⽐ Component 注解的⾃定义性更强，⽽且很多地⽅我们只能通过 @Bean 注解 来注册bean。⽐如当我们引⽤第三⽅库中的类需要装配到 Spring 容器时，则只能通过 @Bean 来实现。

#### 将⼀个类声明为Spring的 bean 的注解有哪些?

⼀般使⽤ @Autowired 注解⾃动装配 bean，要想把类标识成可⽤于 @Autowired 注解⾃动装 配的 bean 的类,采⽤以下注解可实现： 

+ @Component ：通⽤的注解，可标注任意类为 Spring 组件。如果⼀个Bean不知道属于哪个 层，可以使⽤ @Component 注解标注
+ @Repository : 对应持久层即 Dao 层，主要⽤于数据库相关操作
+ @Service : 对应服务层，主要涉及⼀些复杂的逻辑，需要⽤到 Dao层
+ @Controller : 对应 Spring MVC 控制层，主要⽤户接受⽤户请求并调⽤ Service 层返回数 据给前端页面

#### bean 生命周期

#### SpringMVC工作原理

![SpringMVC](..\..\image\SpringMVC.png)

+ M：数据处理

+ V：视图展示
+ C：请求转发

![SpringMVC原理](..\..\image\SpringMVC原理.png)

+ 客户端（浏览器）发送请求，直接请求到 DispatcherServlet 
+ DispatcherServlet 根据请求信息调⽤ HandlerMapping ，解析请求对应的 Handler 
+ 解析到对应的 Handler （也就是我们平常说的 Controller 控制器）后，开始由
  HandlerAdapter 适配器处理
+ HandlerAdapter 会根据 Handler 来调⽤真正的处理器开处理请求，并处理相应的业务逻辑
+ 处理器处理完业务后，会返回⼀个 ModelAndView 对象， Model 是返回的数据对象， View是个逻辑上的 View 
+ ViewResolver 会根据逻辑 View 查找实际的 View 
+ DispaterServlet 把返回的 Model 传给 View （视图渲染）
+ 把 View 返回给请求者（浏览器）

#### Spring事务

+ 编程式事务，在代码中硬编码。(不推荐使⽤) 
+ 声明式事务，在配置⽂件中配置（推荐使⽤）：基于XML、基于注解

##### 隔离级别有哪⼏种?

TransactionDefinition 接⼝中定义了五个表示隔离级别的常量

+ TransactionDefinition.ISOLATION_DEFAULT: 使⽤后端数据库默认的隔离级别，Mysql 默认采 ⽤的 REPEATABLE_READ隔离级别 Oracle 默认采⽤的 READ_COMMITTED隔离级别.
+ TransactionDefinition.ISOLATION_READ_UNCOMMITTED: 最低的隔离级别，允许读取尚未提交的 数据变更，可能会导致脏读、幻读或不可重复读 
+ TransactionDefinition.ISOLATION_READ_COMMITTED: 允许读取并发事务已经提交的数据，可以 阻⽌脏读，但是幻读或不可重复读仍有可能发⽣ 
+ TransactionDefinition.ISOLATION_REPEATABLE_READ: 对同⼀字段的多次读取结果都是⼀致 的，除⾮数据是被本身事务⾃⼰所修改，可以阻⽌脏读和不可重复读，但幻读仍有可能发⽣
+ TransactionDefinition.ISOLATION_SERIALIZABLE: 最⾼的隔离级别，完全服从ACID的隔离级 别。所有的事务依次逐个执⾏，这样事务之间就完全不可能产⽣⼲扰，也就是说，该级别可以防 ⽌脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会⽤到该级别。

##### 事务传播行为

+ ⽀持当前事务的情况
  + TransactionDefinition.PROPAGATION_REQUIRED(默认)： 如果上级没有事务，当前方法就新建一个事务；如果上级有事务，则加入到上级事务中；两者互相影响，任意一回滚都导致另一个回滚
  + TransactionDefinition.PROPAGATION_SUPPORTS：如果上级有事务，当前方法就和上级使用同一事务（只要有一个发生错误回滚，另一个也会回滚）；如果上级没有事务，当前方法就以非事务方式执行。
  + TransactionDefinition.PROPAGATION_MANDATORY： 如果上级没有事务，则会抛出一个异常；如果上级有事务则使用同一个事务；两者互相影响
+ 不⽀持当前事务的情况
  + TransactionDefinition.PROPAGATION_REQUIRES_NEW： 表示当前方法运行在自己创建的新事务中。如果上级存在当前事务，那么当前方法执行期间，上级事务会被挂起。两者互不影响，完全隔离
  + TransactionDefinition.PROPAGATION_NOT_SUPPORTED： 当前方法以非事务方式执行(失败不回滚)，当上一级存在事务时就将它挂起直到当前方法执行完毕（这可能会造成死锁，被挂起的事务先锁数据然后当前方法需要访问同一数据）
  + TransactionDefinition.PROPAGATION_NEVER：当前方法以非事务方式执行(失败不回滚)，当上一级存在事务时则抛出异常
+ 其他情况
  + TransactionDefinition.PROPAGATION_NESTED：如果上级有事务，则当前方法的事务会嵌套在上级事务中执行（上级影响当前方法，但当前方法不影响上级）。如果上级没有事务，则当前方法会新建自己的事务，此时等价于 TransactionDefinition.PROPAGATION_REQUIRED

##### @Transactional(rollbackFor = Exception.class)注解了解吗？

Exception分为运⾏时异常RuntimeException和非运行时异常。事务管理对于企业应用来说是至关重要，即使出现异常情况，它也可以保证数据的⼀致性。当@Transactional 注解作⽤于类上时，该类的所有 public ⽅法将都具有该类型的事务属性，同 时，我们也可以在⽅法级别使⽤该标注来覆盖类级别的定义。如果类或者⽅法加了这个注解，那么这个 类⾥⾯的⽅法抛出异常，就会回滚，数据库里面的数据也会回滚。在 @Transactional 注解中如果不配置 rollbackFor 属性,那么事物只会在遇 到 RuntimeException 的时候才会回滚,加上 rollbackFor=Exception.class ,可以让事物在遇到 ⾮运⾏时异常时也回滚。

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {

    /**
     * 当在配置文件中有多个 TransactionManager , 可以用该属性指定选择哪个事务管理器。
     */
    @AliasFor("transactionManager")
    String value() default "";

    /**
     * 同上。
     */
    @AliasFor("value")
    String transactionManager() default "";

    /**
     * 事务的传播行为，默认值为 REQUIRED。
     */
    Propagation propagation() default Propagation.REQUIRED;

    /**
     * 事务的隔离规则，默认值采用 DEFAULT。
     */
    Isolation isolation() default Isolation.DEFAULT;

    /**
     * 事务超时时间
     */
    int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;

    /**
     * 是否只读事务
     */
    boolean readOnly() default false;

    /**
     * 用于指定能够触发事务回滚的异常类型。
     */
    Class<? extends Throwable>[] rollbackFor() default {};

    /**
     * 同上，指定类名。
     */
    String[] rollbackForClassName() default {};

    /**
     * 用于指定不会触发事务回滚的异常类型
     */
    Class<? extends Throwable>[] noRollbackFor() default {};

    /**
     * 同上，指定类名
     */
    String[] noRollbackForClassName() default {};

}
```



https://www.ibm.com/developerworks/cn/java/j-master-spring-transactional-use/index.html

#### Spring中的设计模式

+ ⼯⼚设计模式 : Spring使⽤⼯⼚模式通过 BeanFactory 、 ApplicationContext 创建 bean 对象
+ 代理设计模式 : Spring AOP 功能的实现
+ 单例设计模式 : Spring 中的 Bean 默认都是单例的
+ 包装器设计模式 : 我们的项⽬需要连接多个数据库，⽽且不同的客户在每次访问中根据需要会 去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源
+ 观察者模式: Spring 事件驱动模型就是观察者模式很经典的⼀个应⽤
+ 适配器模式 :Spring AOP 的增强或通知(Advice)使⽤到了适配器模式、spring MVC 中也是⽤到 了适配器模式适配 Controller 。

https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485303&idx=1&sn=9e4626a1e3f001f9b0d84a6fa0cff04a&chksm=cea248bcf9d5c1aaf48b67cc52bac74eb29d6037848d6cf213b0e5466f2d1fda970db700ba41&token=255050878&lang=zh_CN%23rd