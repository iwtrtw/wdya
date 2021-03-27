spring的最大特性，spring aop ,ioc

+ IOC是一种设计思想，为相互依赖的组件提供抽象，将依赖（低层模块）对象的获得交给第三方（系统）来控制**，**即依赖对象不再被依赖模块的类中直接通过new来获取；在Spring中，IOC主要是完成对象的创建和依赖的管理注入，Ioc意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制，所谓控制反转，就是把原先在对象内部那些直接对对象创建、依赖的操作反交给容器来帮忙实现

  ```
  即将创建对象、属性值的方式进行了反转转，从原本new、setXxx()  转变成了为了从springIOC容器getBean()无论要什么对象，都可以直接去springioc容器中获取，而不需要自己操作（new\setXxx()）所以要先给springioc中存放对象并赋值，然后就可以直接使用了
  ```

+ AOP：面向对象是不同对象负责不同模块，分工明确，但不同模块会有一些公共行为（安全、日志、事务、性能，这种公共的行为很难通过继承的方式来实现，如果用工具类的话也不利于维护，代码也显得异常繁琐）切面（AOP）的引入就是为了解决这类问题而生的，它要达到的效果是保证开发者在**不修改源代码**的前提下，为系统中不同的业务组件添加某些通用功能

  + **JoinPoint**: 程序在执行流程中经过的一个个时间点，这个时间点可以是方法调用时，或者是执行方法中异常抛出时，也可以是属性被修改时等时机，在这些时间点上你的切面代码是**可以**（注意是可以但未必）被注入的。

  + **Pointcut**: JoinPoints 只是切面代码可以被织入的地方，但我并不想对所有的 JoinPoint 进行织入，这就需要某些条件来筛选出那些需要被织入的 JoinPoint，Pointcut 就是通过一组规则(使用 AspectJ pointcut expression language 来描述) 来定位到匹配的 joinpoint

  + **Advice**:  代码织入（也叫增强），Pointcut 通过其规则指定了哪些 joinpoint 可以被织入，而 Advice 则指定了这些 joinpoint 被织入（或者增强）的具体时机与逻辑，是**切面代码真正被执行的地方**，主要有五个织入时机：

    1. Before Advice: 在 JoinPoints 执行前织入；
    2. After Advice: 在 JoinPoints 执行后织入（不管是否抛出异常都会织入）；
    3. After Returning Advice: 在 JoinPoints 执行正常退出后织入（抛出异常则不会被织入）；
    4. After Throwing Advice: 方法执行过程中抛出异常后织入；
    5. Around Advice: 这是所有 Advice 中最强大的，它在 JoinPoints 前后都可织入切面代码，也可以选择是否执行原有正常的逻辑，如果不执行原有流程，它甚至可以用自己的返回值代替原有的返回值，甚至抛出异常。在这些 advice 里我们就可以写入切面代码了。综上所述，切面（Aspect）我们可以认为就是 pointcut 和 advice，pointcut 指定了哪些 joinpoint 可以被织入，而 advice 则指定了在这些 joinpoint 上的代码织入时机与逻辑。

    *画外音：织入（weaving），将切面作用于委托类对象以创建 adviced object 的过程（即代理，下文会提）。*

  统计每个方法的执行时间，如果不用切面你需要在每个方法的首尾计算下时间，然后相减：可维护性差，业务代码与非业务代码严重耦合

  ```java
  package com.example.demo.annotation
      
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.METHOD)
  public @interface GlobalErrorCatch {
  
  }
  ```

  然后将所有 service 中方法里的在方法签名上加上上述我们定义好的注解

  ```java
  public class TestServiceImpl implements TestService {
     @Override
     @GlobalErrorCatch
     public ServiceResultTO<Boolean> test() {
          // 此处写服务里的执行逻辑
          boolean result = xxx;
          return ServiceResultTO.buildSuccess(result);
     }
  }
  ```

  然后再指定注解形式的 pointcuts 及 around advice

  ```java
  @Aspect
  @Component
  public class TestAdvice {
     // 1. 定义所有带有 GlobalErrorCatch 的注解的方法为 Pointcut
    @Pointcut("@annotation(com.example.demo.annotation.GlobalErrorCatch)")
     private void globalCatch(){}
     // 2. 将 around advice 作用于 globalCatch(){} 此 PointCut 
      
     @Around("globalCatch()")
     public Object handlerGlobalResult(ProceedingJoinPoint point){
             Long startTime = 当前时间戳
             return point.proceed();
             Long afterTime = 当前时间戳
             Long resultTime = afterTime -startTime
     }
  
  }
  ```

  代理在生活中随处可见，比如说我要买房，我一般不会直接和卖家对接，一般会和中介打交道，中介就是代理，卖家就是目标对象，我就是调用者，代理不仅实现了目标对象的行为（帮目标对象卖房），还可以添加上自己的动作（收保证金，签合同等）

  代理主要分为两种类型：静态代理和动态代理，动态代理又有 JDK 代理和 CGLib 代理两种,首先 Java 源代码经过编译生成字节码，然后再由 JVM 经过类加载、连接，初始化成 Java 类型，可以看到字节码是关键，静态和动态的区别就在于字节码生成的时机。**静态代理**：由程序员创建代理类或特定工具自动生成源代码再对其编译。在编译时已经将接口、被代理类（委托类）、代理类等确定下来，在**程序运行前**代理类的.class文件就已经存在了。**动态代理**：在**程序运行后**通过反射创建生成字节码再由 JVM 加载而成

  ```java
  // 委托类
  public class RealSubject implements Subject {
     @Override
     public void request() {
         // 卖房
         System.out.println("卖房");
     }
  }
  
  
  import java.lang.reflect.InvocationHandler;
  import java.lang.reflect.Method;
  import java.lang.reflect.Proxy;
  
  public class ProxyFactory {
  
     private Object target;// 维护一个目标对象
  
     public ProxyFactory(Object target) {
         this.target = target;
     }
  
     // 为目标对象生成代理对象
     public Object getProxyInstance() {
         return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(),
                 new InvocationHandler() {
  
                     @Override
                     public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                         System.out.println("计算开始时间");
                         // 执行目标对象方法
                         method.invoke(target, args);
                         System.out.println("计算结束时间");
                         return null;
                     }
                 });
     }
  
     public static void main(String[] args) {
         RealSubject realSubject = new RealSubject();
         System.out.println(realSubject.getClass());
         Subject subject = (Subject) new ProxyFactory(realSubject).getProxyInstance();
         System.out.println(subject.getClass());
         subject.request();
     }
  }```
  打印结果如下:
  ​```shell
  原始类:class com.example.demo.proxy.staticproxy.RealSubject
  代理类:class com.sun.proxy.$Proxy0
  计算开始时间
  卖房
  计算结束时间
  ```

  

  ```java
  public static Object newProxyInstance(ClassLoader loader,
                                           Class<?>[] interfaces,
                                           InvocationHandler h);
  ```

  > 1. **loader**: 代理类的ClassLoader，最终读取动态生成的字节码，并转成 java.lang.Class 类的一个实例（即类），通过此实例的 newInstance() 方法就可以创建出代理的对象。
  > 2. **interfaces**: 委托类实现的接口，JDK 动态代理要实现所有的委托类的接口。
  > 3. **InvocationHandler**: 委托对象所有接口方法调用都会转发到 InvocationHandler.invoke()，在 invoke() 方法里我们可以加入任何需要增强的逻辑，主要是根据委托类的接口等通过反射生成的。

