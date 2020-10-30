### Java基础

#### 概念

​		Java是一门面向对象编程语言，以对象为基本单位，包含属性与方法，对象的状态使用属性来表达，对象的行为用方法表达。Java是跨平台的，java源代码经编译后生成的二进制字节码是与平台无关的，通过Java虚拟机可以将字节码文件解释成具体平台上的机器指令去执行，从而实现一次编译，到处运行。Java语言是健壮的，Java中有强类型机制、异常处理、垃圾自动回收和安全检查机制使得java程序更具健壮性。Java支持多个线程同时执行，并提供多线程之间的同步机制；Java是安全的，Java提供了一个安全机制以防止恶意代码的攻击，Java提供了一种可定制的"沙箱"(sandbox)安全模型，java程序只能在沙箱内运行，保证对系统或用户无潜在风险。

+ JDK（Java Development Kit）主要用于java程序的开发，包括了Java运行环境JRE、Java工具和Java基础类库。（JRE+Java的开发工具）
+ Java Runtime Environment（JRE）主要是用于运行JAVA程序，它是Java程序运行所必须的环境的集合，包含JVM标准实现及Java核心类库。（JVM+Java语言的核心类库Lib ）
+ Java Virtual Machine（JVM）主要用于解析字节码文件，它需要借助JRE下的lib类库才可以执行字节码文件（将字节码文件转成具体系统平台的机器指令）

通常的实际开发情况是：利用JDK（调用JAVA API）开发JAVA程序，然后通过JDK中的编译程序（javac）将java源代码编译成JAVA字节码，接着在JRE上运行这些JAVA字节码，运行过程中JVM解析字节码文件中的指令，映射到CPU指令集或OS的系统调用完成程序的正常运行。

##### 基本数据类型

+ 整数型：byte、short、int、long
+ 浮点型：float、double
+ 字符型：char(2个字节)
+ 布尔型：boolean

在 Java 语言中，布尔类型的值不能转换成任何数据类型，true 常量不等于 1，而 false 常量也不等于 0。这两个值只能赋给声明为 boolean 类型的变量，或者用于布尔运算表达式中。java对于-128到127之间的数，会进行缓存，Integer i = 100 在编译时，会成为Integer i = Integer.valueOf(100)，会从缓存中寻找数据；int 变量 与 Integer、 new Integer() 比较时，只要两个的值是相等，则为true,包装类Integer 和 基本数据类型int 比较时，java会自动拆包装为int ，然后进行比较，实际上就变为两个int变量的比较。

```java
public static Integer valueOf(int i){
    assert IntegerCache.high >= 127;
    if (i >= IntegerCache.low && i <= IntegerCache.high){
        return IntegerCache.cache[i + (-IntegerCache.low)];
    }
    return new Integer(i);
}
```

+ 强类型：Java是一种强类型语言，每个变量都有一个类型，每个表达式都有一个类型，每个类型都严格定义；检查所有分配的类型兼容性。没有自动强制或有冲突类型的转换。Java编译器检查所有表达式和参数，以确保类型是兼容的。

##### 封装/继承/多态

+ 封装：将类的属性、方法等实现细节隐藏在类内部，仅对外提供部分允许访问的方法，使用者不必了解具体的实现细节。这样可以提高数据安全性，防止数据被恶意破坏。通常是将成员变量访问修饰设为private,并提供公有的setter和getter方法。

+ 继承：子类继承父类的特征和行为，使得子类具有父类的一些属性和方法。在继承关系中，父类更通用、子类更具体。父类具有更一般的特征和行为，而子类除了具有父类的特征和行为，还具有一些自己特殊的特征和行为。这样可以实现代码重用(非静态属性和方法可以被继承和重写。但是静态属性和方法可以被继承，但是没有被重写(overwrite)而是被隐藏，即调用的都是父类的属性和方法；创建子类对象时默认会先调用父类无参的构造函数。)

  + 普通类可以继承（extends）普通类，可以继承（extends）抽象类，可以继承（implements）接口。
  + 抽象类可以继承（extends）普通类，可以继承（extends）抽象类，可以继承（implements）接口。
  + 接口只能继承（extends）接口。
  + 一个普通类继承一个抽象类后，必须实现这个抽象类中定义的所有抽象（abstract）方法，否则就只能被定义为抽象类。
  + 一个普通类继承一个接口后，必须实现这个接口中定义的所有方法，否则就只能被定义为抽象类。
  + 抽象类继承抽象类，或者实现接口时，可以部分、全部或者完全不实现父类抽象类的抽象（abstract）方法或父类接口中定义的方法。

+ 多态：一个类实例的相同方法在不同情形有不同表现形式，比如同一个实现接口，使用不同的实例而执行不同的操作。这样可以增强程序的可扩展性及可维护性。Java中实现多态的方式有基于继承和基于接口。基于继承实现的多态主要表现在父类和继承该父类的一个或多个子类对某些方法的重写，多个子类对同一方法的重写可以表现出不同的行为；基于接口实现的多态，在接口的多态中，指向接口的引用必须是指定这实现了该接口的一个类的实例，在运行时，根据对象引用的实际类型来执行对应的方法。

  多态的三个条件

  + 存在继承，子类继承父类
  + 子类重写父类的方法
  + 父类引用变量指向子类对象

##### 重写与重载

+ 重写：发生在有继承关系的子类中，子类中定义了一个方法，其方法名、返回值类型、参数列表与父类中某个方法一样，此时就是子类中重写(覆盖)了父类的同名方法

  方法重写时，方法名与形参列表必须一致；子类的权限修饰符必须要大于或者等于父类的权限修饰符；子类的返回值类型必须要小于或者，等于父类的返回值类型；子类抛出的异常类型要小于或者等于父类抛出的异常类型。

+ 重载：发生在同一个类中，存在多个方法的方法名相同，但是参数列表不同（参数个数、参数类型或者参数的顺序不同）

##### 修饰符权限

private 只能用于修饰内部类

|           | 同一类 | 同包中的子类 | 同包中的非子类 | 不同包的子类 | 不同包的非子类 |
| --------- | ------ | ------------ | -------------- | ------------ | -------------- |
| public    | √      | √            | √              | √            | √              |
| protected | √      | √            | √              | √            |                |
| default   | √      | √            | √              |              |                |
| private   | √      |              |                |              |                |

##### 抽象类与接口

