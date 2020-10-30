#### volatile保证可见性

​		volatile的可以保证变量内存的可见性且并且可以提供了一定的有序性，禁止指令重排，但是无法保证原子性。 被volatile修饰的变量能够保证每个线程能够获取该变量的最新值，从而避免出现数据脏读的现象。volatile可以用来修饰成员变量和静态成员变量，可以避免线程从自己的工作缓存中查找变量值，必须到主存中获取它的值，线程操作volatile变量都是直接操作主存。

​		Java内存模型(Java Memory Mode)规定所有的共享变量都是存在主存中，每个线程都有自己的工作内存。线程对变量的所有操作都是在自己的工作内存中进行，不能直接读写主内存中的变量，并且每个线程不能直接访问其他线程的工作内存，线程间变量的值的传递需要通过主内存中转来完成。这样就会造成可见性问题，可能会导致线程无法将工作内存中的修改值更新到主存中，也可能导致线程没有及时将主存中的最新值同步到工作内存中，总之，线程在工作内存中的共享变量值可能并不是最新的。

​		在生成汇编代码时会在volatile修饰的共享变量进行写操作的时候会多出**Lock前缀的指令**，Lock前缀指令的影响主要有两方面：

**I.修改缓存行时会强制将数据写会主存**

**II.这个写回内存的操作会使得其他CPU里缓存了该内存地址的数据无效**

​		为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存（L1，L2或其他）后再进行操作，但操作完不知道何时会写到内存。如果对声明了volatile的变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存。但是，就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题。所以，在多处理器下，为了保证各个处理器的缓存是一致的，就会实现**缓存一致性**协议，**每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期**了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。	

+ Lock前缀的指令会引起处理器缓存写回内存
+ 一个处理器的缓存回写到内存会导致其他处理器的缓存失效
+ 当处理器发现本地缓存失效后，就会从内存中重读该变量数据，即可以获取当前最新值

针对volatile变量通过这样的机制就使得每个线程都能获得该变量的最新值

#### volatile禁止指令重排序

JVM的实现会在volatile读写前后均加上内存屏障，在一定程度上保证有序性。volatile写是在前面和后面**分别插入内存屏障**，而volatile读操作是在**后面插入两个内存屏障**

> StoreStoreBarrier
> volatile 写操作
> StoreLoadBarrier
>
> 
>
> volatile 读操作
> LoadLoadBarrier
> LoadStoreBarrier

+ **StoreStore屏障**：禁止上面的普通写和下面的volatile写重排序
+ **StoreLoad屏障**：防止上面的volatile写与下面可能有的volatile读/写重排序
+ **LoadLoad屏障**：禁止下面所有的普通读操作和上面的volatile读重排序
+ **LoadStore屏障**：禁止下面所有的普通写操作和上面的volatile读重排序

volatile禁止指令重排序也有一些规则：

> a.当程序执行到volatile变量的读操作或者写操作时，在其前面的操作的更改肯定全部已经进行，且结果已经对后面的操作可见；在其后面的操作肯定还没有进行；

> b.在进行指令优化时，不能将在对volatile变量访问的语句放在其后面执行，也不能把volatile变量后面的语句放到其前面执行。

**即执行到volatile变量时，其前面的所有语句都执行完，后面所有语句都未执行。且前面语句的结果对volatile变量及其后面语句可见。**

+ 在每个volatile写操作前插入StoreStore屏障，在写操作后插入StoreLoad屏障

![volatile写屏障](image\volatile写屏障.png)

+ 在每个volatile读操作的后面插入一个LoadLoad屏障,在每个volatile读操作的后面插入一个LoadStore屏障

![volatile读屏障](image\volatile读屏障.png)

#### volatile的的底层实现

+ Java代码层面

```java
public class TestVolatile {
    
    public static volatile int counter = 1;

    public static void main(String[] args){
        counter = 2;
        System.out.println(counter);
    }
}
```

+ 字节码层面：使用访问标志**ACC_VOLATILE**来表示，供后续操作此变量时判断访问标志是否为ACC_VOLATILE，来决定是否遵循volatile的语义处理

```java
public static volatile int counter;
    descriptor: I
    flags: ACC_PUBLIC, ACC_STATIC, ACC_VOLATILE
```

+ 汇编层面

```java
lock;addl $0,0(%%rsp) //addl $0,0(%%rsp) 是把寄存器的值加0，相当于一个空操作
```