由于动态代理是程序运行后才生成的，哪个委托类需要被代理到，只要生成动态代理即可，避免了静态代理那样的硬编码，另外所有委托类实现接口的方法都会在 Proxy 的 InvocationHandler.invoke() 中执行，这样如果要统计所有方法执行时间这样相同的逻辑，可以统一在 InvocationHandler 里写， 也就避免了静态代理那样需要在所有的方法中插入同样代码的问题，代码的可维护性极大地提高了。

JDK 动态代理是通过与委托类实现同样的接口，然后在实现的接口方法里进行增强来实现的，这就意味着**如果要用 JDK 代理，委托类必须实现接口**

CGlib 动态代理也提供了类似的  Enhance 类，增强逻辑写在 MethodInterceptor.intercept() 中，也就是说所有委托类的**非 final 方法**都会被方法拦截器拦截，从而在拦截器里实现增强

```java
public class MyMethodInterceptor implements MethodInterceptor {
   @Override
   public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
       System.out.println("目标类增强前！！！");
       //注意这里的方法调用，不是用反射哦！！！
       Object object = proxy.invokeSuper(obj, args);
       System.out.println("目标类增强后！！！");
       return object;
   }
}

public class CGlibProxy {
   public static void main(String[] args) {
       //创建Enhancer对象，类似于JDK动态代理的Proxy类，下一步就是设置几个参数
       Enhancer enhancer = new Enhancer();
       //设置目标类的字节码文件
       enhancer.setSuperclass(RealSubject.class);
       //设置回调函数
       enhancer.setCallback(new MyMethodInterceptor());

       //这里的creat方法就是正式创建代理类
       RealSubject proxyDog = (RealSubject) enhancer.create();
       //调用代理类的eat方法
       proxyDog.request();
   }
}
```