+ 抽象类：如果一个类含有抽象方法，则称这个类为抽象类，抽象类必须在类前用abstract关键字修饰；抽象类就是为了继承而存在的，如果你定义了一个抽象类，却不去继承它，那么等于白白创建了这个抽象类。抽象类和普通类一样，同样可以拥有成员变量和普通的成员方法。注意，抽象类和普通类的主要有三点区别：
  1. 抽象方法必须为public或者protected（因为如果为private，则不能被子类继承，子类便无法实现该方法），缺省情况下默认为public。
  2. 抽象类不能用来创建对象
  3. 如果一个类继承于一个抽象类，则子类必须实现父类的抽象方法。如果子类没有实现父类的抽象方法，则必须将子类也定义为为abstract类。
+ 接口：接口是对行为的抽象。接口中可以含有变量和方法。但接口中的所有变量会被隐式地指定为public static final变量，而所有方法也会被隐式地指定为public abstract方法（用其他关键字，比如private、protected、static、 final等修饰会报编译错误），即接口中的方法必须都是抽象方法。
+ 区别：抽象类中抽象方法的访问权限可以是public、protected；接口只能是public；抽象类中可以有非抽象方法；接口中的方法都是抽象方法；抽象类中可以定义普通成员变量；接口中不能定义普通成员变量，接口中只能定义常量；一个类只能继承一个抽象类；但一个类可以实现多个接口，接口可以继承多个接口(类是单继承)

##### 异常机制

​		在 Java 中，通过Throwable以及它的相关子类来描述各种不同的异常类型。Throwable有两个重要的子类：Exception（异常）和 Error（错误），二者都是 Java 异常处理的重要子类，各自都包含大量子类。异常能被程序本身可以处理，错误是无法处理。

+ Error：程序无法处理的错误，表示运行应用程序中较严重问题，会直接退出JVM。大多数错误与代码编写者执行的操作无关，而表示代码运行时 JVM（Java 虚拟机）出现的问题。常见的有虚拟机错误、内存溢出、线程死锁等，这些错误往往是不可查的，因为它们在应用程序的控制和处理能力之外，而且绝大多数是程序运行时不允许出现的状况。(StackOverFlowError、OutOfMemonyError)
+ Exception：程序本身可以处理的异常，如果没处理也会退出JVM。异常处理通常指的是针对这类异常的处理。Exception类的异常包括Unchecked Exception和Checked Exception。
  + Checked Exception(检查异常)：即编译时异常，要求必须在编译时，除了RuntimeException以及它的相关子类，其他的Exception子类都是可检查异常，如IOException、SQLException。当程序中出现这类异常时，要么用try-catch语句捕获它，要么用throws子句声明抛出它，否则编译不会通过。
  + Unchecked Exception(非检查异常)：即运行时异常，RuntimeException以及它的相关子类均属于这种异常，不要求强制去处理。

在 Java 应用程序中，异常处理机制为：抛出异常throws，捕捉异常try-catch-finally。

+ throws：方法可能抛出异常的声明。在方法声明处用throws子句来声明抛出异常。throws 可以理解成是一种通知行为，没有实际的抛出异常的动作，而仅仅是告诉调用他的上层函数，这里可能会抛出这个异常。当方法抛出异常时，方法将不对这些类型及其子类类型的异常作处理，而抛向调用该方法的方法，由他去处理，而是推卸责任
+ throw：明确抛出一个异常。throw总是出现在函数体中，用来抛出一个Throwable类型的异常。表示抛出一个实际的异常的实际动作，如果在函数内没有捕获并处理，那么将会一直向上抛出这个异常直到被main()/Thread.run()抛出。
+ finally：异常机制总是保证finally块总是被执行。只有finally块，执行完成之后，才会回来执行try或者catch块中的return或者throw语句，如果finally中使用了return或者   throw等终止方法的语句，则就不会跳回执行，直接停止。finally块中永远不要写return语句，因为finally块中总是最后执行，他会改变预期在try和catch块中的返回值（举个例子，你在catch中捕获了一个异常并抛出e，又在finally语句中return true,这样你抛出的异常就"消失"了，因为当前函数的执行结果已经从抛出异常 转变成 return true）。另外，在使用资源对象与流对象时，finally块必须对资源对象、流对象进行关闭。

RuntimeException子类

```
1.java.lang.ArrayIndexOutOfBoundsException:数组索引越界异常。当对数组的索引值为负数或大于等于数组大小时抛出。
2.java.lang.NullPointerException:空指针异常。当应用试图在要求使用对象的地方使用了null时，抛出该异常。譬如：调用null对象的实例方法、访问null对象的属性、计算null对象的长度、使用throw语句抛出null等等
3.java.lang.ArithmeticException:算术条件异常。譬如：整数除零等。
4.java.lang.ClassNotFoundException:找不到类异常。当应用试图根据字符串形式的类名构造类，而在遍历CLASSPAH之后找不到对应名称的class文件时，抛出该异常。
```

IOException

```
1.IOException：操作输入流和输出流时可能出现的异常。
2.EOFException   文件已结束异常
3.FileNotFoundException   文件未找到异常
```

用户自定义异常类，只需继承Exception类即可

1. 创建自定义异常类：一般会选择继承Exception和RuntimeException，如果不要求调用者一定要处理抛出的异常，就继承RuntimeException。
2. 抛出自定义异常：在方法中通过throw关键字抛出异常对象。
3. 捕获自定义异常：如果在当前抛出异常的方法中处理异常，可以使用try-catch语句捕获并处理；否则在方法的声明处通过throws关键字指明要抛出给方法调用者的异常，继续进行下一步操作。

##### File

##### instanceof

​		instanceof关键字的使用前提：判断的对象与指定的类别必须要存在继承或者实现的关系；instanceof关键字的作用：判断一个对象是否属于指定的类别。一般做强制类型转换之前都会使用该关键字先判断一把，然后再进行转换的。

```java
对象 instanceof 类别
```

##### final关键字

​		final关键字修饰一个类的时候，该类不能被继承，并且final类中的所有成员方法都会被隐式地指定为final方法，final类中的成员变量可以根据需要设为final。一般来说工具类我们往往都会设计成为一个fianl类。在JDK中，被设计为final类的有String、System等。所有包装类如Integer，Float等都是final类。

​		final关键字修饰方法时，该方法不能被重写。一个类的private方法会隐式的被指定为final方法。使用final方法的原因有两个：第一个原因是把方法锁定，以防任何继承类修改它的含义，不能被重写。第二个原因是效率，final方法比非final方法要快，因为在编译的时候已经静态绑定了，不需要在运行时再动态绑定。

​		fianl关键字修饰一个引用类型变量时，在对其初始化之后便不能再让其指向另一个对象，即不能重新指向另一个对象(否则会出现编译异常)。final关键字修饰一个基本类型的变量时，其数值一旦在初始化之后便不能更改，即不能被重新赋值(否则会出现编译)。被fianl修饰的成员变量赋值，有两种方式：直接赋值 、全部在构造方法中赋初值

​		String能被设计成不可变类型的一个重要前是因为它是编程语言里面使用频率最高的一种类型。不可变类型带来的好处，体现在四个方面，分别是：缓存，安全，同步和性能。在JVM的运行时数据区域里面，有一个专门的字符串常量池用来存储字符串字面量，当第一个字面量声明的时候，它的值会被字符串常量池存储，当s2变量声明的时候，jvm发现常量池已经存在该对象，所以就不会再创建一次，而是直接将一样的内存指针赋值给s2变量，从避免了重复创建对象，节省了内存空间。此外，由于字符串的不可变性，从而可以让其hashCode也被缓存，在Java里面哈希类数据结构如HashMap, HashTable, HashSet其key用的最多的基本都是String类型，如此一来key的hashCode的也可以在第一次调用之后被缓存，之后直接使用无须重新生成，从而间接的提升访问效率。

##### static

​		static方便在没有创建对象的情况下来进行调用（方法/变量），被static关键字修饰的方法或者变量不需要依赖于对象来进行访问，只要类被加载了，就可以通过类名去进行访问。静态变量被所有的对象所共享，在内存中只有一个副本，它当且仅当在类初次加载时会被初始化。在类初次被加载的时候，会按照static块的顺序来执行每个static块，并且只会执行一次。

##### String/StringBuffer/StringBuilder

+ 可变性：String 是不可变的对象, 每次对 String 进行修改时，其实会生成了一个新的 String 对象，所以经常需要改变内容的字符串最好不要用 String ，不用每次生成新对象，这会占用内存，会对系统性能产生影响。StringBuffer 和 StringBuilder是可变的，StringBuffer每次结果都会对 StringBuffer 对象本身进行操作，而不是生成新的对象
+ 线程安全性：String是不可变的，保证了线程的安全性；StringBuilder是线程不安全的；StringBuffer由于有synchronized修饰，所以是线程安全的
+ 执行速度：StringBuilder >  StringBuffer  >  String

String，StringBuffer，StringBuilder，都是final类，不允许被继承。如果要操作少量的数据，用String ；单线程操作大量数据，用StringBuilder ；多线程操作大量数据，用StringBuffer。不要使用String类的”+”来进行频繁的拼接，因为那样的性能极差的，应该使用StringBuffer或StringBuilder类，这在Java的优化上是一条比较重要的原则

String 类不可变，内部维护的char[] 数组长度不可变，为final修饰，String类也是final修饰，不存在扩容。字符串拼接，截取，都会生成一个新的对象。频繁操作字符串效率低下，因为每次都会生成新的对象。

StringBuilder 类内部维护可变长度char[] ， 初始化数组容量为16，存在扩容， 其append拼接字符串方法内部调用System的native方法，进行数组的拷贝，不会重新生成新的StringBuilder对象。非线程安全的字符串操作类， 其每次调用 toString方法而重新生成的String对象，不会共享StringBuilder对象内部的char[]，会进行一次char[]的copy操作。

StringBuffer 类内部维护可变长度char[]， 基本上与StringBuilder一致，但其为线程安全的字符串操作类，大部分方法都采用了Synchronized关键字修改，以此来实现在多线程下的操作字符串的安全性。其toString方法而重新生成的String对象，会共享StringBuffer对象中的toStringCache属性（char[]），但是每次的StringBuffer对象修改，都会置null该属性值。

##### 反射机制

​		在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法，都能调用它的任意一个方法和属性。反射就是把java类中的各种成分映射成一个个的Java对象，一个类有：成员变量、方法、构造方法、包等等信息，利用反射技术可以对一个类进行解剖，把个个组成部分映射成一个个对象。通过反射可以在程序在运行时获得某个类的Class实例（一个类在 JVM 中只会有一个 Class 实例），然后进一步获取其内部信息包括它modifiers(修饰符)，fields(属性)，methods(方法)等，并且可在运行时改变fields内容或调用methods。这样可以灵活编码，使得代码可以在运行时装配，无需在组件之间进行源代码链接，降低代码的耦合度，另外还可以实现动态代理。但是，使用反射的性能较低 ，使用反射相对来说不安全 ，并且会破坏类的封装性，可以通过反射获取这个类的私有方法和属性 

| 类名          | 用途                                                         |
| ------------- | ------------------------------------------------------------ |
| Class类       | 代表类的实体（即类和接口），一个类在一个JVM中值存在一个Class实例 |
| Field类       | 代表类的成员变量                                             |
| Method类      | 代表类的方法                                                 |
| Constructor类 | 代表类的构造方法                                             |

+ Class

| 方法                                                         | 用途                                               |
| ------------------------------------------------------------ | -------------------------------------------------- |
| getClassLoader()                                             | 获得类的加载器                                     |
| forNames(String className)                                   | 根据全类名返回类的Class实例                        |
| newInstance()                                                | 创建类的实例                                       |
| getInterfaces()                                              | 获取当前类实现的接口                               |
| getDeclaredFields()                                          | 获取当前类的所有属性                               |
| getDeclaredField(String fieldName)                           | 获得当前类的filedName属性                          |
| getDeclaredConstructors()                                    | 获取当前类的所有构造方法                           |
| getDeclaredConstructor(Class...<?> parameterTypes)           | 获得当前类与参数类型匹配的构造方法（根据参数类型） |
| getDeclaredMethods()                                         | 获得当前类的所有方法                               |
| getDeclaredMethod(String methodName, Class...<?> parameterTypes) | 获得当前类与方法名和参数类型匹配的方法             |

+ Field

| 方法                         | 用途                    |
| ---------------------------- | ----------------------- |
| set(Object obj,Object value) | 设置obj中对应属性值     |
| get(Object obj)              | 获得obj中对应的属性值   |
| equals(Object obj)方法       | 属性与obj相等则返回true |

+ Method

| 方法                               | 用途                                     |
| ---------------------------------- | ---------------------------------------- |
| invoke(Object obj, Object... args) | 传递object对象及参数调用该对象对应的方法 |