它是通过**继承**自委托类，重写委托类的非 final 方法（final 方法不能重载），并在方法里调用委托类的方法来实现代码增强的，它的实现大概是这样：

```java
public class RealSubject {
   @Override
   public void request() {
       // 卖房
       System.out.println("卖房");
   }
}

/** 生成的动态代理类（简化版）**/
public class RealSubject$$EnhancerByCGLIB$$889898c5 extends RealSubject {
   @Override
   public void request() {
       System.out.println("增强前");
       super.request();
       System.out.println("增强后");
   }
}
```

只能代理委托类中任意的非 final 的方法，另外它是通过继承自委托类来生成代理的，所以如果委托类是 final 的，就无法被代理了（final 类不能被继承）



JDK 动态代理的拦截对象是通过反射的机制来调用被拦截方法的，由于反射的效率比较低，所以 CGlib 采用了 FastClass 的机制来实现对被拦截方法的调用。FastClass 机制就是对一个类的方法建立索引，通过索引来直接调用相应的方法



bean组件的作用域

Spring的声明式事务管理

spring的事务管理方式，注解Transactional的底层怎么实现

spring事务，隔离机制 

spring事务的传播机制 

spring事务配置和隔离级别 

面向切面编程举一个实例 

切面到底是什么

spring用过哪些注解，具体讲一下 

springboot特性，自动配置原理 

spring用了什么设计模式 

spring bean加载过程，怎么传入构造器参数，怎么开启事务，beanfactory和factorybean

springboot常用注解，说一下SpringBoot和Spring的区别

```
Spring Boot基本上是Spring框架的扩展，它消除了设置Spring应用程序所需的复杂例行配置。

它的目标和Spring的目标是一致的，为更快，更高效的开发生态系统铺平了道路。以下是Spring Boot中的一些功能：

通过starter这一个依赖，以简化构建和复杂的应用程序配置。
可以直接main函数启动，嵌入式web服务器，避免了应用程序部署的复杂性，Metrics度量，Helth check健康检查和外部化配置。
尽可能的自动化配置Spring功能。
```

SpringBoot加载配置文件的优先级

`file:./config/`  - 优先级最高（项目根路径下的`config`）

`file:./`  - 优先级第二  -（项目根路径下）

`classpath:/config/`  - 优先级第三（项目`resources/config`下）

`classpath:/`  - 优先级第四（项目`resources`根目录）

- `高优先级配置`会覆盖`低优先级配置`
- 多个配置文件`互补`

properties>yaml>yml

项目打包运行后可通过命令指定配置文件位置

> --spring.config.location=d:/application.properties

application-{profile}.properties/yml

```
server:
  port: 8081
spring:
  profiles:
    active: prod #指定使用哪个环境

---
server:
  port: 8083
spring:
  profiles: dev  #指定属于哪个环境
```

>java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev

springboot自动装配

springboot原理

注解@SpringBootApplication实现原理 