+ Constructor

| 方法                            | 用途                       |
| ------------------------------- | -------------------------- |
| newInstance(Object... initargs) | 根据传递的参数创建类的对象 |

##### JDK/CGLib动态代理

+ 区别

  >  JDK动态代理是实现了被代理对象的接口，Cglib是继承了被代理对象。
  >
  > JDK和Cglib都是在运行期生成字节码，JDK是直接写Class字节码，Cglib使用ASM框架写Class字节码，Cglib代理实现更复杂，生成代理类比JDK效率低。
  >
  > JDK调用代理方法，是通过反射机制调用，Cglib是通过FastClass机制直接调用方法，Cglib执行效率更高。

###### JDK动态代理

+ JDK动态代理：基于拦截器和反射来实现（必须实现InvocationHandler接口、使用Proxy.newProxyInstance产生代理对象、被代理的对象必须要实现接口）

1. 通过实现InvocationHandler接口来自定义自己的InvocationHandler
2. 通过Proxy.getProxyClass获得动态代理类
3. 通过反射机制获得代理类的构造方法，方法签名为getConstructor(InvocationHandler.class)
4. 通过构造函数获得代理对象并将自定义的InvocationHandler实例对象传为参数传入
5. 通过代理对象调用目标方法

```java
package com.jpeony.spring.proxy.jdk;
 
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
 
public class MyInvocationHandler implements InvocationHandler {
 
    /** 目标对象 */
    private Object target;
 
    public MyInvocationHandler(Object target){
        this.target = target;
    }
 
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("------插入前置通知代码-------------");
        // 执行相应的目标方法
        Object rs = method.invoke(target,args);
        System.out.println("------插入后置处理代码-------------");
        return rs;
    }
}
```

```java
package com.jpeony.spring.proxy.jdk;
 
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Proxy;
 
/**
 * 使用JDK动态代理的五大步骤:
 * 1.通过实现InvocationHandler接口来自定义自己的InvocationHandler;
 * 2.通过Proxy.getProxyClass获得动态代理类
 * 3.通过反射机制获得代理类的构造方法，方法签名为getConstructor(InvocationHandler.class)
 * 4.通过构造函数获得代理对象并将自定义的InvocationHandler实例对象传为参数传入
 * 5.通过代理对象调用目标方法
 */
public class MyProxyTest {
    public static void main(String[] args)
            throws NoSuchMethodException, IllegalAccessException, InstantiationException, InvocationTargetException {
        // =========================第一种==========================
        // 1、生成$Proxy0的class文件
      System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
        // 2、获取动态代理类
        Class proxyClazz = Proxy.getProxyClass(IHello.class.getClassLoader(),IHello.class);
        // 3、获得代理类的构造函数，并传入参数类型InvocationHandler.class
        Constructor constructor = proxyClazz.getConstructor(InvocationHandler.class);
        // 4、通过构造函数来创建动态代理对象，将自定义的InvocationHandler实例传入
        IHello iHello1 = (IHello) constructor.newInstance(new MyInvocationHandler(new HelloImpl()));
        // 5、通过代理对象调用目标方法
        iHello1.sayHello();
        
 
        // ==========================第二种=============================
        /**
         * Proxy类中还有个将2~4步骤封装好的简便方法来创建动态代理对象，
         *其方法签名为：newProxyInstance(ClassLoader loader,Class<?>[] instance, InvocationHandler h)
         */
        IHello  iHello2 = (IHello) Proxy.newProxyInstance(IHello.class.getClassLoader(), // 加载接口的类加载器
                new Class[]{IHello.class}, // 一组接口
                new MyInvocationHandler(new HelloImpl())); // 自定义的InvocationHandler
        iHello2.sayHello();
    }
}
```

简易版JDK动态代理

```java
public interface IHello {
    void sayHello();
    void sayThankyou();
    void sayBye();
}
```

```java
public class HelloImpl implements IHello {
    @Override
    public void sayHello() {
        System.out.println("Hello World");
    }

    @Override
    public void sayThankyou() {
        System.out.println("Thankyou World");

    }

    @Override
    public void sayBye() {
        System.out.println("GoodBye World");

    }
}
```

```java
public  class JDKProxyUtil implements InvocationHandler {

    //目标对象
    private Object target;

    //产生代理对象
    public  Object createProxy(Object target){
        this.target =target;
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),target.getClass().getInterfaces(),this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        checkPopedom();
        System.out.println("ABCJ");
        Object rs = method.invoke(target,args);
        return rs;
    }

    /**
     * 模拟检查权限的例子
     */
    private void checkPopedom() {
        System.out.println("======检查权限checkPopedom()======");
    }
}
```

Proxy.newProxyInstance方法创建一个动态代理类实例$Proxy0，$Proxy0重写了Object中的hashCode(),equals(),toString()三个方法以及目标接口的方法，每次调用这些方法时，通过调用自定义InvocationHandler实现类的invoke方法。简单的来说就是，JDK动态代理技术可以为一组接口生成代理类，这个代理类只是是一层壳，简单实现了接口中定义的方法，实际是通过其成员实例InvocationHandler的invoke方法去调用目标对象的方法并做相应的增强处理。

```java
import com.iwtrtw.dp.test.proxy.IHello;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy0 extends Proxy implements IHello {
    private static Method m1;
    private static Method m4;
    private static Method m2;
    private static Method m3;
    private static Method m5;
    private static Method m0;
    
        static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m4 = Class.forName("com.iwtrtw.dp.test.proxy.IHello").getMethod("sayHello");
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m3 = Class.forName("com.iwtrtw.dp.test.proxy.IHello").getMethod("sayThankyou");
            m5 = Class.forName("com.iwtrtw.dp.test.proxy.IHello").getMethod("sayBye");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }

    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final void sayHello() throws  {
        try {
            super.h.invoke(this, m4, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final void sayThankyou() throws  {
        try {
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final void sayBye() throws  {
        try {
            super.h.invoke(this, m5, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

}
```

Proxy.newProxyInstance()主要做了以下三步:

> 生成代理类: Class<?> cl = getProxyClass0(loader, intfs);

> 获取构造器: final Constructor<?> cons = cl.getConstructor(constructorParams);

> 生成代理对象: cons.newInstance(new Object[]{h});

```java
    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
        //如果h为空直接抛出空指针异常，之后所有的单纯的判断null并抛异常，都是此方法
        Objects.requireNonNull(h);

        // 拷贝类实现的所有接口
        final Class<?>[] intfs = interfaces.clone();
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }

        // 1.查找或生成指定的代理类        
        Class<?> cl = getProxyClass0(loader, intfs);

        /*
         * Invoke its constructor with the designated invocation handler.
         */
        try {
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }
        /*
	    * 2.获取代理类的构造方法。
	    * constructorParams是类常量，作为代理类构造函数的参数类型，常量定义如下:
	    * private static final Class<?>[] constructorParams = { InvocationHandler.class };
	    */
            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            if (!Modifier.isPublic(cl.getModifiers())) {
                //对其访问权限进行判断，如果不是public，则将其设置可访问的
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            // 3.通过代理类的构造方法对象,传入参数InvocationHandler的实例h，创建代理类实例并返回
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
    }
```

getProxyClass0()：生成代理类的Class实例，通过Proxy类的静态变量proxyClassCache.get()可以得到缓存的代理类或创建代理类（没有缓存的情况）

```java
private static Class<?> getProxyClass0(ClassLoader loader,
                                           Class<?>... interfaces) {
        if (interfaces.length > 65535) {
            throw new IllegalArgumentException("interface limit exceeded");
        }

        /*
        If the proxy class defined by the given loader implementing the given interfaces exists, this will simply return the cached copy; otherwise, it will create the proxy class via the ProxyClassFactory
如果实现给定接口的给定加载器定义的代理类存在，这将简单地返回缓存的副本;否则，它将通过ProxyClassFactory创建代理类
        */
        return proxyClassCache.get(loader, interfaces);
    }
```

```java
    /**
    WeakCache<K, P, V>K代表key的类型，P代表参数的类型，V代表value的类型  
    WeakCache<ClassLoader, Class<?>[], Class<?>>  proxyClassCache
    说明proxyClassCache存的值是Class<?>对象，正是我们需要的代理类对象。
     */
    private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
        proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());
```

![WeakCache](/image/WeakCache.webp)

WeakCache缓存的数据结构是双重map(key,sub-key)->(value)。这个结构和Redis里面的hash结构很类似，根据一级的键（key）、二级的键（sub-key）为索引，去检索出值（value）。对应到WeakCache类代码里面，就是一个ConcurrentMap实例map，这个map的key就是一级键，map的value又是个ConcurrentMap实例，这个子map的key是二级键，子map的value就是缓存的的值。

WeakCache是一个双重的Map结构: (key, sub-key) -> value。其中key是传进来的Classloader进行包装后的对象，sub-key是由WeakCache构造函数传人的Proxy.KeyFactory()生成的。value就是产生代理类的对象，是由WeakCache构造函数传人的Proxy.ProxyClassFactory()生成的

proxyClassCache.get()

```java
// key：类加载器；parameter：接口数组
public V get(K key, P parameter) {
        Objects.requireNonNull(parameter);

        expungeStaleEntries();
    	
    	// 将ClassLoader包装成CacheKey, 作为一级缓存的key
        Object cacheKey = CacheKey.valueOf(key, refQueue);

        // 根据一级key得到 ConcurrentMap<Object, Supplier<V>>对象。如果之前不存在，则新建一个ConcurrentMap<Object, Supplier<V>>和cacheKey（一级key）一起放到map中

        ConcurrentMap<Object, Supplier<V>> valuesMap = map.get(cacheKey);
        if (valuesMap == null) {
            ConcurrentMap<Object, Supplier<V>> oldValuesMap
                = map.putIfAbsent(cacheKey,
                                  valuesMap = new ConcurrentHashMap<>());
            if (oldValuesMap != null) {
                valuesMap = oldValuesMap;
            }
        }

        // 生成sub-key：subKeyFactory.apply(key, parameter)
        Object subKey = Objects.requireNonNull(subKeyFactory.apply(key, parameter));
    
    	//通过sub-key得到supplier
        Supplier<V> supplier = valuesMap.get(subKey);
    
  		//supplier实际上就是这个factory
    //private final class Factory implements Supplier<V>
        Factory factory = null;

        while (true) {
            //如果缓存里有supplier ，那就直接通过get方法，得到代理类对象
            if (supplier != null) {
                V value = supplier.get();
                // 在这里supplier可能是一个Factory也可能会是一个CacheValue
				// 在这里不作判断, 而是在Supplier实现类的get方法里面进行验证
                if (value != null) {
                    return value;
                }
            }
            //如果缓存中没有supplier，则创建一个Factory对象，把factory对象在多线程的环境下安全的赋给supplier。因为是在while（true）中，赋值成功后又回到上面去调get方法，返回才结束。
            if (factory == null) {
                factory = new Factory(key, parameter, subKey, valuesMap);
            }

            if (supplier == null) {
                // 到这里表明subKey没有对应的值, 就将factory作为subKey的值放入
                supplier = valuesMap.putIfAbsent(subKey, factory);
                if (supplier == null) {
                    // 到这里表明成功将factory放入缓存
                    supplier = factory;
                }
                // 否则, 可能期间有其他线程修改了值, 那么就不再继续给subKey赋值, 而是取出来直接用
            } else {
                // 期间可能其他线程修改了值, 那么就将原先的值替换
                if (valuesMap.replace(subKey, supplier, factory)) {
                    // 成功将factory替换成新的值
                    supplier = factory;
                } else {
                    // 替换失败, 继续使用原先的值
                    supplier = valuesMap.get(subKey);
                }
            }
        }
    }
```

Factory.get()/supplier.get()：获取代理对象

```java
        @Override
        public synchronized V get() { // serialize access
            Supplier<V> supplier = valuesMap.get(subKey);
            //重新检查得到的supplier是不是当前对象
            if (supplier != this) {
                return null;
            }
            
            V value = null;
            try {
                //代理类就是在这个位置调用valueFactory生成的
                //valueFactory就是我们传入的 new ProxyClassFactory()
                //一会我们分析ProxyClassFactory()的apply方法
                value = Objects.requireNonNull(valueFactory.apply(key, parameter));
            } finally {
                if (value == null) {
                    valuesMap.remove(subKey, this);
                }
            }
            
            assert value != null;

            // wrap value with CacheValue (WeakReference)
            //把value包装成弱引用
            CacheValue<V> cacheValue = new CacheValue<>(value);

            // put into reverseMap
            // reverseMap是用来实现缓存的有效性
            reverseMap.put(cacheValue, Boolean.TRUE);

            if (!valuesMap.replace(subKey, this, cacheValue)) {
                throw new AssertionError("Should not reach here");
            }

            return value;
        }
    }
```