@SpringBootApplication 这个注解去看下，@SpringBootApplication 注解主要由三个注解组成：@ComponentScan、@SpringBootConfiguration、**@EnableAutoConfiguration**。

```
@Target({ElementType.TYPE}) //接口、类、枚举、注解
@Retention(RetentionPolicy.RUNTIME) //这种类型的Annotations将被JVM保留,所以他们能在运行时被JVM或其他使用反射机制的代码所读取和使用。
@Documented // 说明该注解将被包含在javadoc中
@Inherited // 说明子类可以继承父类中的该注解
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
 		...
}
```

**@EnableAutoConfiguration**

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage // 把使用了该注解的类所在的包及子包下所有组件扫描到Spring IOC容器中
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}

```

AutoConfigurationImportSelector

```
public class AutoConfigurationImportSelector implements DeferredImportSelector,***{}
```

```
public interface DeferredImportSelector extends ImportSelector {}
```

ImportSelector:selectImports：返回一个数组（一些类路径），Spring会把数组中的类全部加载并注入springIOC容器中

```
public interface ImportSelector {

	String[] selectImports(AnnotationMetadata importingClassMetadata);

}
```





@Pathvarible和@RequestParam的区别

@RequestParam和@PathVariable都能够完成类似的功能——因为本质上，它们都是用户的输入，只不过输入的部分不同，一个在URL路径部分，另一个在参数部分。要访问一篇博客文章，这两种URL设计都是可以的：

- 通过@PathVariable，例如/blogs/1
- 通过@RequestParam，例如blogs?blogId=1

那么究竟应该选择哪一种呢？建议：

1、当URL指向的是某一具体业务资源（或资源列表），例如博客，用户时，使用@PathVariable

2、当URL需要对资源或者资源列表进行过滤，筛选时，用@RequestParam

什么是SpringMVC?



spring mvc拦截器 过滤器 

| 依赖         | Servlet容器                                                  | Spring Web                                                   | Spring                                       |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------------- |
| 基于实现     | 回调机制                                                     | 反射机制(AOP思想)                                            | 动态代理                                     |
| 类别         | Filter                                                       | Interceptor                                                  | AOP                                          |
| 实现方式     | 实现接口Filter                                               | 实现接口HandlerInterceptor                                   | 注解@Aspect                                  |
| 作用范围     | 所有URL请求(可过滤)                                          | 所有Controller的action 包括自己定义的和其他组件定义的        | spring的bean(可过滤)                         |
| 可操作数据   | 原始Http请求信息： ServletRequest  request, ServletResponse response | (1)Http请求信息： HttpServletRequest request,  HttpServletResponse response,  (2)springMvc执行的方法信息： HandlerMethod handlerMethod  (3)返回结果(执行Action方法后，不报错)： ModelAndView modelAndView  (4)异常信息(执行Action方法后)： Exception ex | 请求参数 返回结果 异常信息                   |
| 不可操作数据 | 执行方法相关信息                                             | ResponseBody的返回结果                                       | http请求信息                                 |
| 相关方法     | doFilter                                                     | preHandle postHandle afterCompletion@                        | @Aspect @Pointcut @Before @After @Around     |
| 用途         | 字符编码， 鉴权操作， 防重复提交 记录执行时间， 脱敏信息、 过滤敏感词、 多租户切换 ...... | 字符编码 鉴权操作 防重复提交 异常记录 ......                 | 日志记录 异常记录 数据源切换 请求埋点 ...... |

基于SpringBoot的web程序，Filter、Interceptor、Aop的请求顺序如下：

**Filter->Interceptor->AOP->Controller**



SpringBoot依赖包的x,y坐标说一下？ 

Mybatis的自增主键怎么设置，如何设置表的关联？

Mybatis吗?能讲一下什么是JPA吗？

mybatis相关，怎么保证跨库调用数据的一致性.... 

Mybatis的底层实现

一个Java实例插入到数据库，mybatis的过程 

mybatis一级缓存和二级缓存 

mybatis的对象映射持久化

MyBatis中#和$区别，哪个安全 

MyBatis二级缓存 