ProxyClassFactory.apply()

```java
private static final class ProxyClassFactory
        implements BiFunction<ClassLoader, Class<?>[], Class<?>>
    {
        // prefix for all proxy class names
    	//所有代理类名字的前缀$Proxy
        private static final String proxyClassNamePrefix = "$Proxy";

        // next number to use for generation of unique proxy class names
    	//用于生成代理类名字的计数器,使用唯一的编号给作为代理类名的一部分，如$Proxy0,$Proxy1等
        private static final AtomicLong nextUniqueNumber = new AtomicLong();

        @Override
        public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {

            Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
            //验证代理接口
            for (Class<?> intf : interfaces) {
                /*
                 * Verify that the class loader resolves the name of this
                 * interface to the same Class object.
                 */
                Class<?> interfaceClass = null;
                try {
                    interfaceClass = Class.forName(intf.getName(), false, loader);
                } catch (ClassNotFoundException e) {
                }
                if (interfaceClass != intf) {
                    throw new IllegalArgumentException(
                        intf + " is not visible from class loader");
                }
                /*
                 * Verify that the Class object actually represents an
                 * interface.
                 验证该Class对象是不是接口
                 */
                if (!interfaceClass.isInterface()) {
                    throw new IllegalArgumentException(
                        interfaceClass.getName() + " is not an interface");
                }
                /*
                 * Verify that this interface is not a duplicate.
                  验证该接口是否重复
                 */
                if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
                    throw new IllegalArgumentException(
                        "repeated interface: " + interfaceClass.getName());
                }
            }

            //生成的代理类的包名
            String proxyPkg = null;
            //代理类访问控制符: public ,final
            int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

            /*
             * Record the package of a non-public proxy interface so that the
             * proxy class will be defined in the same package.  Verify that
             * all non-public proxy interfaces are in the same package.
             */
            //验证所有非公共的接口在同一个包内；公共的就无需处理
            //生成包名和类名的逻辑，包名默认是com.sun.proxy，类名默认是$Proxy 加上一个自增的整数值
            //如果被代理类是 non-public proxy interface ，则用和被代理类接口一样的包名
            for (Class<?> intf : interfaces) {
                int flags = intf.getModifiers();
                if (!Modifier.isPublic(flags)) {
                    accessFlags = Modifier.FINAL;
                    String name = intf.getName();
                    int n = name.lastIndexOf('.');
                    String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                    if (proxyPkg == null) {
                        proxyPkg = pkg;
                    } else if (!pkg.equals(proxyPkg)) {
                        throw new IllegalArgumentException(
                            "non-public interfaces from different packages");
                    }
                }
            }

            if (proxyPkg == null) {
                // if no non-public proxy interfaces, use com.sun.proxy package
                proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
            }

          
            long num = nextUniqueNumber.getAndIncrement();
            //代理类的完全限定名，如com.sun.proxy.$Proxy0.calss
            String proxyName = proxyPkg + proxyClassNamePrefix + num;

            //生成代理类的字节码
            byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
                proxyName, interfaces, accessFlags);
            try {
                //把代理类加载到JVM中，至此动态代理过程基本结束了
                return defineClass0(loader, proxyName,
                                    proxyClassFile, 0, proxyClassFile.length);
            } catch (ClassFormatError e) {
                throw new IllegalArgumentException(e.toString());
            }
        }
    }
```

ProxyGenerator.generateProxyClass()：生成代理类字节码

```java
public static byte[] generateProxyClass(final String var0, Class<?>[] var1, int var2) {
        ProxyGenerator var3 = new ProxyGenerator(var0, var1, var2);
        // 真正生成字节码的方法
        final byte[] var4 = var3.generateClassFile();
        if (saveGeneratedFiles) {
            AccessController.doPrivileged(new PrivilegedAction<Void>() {
                public Void run() {
                    try {
                        int var1 = var0.lastIndexOf(46);
                        Path var2;
                        if (var1 > 0) {
                            Path var3 = Paths.get(var0.substring(0, var1).replace('.', File.separatorChar));
                            Files.createDirectories(var3);
                            var2 = var3.resolve(var0.substring(var1 + 1, var0.length()) + ".class");
                        } else {
                            var2 = Paths.get(var0 + ".class");
                        }

                        Files.write(var2, var4, new OpenOption[0]);
                        return null;
                    } catch (IOException var4x) {
                        throw new InternalError("I/O exception saving generated file: " + var4x);
                    }
                }
            });
        }

        return var4;
    }

```

**ProxyGenerator.generateClassFile()**

```java
private byte[] generateClassFile() {
    //1.为所有方法生成代理调度代码，将代理方法对象集合起来
    
    	//增加 hashcode、equals、toString方法
        this.addProxyMethod(hashCodeMethod, Object.class);
        this.addProxyMethod(equalsMethod, Object.class);
        this.addProxyMethod(toStringMethod, Object.class);
        Class[] var1 = this.interfaces;
        int var2 = var1.length;

        int var3;
        Class var4;
    
    	// 获得所有接口中的所有方法，并将方法添加到代理方法中
        for(var3 = 0; var3 < var2; ++var3) {
            var4 = var1[var3];
            Method[] var5 = var4.getMethods();
            int var6 = var5.length;
            for(int var7 = 0; var7 < var6; ++var7) {
                Method var8 = var5[var7];
                this.addProxyMethod(var8, var4);
            }
        }

        Iterator var11 = this.proxyMethods.values().iterator();

        List var12;
        while(var11.hasNext()) {
            var12 = (List)var11.next();
            checkReturnTypes(var12);
        }

    //2.为类中的方法生成字段信息和方法信息
        Iterator var15;
        try {
            this.methods.add(this.generateConstructor());
            var11 = this.proxyMethods.values().iterator();

            while(var11.hasNext()) {
                var12 = (List)var11.next();
                var15 = var12.iterator();

                while(var15.hasNext()) {
                    ProxyGenerator.ProxyMethod var16 = (ProxyGenerator.ProxyMethod)var15.next();
                    this.fields.add(new ProxyGenerator.FieldInfo(var16.methodFieldName, "Ljava/lang/reflect/Method;", 10));
                    this.methods.add(var16.generateMethod());
                }
            }
            
            // 为代理类生成静态代码块，对一些字段进行初始化
            this.methods.add(this.generateStaticInitializer());
        } catch (IOException var10) {
            throw new InternalError("unexpected I/O Exception", var10);
        }

        if (this.methods.size() > 65535) {
            throw new IllegalArgumentException("method limit exceeded");
        } else if (this.fields.size() > 65535) {
            throw new IllegalArgumentException("field limit exceeded");
        } else {
            
            //3.编写最终类文件
            this.cp.getClass(dotToSlash(this.className));
            this.cp.getClass("java/lang/reflect/Proxy");
            var1 = this.interfaces;
            var2 = var1.length;

            for(var3 = 0; var3 < var2; ++var3) {
                var4 = var1[var3];
                this.cp.getClass(dotToSlash(var4.getName()));
            }
            
            this.cp.setReadOnly();
            ByteArrayOutputStream var13 = new ByteArrayOutputStream();
            DataOutputStream var14 = new DataOutputStream(var13);

            try {
                var14.writeInt(-889275714);
                var14.writeShort(0);
                var14.writeShort(49);
                this.cp.write(var14);
                var14.writeShort(this.accessFlags);
                var14.writeShort(this.cp.getClass(dotToSlash(this.className)));
                var14.writeShort(this.cp.getClass("java/lang/reflect/Proxy"));
                var14.writeShort(this.interfaces.length);
                Class[] var17 = this.interfaces;
                int var18 = var17.length;

                for(int var19 = 0; var19 < var18; ++var19) {
                    Class var22 = var17[var19];
                    var14.writeShort(this.cp.getClass(dotToSlash(var22.getName())));
                }

                var14.writeShort(this.fields.size());
                var15 = this.fields.iterator();

                while(var15.hasNext()) {
                    ProxyGenerator.FieldInfo var20 = (ProxyGenerator.FieldInfo)var15.next();
                    var20.write(var14);
                }

                var14.writeShort(this.methods.size());
                var15 = this.methods.iterator();

                while(var15.hasNext()) {
                    ProxyGenerator.MethodInfo var21 = (ProxyGenerator.MethodInfo)var15.next();
                    var21.write(var14);
                }

                var14.writeShort(0);
                return var13.toByteArray();
            } catch (IOException var9) {
                throw new InternalError("unexpected I/O Exception", var9);
            }
        }
    }
```



![getProxyClass0](/image/getProxyClass0.webp)

当代理对象生成后，最后由InvocationHandler的invoke()方法调用目标方法，在动态代理中InvocationHandler是核心，每个代理实例都具有一个关联的InvocationHandler;对代理实例调用方法时，将对方法调用进行编码并将其指派到它的InvocationHandler的invoke()方法。所以对代理方法的调用都是通InvocationHadler的invoke来实现中，而invoke方法根据传入的代理对象，方法和参数来决定调用代理的哪个方法。

> invoke(Object Proxy，Method method，Object[] args)

第一个参数为$Proxy0(代理对象)，第二个参数为目标类的真实方法，第三个参数为目标方法参数，如果没有参数，则设置为null。

###### CGLib动态代理

+ CGLib(Code Generation Library)动态代理：Cglib并不依赖接口，可以直接生成委托类的代理对象，而且可以代理委托类的任意非final修饰的public和protected方法。其原理是通过字节码技术为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。

HelloServiceImpl

```java
public class HelloServiceImpl {
    public void sayHello(){
        System.out.println("Hello World");
    }

    public void sayBey(){
        System.out.println("Bye World");
    }
}
```

HelloMethodInterceptor  

```java
public class HelloMethodInterceptor  implements MethodInterceptor{
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("Before: "  + method.getName());
        Object object = methodProxy.invokeSuper(o, objects);
        System.out.println("After: " + method.getName());
        return object;
    }
}
```

Test

```java
public class Client {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        //继承被代理类
        enhancer.setSuperclass(HelloServiceImpl.class);
        //设置回调
        enhancer.setCallback(new HelloMethodInterceptor());
        //设置代理类对象
        HelloServiceImpl helloService = (HelloServiceImpl) enhancer.create();
        //在调用代理类中方法时会被我们实现的方法拦截器进行拦截
        helloService.sayBey();
    }
}
```

代理类是由enhancer.create()创建的。Enhancer是CGLIB的字节码增强器，可以很方便的对类进行拓展,创建代理类的过程：

- 生成代理类的二进制字节码文件；
- 加载二进制字节码，生成Class对象；
- 通过反射机制获得实例构造，并创建代理类对象。

代理类将委托类HelloServiceImpl作为自己的父类并为其中的非final方法创建两个方法，一个是重写父类方法，它会判断是否存在实现了MethodInterceptor接口的对象，若存在则将调用intercept方法对委托方法进行代理。另一个是代理类独有的方法，会直接通过super调用父类方法。当我们去调用方法一的时候，在代理类中会先判断是否实现了方法拦截的接口，没实现的话直接调用目标类的方法一；如果实现了那就会被方法拦截器拦截，在方法拦截器中会对目标类中所有的方法建立索引，其实大概就是将每个方法的引用保存在数组中，我们就可以根据数组的下标直接调用方法，而不是用反射；索引建立完成之后，方法拦截器内部就会调用invoke方法（这个方法在生成的FastClass中实现），在invoke方法内就是调用CGLIB$方法一$这种方法，也就是调用对应的目标类的方法一。

CGLIB在类生成期间的操作会相对耗时，而且生成的类数目比较多，会占据大量永久代或者元空间的内存。子类一旦生成，后面的方法调用就会变成搜索方法索引和直接调用，这样的操作在特定的条件下效率会比JDK的反射高，CGLib所创建的动态代理对象在实际运行时候的性能要比JDK动态代理高不少，但是CGLib在创建对象的时候所花费的时间却比JDK动态代理要多很多，对于singleton的代理对象或者具有实例池的代理，因为无需频繁的创建代理对象，所以比较适合采用CGLib动态代理，反正，则比较适用JDK动态代理

#### I/O

##### BIO

##### NIO

#### 集合

​		Java 集合框架主要包括两种类型的容器，一种是集合（Collection），存储一个元素集合，另一种是图（Map），存储键/值对映射。Collection 接口中常见的子接口有List、Set 和 Queue；Map接口中的常用子接口有AbstractMap、SortedMap、ConcurrentMap

```java
public interface Collection<E> extends Iterable<E> 
    public interface Set<E> extends Collection<E>
    public abstract class AbstractCollection<E> implements Collection<E>
    public interface List<E> extends Collection<E>
    public interface Queue<E> extends Collection<E>
```

##### Set

```java
public abstract class AbstractSet<E> extends AbstractCollection<E> implements Set<E>
```

+ HashSet

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
```

+ LinkedHashSet

```java
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
```

+ TreeSet

```java
public interface SortedSet<E> extends Set<E>
    public interface NavigableSet<E> extends SortedSet<E>
    	public class TreeSet<E> extends AbstractSet<E> implements NavigableSet<E>, Cloneable, java.io.Serializable
```

##### List

+ ArrayList：ArrayList底层采用数组实现，拥有快速随机访问能力，是非线程安全的集合。ArrayList默认容量为10，扩容规则为当要保存的新元素所需的容量不足时触发，基本规则为扩容1.5倍。如果在遍历的时候发生结构性变化，会触发ConcurrentModificationException异常。结构性变化包括：添加新元素，删除元素。ArrayList支持序列化功能，支持克隆（浅拷贝）功能，排序功能等

```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E>
```

```java
public class ArrayList<E> 
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

+ LinkedList：LinkedList的本质是双向链表；顺序访问会非常高效，而随机访问效率比较低。

```java
public abstract class AbstractSequentialList<E> extends AbstractList<E>
```

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

由于LinkedList实现了Deque，而Deque接口定义了在双端队列两端访问元素的方法。提供插入、移除和检查元素的方法。每种方法都存在两种形式：一种形式在操作失败时抛出异常，另一种形式返回一个特殊值（null 或 false，具体取决于操作）

|      | 抛出异常      | 特殊值        | 抛出异常     | 特殊值       |
| ---- | ------------- | ------------- | ------------ | ------------ |
| 插入 | addFirst(e)   | offerFirst(e) | addLast(e)   | offerLast(e) |
| 移除 | removeFirst() | pollFirst()   | removeLast() | pollLast()   |
| 检查 | getFirst()    | peekFirst()   | getLast()    | peekLast()   |

LinkedList可以作为FIFO(先进先出)的队列，作为FIFO的队列时

| 队列方法  | linkedlist方法 |
| --------- | -------------- |
| add(e)    | addLast(e)     |
| offer(e)  | offerLast(e)   |
| remove()  | removeFirst()  |
| poll()    | pollFirst()    |
| element() | getFirst()     |
| peek()    | peekFirst()    |

LinkedList可以作为LIFO(后进先出)的栈，作为LIFO的栈时

| 栈方法  | linkedlist    |
| ------- | ------------- |
| push(e) | addFirst(e)   |
| pop()   | removeFirst() |
| peek()  | peekFirst()   |

LinkedList通过add(int index, E element)向LinkedList插入元素时。先是在双向链表中找到要插入节点的位置index；找到之后，再插入一个新节点。双向链表查找index位置的节点时，有一个加速动作：若index < 双向链表长度的1/2，则从前向后查找; 否则，从后向前查找。ArrayList向指定位置插入元素时，真正耗时的操作是 System.arraycopy(elementData, index, elementData, index + 1, size - index)，会移动index之后所有元素

##### fail-fast机制

fail-fast 机制是java集合(Collection)中的一种错误机制。当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。例如：当某一个线程A通过iterator去遍历某集合的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。fail-fast机制，是一种错误检测机制。它只能被用来检测错误，因为JDK并不保证fail-fast机制一定会发生。若在多线程环境下使用fail-fast机制的集合，建议使用“java.util.concurrent包下的类”去取代“java.util包下的类”。

ConcurrentModificationException是在操作Iterator时抛出的异常。我们先看看Iterator的源码。ArrayList的Iterator是在父类AbstractList.java中实现的，在调用 next() 和 remove()时，都会执行 checkForComodification()。若 “modCount 不等于 expectedModCount”，则抛出ConcurrentModificationException异常，产生fail-fast事件。在ArrayList中，无论是add()、remove()，还是clear()，只要涉及到修改集合中的元素个数时，都会改变modCount的值。

```java
// Itr是Iterator(迭代器)的实现类
private class Itr implements Iterator<E> {
        int cursor = 0;

        int lastRet = -1;

        // 修改数的记录值。
        // 每次新建Itr()对象时，都会保存新建该对象时对应的modCount；
        // 以后每次遍历List中的元素的时候，都会比较expectedModCount和modCount是否相等；
        // 若不相等，则抛出ConcurrentModificationException异常，产生fail-fast事件。
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size();
        }

        public E next() {
            // 获取下一个元素之前，都会判断“新建Itr对象时保存的modCount”和“当前的modCount”是否相等；
            // 若不相等，则抛出ConcurrentModificationException异常，产生fail-fast事件。
            checkForComodification();
            try {
                int i = cursor;
                E next = get(i);
                lastRet = i;
                cursor = i + 1;
                return next;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                AbstractList.this.remove(lastRet);
                if (lastRet < cursor)
                    cursor--;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException e) {
                throw new ConcurrentModificationException();
            }
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```

##### Map

```java
public interface Map<K,V>
    public abstract class AbstractMap<K,V> implements Map<K,V>
    public interface SortedMap<K,V> extends Map<K,V>
    public interface ConcurrentMap<K, V> extends Map<K, V>
```

+ HashMap：HashMap 是一个散列表，它存储的内容是键值对(key-value)映射；它是非线程安全，key可以为null（但只能有一个）；它是通过“拉链法”实现的，它包括几个重要的成员变量：table, size, threshold, loadFactor, modCount。
  + table：一个Entry[]数组类型，而Entry实际上就是一个单向链表。哈希表的"key-value键值对"都是存储在Entry数组中的。
  + size是HashMap的大小，它是HashMap保存的键值对的数量，初始容量默认为16
  + threshold是HashMap的阈值，用于判断是否需要调整HashMap的容量。threshold的值=容量*加载因子，当HashMap中存储数据的数量达到threshold时，就需要扩容（2倍）
  + loadFactor就是加载因子，默认为0.75
  + modCount是用来实现fail-fast机制的。

```java
public class HashMap<K,V> 
    extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
```

+ Hashtable

```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable
```

+ ConcurrentHashMap

```java
public class ConcurrentHashMap<K,V> 
    extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable
```

+ TreeMap

```java
public interface NavigableMap<K,V> extends SortedMap<K,V>
```

```java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable
```

#### 并发

##### synchronized

##### Lock

#### 其他

##### 排序