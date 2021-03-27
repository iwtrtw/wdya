### 如何查看SQL执行计划？

CREATE INDEX index_name ON table_name (column_list)

CREATE UNIQUE INDEX index_name ON table_name (column_list)

使用explain + SQL;通过执行计划可以大概估计查询要扫描多少行、使用哪些索引，了解SQL在数据库中执行时的表现情况

+ Id：表示执行顺序，id的数字越大越先执行，如果数字一样，那么从上往下依次执行，如果为null表示这是一个结果集，不需要用它来进行查询，最后执行

+ select_type：表示查询的类型，主要是用于区分普通查询、联合查询、子查询等复杂的查询。SIMPLE值表明没有子查询或Union；如果有子查询，那么最外层标记为Primary

  + SUBQUERY：在select 或 where列表中包含了子查询

  > id，age建立了索引
  >
  > explain select age,(select count(*) from user) from user where id>(select id from user where id=4800);

  ![SQL-subquery](..\image\SQL-subquery.png)

  + DERIVED：包含在from中的子查询，MySQL会递归执行并将结果放在一个临时表中，成为派生表，从子查询中派生出来的。
  + union：若第二个select出现在union之后，则被标记为union；若union包含在from子句的子查询中，外层select将被标记为derived
  + union result：从union表获取结果的select，在union和union all语句中,因为它不需要参与查询，所以id字段为null

  > id>5800:10条数据   id<10:4条数据
  >
  > explain select id,name from user where id>5800 union  select id,name from user where id<10;

  ![SQL-UNION](..\image\SQL-UNION.png)

  > explain select * from (select id,name from user where id>5800 union  select id,name from user where id<10) a;

  ![SQL-UNION-DERIVED](..\image\SQL-UNION-DERIVED.png)

+ table：显示当前行的数据来自于哪一张表，如果查询使用了别名，那么这里显示的是别名，如果不涉及对数据表的操作，那么这显示为null，如果显示为尖括号括起来的就表示这个是临时表，后边的N就是执行计划中的id，表示结果来自于这个查询产生。如果是尖括号括起来的<union M,N>，与类似，也是一个临时表，表示这个结果来自于union查询的id为M,N的结果集。

+ type：访问类型，结果值从好到坏依次是NULL>system>const >eq_ref > ref > range>index>ALL，一般来说，保证查询至少能到达range级别，最好能达到ref

  + system：system 表中只有一行数据（系统表）

  + const 通过索引一次就能找到的数据，比如primary key 和union key，主键在where条件中，就能将查询转换成一个常量。比如user表中id是主键

    > explain select * from user where id=1;

    ![SQL-TYPE-CONST](..\image\SQL-TYPE-CONST.png)

  + eq_ref 唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描。(对于前表的每一行(row)，后表只有一行被扫描)

    > explain select it.title,it.price,st.stock from item it left join item_stock st on it.id=st.item_id;    //st.item_id建了唯一索引

    ![SQL-TYPE-EQ_REF](..\image\SQL-TYPE-EQ_REF.png)

  + ref 非唯一性索引扫描，返回匹配某个单独值得所有行 比如：student表中的age是一个普通索引 （对于前表的每一行(row)，后表可能有多于一行的数据被扫描）

    > explain select age from user where age=21; //age非唯一索引
    >
    > explain select name from user where age=21;  //name没有建索引

    ![SQL-TYPE-REF](..\image\SQL-TYPE-REF.png)

  + range：只检索给定范围的行，使用一个索引来选择行，key列显示使用了哪个索引，一般就是where条件中出现“>”、“<”、“between”、“in”等条件。这种范围扫描索引扫描比全表扫描要好，因为它只需要开始于索引的某一点，而结束语另一点，不用扫描全部索引

    > explain select * from user where id<10;
    >
    > explain select * from user where id in (1,3);
    >
    > explain select * from user where id between 1 and 10;

  + index：索引树扫描(（Index与ALL虽然都是读全表，但index是从索引中读取，而ALL是从硬盘读取） )

    > explain select id from user;   //使用辅助索引age，而不是主键索引id
    >
    > explain select age from user;  
    >
    > explain select age,id,name from user; //覆盖索引，使用辅助索引

    ![SQL-TYPE-INDEX](..\image\SQL-TYPE-INDEX.png)

  + all：全表扫描

+ possible_keys：查询涉及到的字段上存在索引，则该索引将被列出，但不一定被查询实际使用

+ key：实际使用的索引，如果为NULL，则没有使用索引。查询中如果使用了覆盖索引，则该索引名字仅出现在key列表中

+ key_len：用于处理查询的索引长度，如果是单列索引，那就整个索引长度算进去，如果是多列索引，那么查询不一定都能使用到所有的列，具体使用到了多少个列的索引，这里就会计算进去，没有使用到的列，这里不会计算进去

  ```
  索引中使用的字节数，可通过该列计算出使用的是哪些列，长度越短越好。显示的是索引字段的最大可能长度，并非实际使用长度，根据表定义计算而来，不是通过表内检索而来。需要注意的是：
  1、char字段一个字符在utf8编码下最多占3个字节，可变长字段需要额外的两个字节记录长度，外加需要存入一个null值，一个null是一个字节 
  2、复合索引有最左前缀的特性，如果复合索引能全部使用上，则是复合索引字段的索引长度之和，这也可以用来判定复合索引是否部分使用，还是全部使用。
  比如：student表的id是int类型，四个字节，所以key_len是4
  name字段类型是varchar(20)，所以ken_len=20*3+2+1=63
  ```

+ ref：显示索引的那一列被使用了，如果可能，是一个常量const。如果是使用的常数等值查询，这里会显示const，如果是连接查询，被驱动表的执行计划这里会显示驱动表的关联字段，如果是条件使用了表达式或者函数，或者条件列发生了内部隐式转换，这里可能显示为func

+ rows：根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数

+ extra：额外信息

  + Using where：使用where进行范围查询（在查找使用索引的情况下，需要回表去查询所需的数据）

    > explain select * from user where id>10
    >
    > explain select name from user where age>10;

  + Using index：SQL所需要返回的所有列数据均在一棵索引树上，即使用了**覆盖索引**（Covering Index），而无需访问实际的行记录（覆盖索引、通过辅助索引查id）

    > 如果同时出现Using where，查找使用了索引，但是需要的数据都在索引列中能找到，所以不需要回表查询数据

  + Using index condition：确实命中了索引，但不是所有的列数据都在索引树上，还需要访问实际的行记录(查找使用了索引，但是需要回表查询数据)

    > explain select name from user where age<30;   //age小于30记录较少

    ![SQL-EXTRA-UIC](..\image\SQL-EXTRA-UIC.png)

  + Using filesort：得到所需结果集，需要对所有记录进行文件排序。典型的，在一个没有建立索引的列上进行了order by，就会触发，常见的优化方案是，在order by的列上添加索引，避免每次查询都全量排序(**需要优化**)

    > explain select name from user where age<30 order by name;

    ![SQL-EXTRA-UF](..\image\SQL-EXTRA-UF.png)

  + Using temporary：使用临时表保存中间结果，也就是说mysql在对查询结果排序时使用了临时表，常见于order by 和 group by （**需要优化**）

### 线程间的同步怎么实现？

### 怎么创建线程池？ ThreadPoolExecutor的构造函数参数有哪些？ 拒绝策略介绍下？ 默认的拒绝策略是哪一个？ 

#### 创建线程的方式

1. 继承Thread类并重写run方法

   ```java
   //线程创建
   Thread t = new Thread("mythread"){
       @Override
       public void run(){
           //要执行的任务
       }
   }
   //线程启动
   t.start()
       
   /* 匿名线程 */
   new Thread(() -> {
       System.out.println("anonymity");
   },"athred").start();
   ```

2. 实现Runnable接口

   ```java
   Runnable runnable = new Runnable(){
       @Override
       public void run(){
           //要执行的任务
       }
   }
   //创建线程对象
   Thread t = new Thread(runnable)
   //启动线程
   t.start()
   ```

   在创建Thread对象时若传入Runnable对象，如果有则执行Runnable的run方法；否则会执行Thread对象自己的run方法

   ```java
   /*
   java.lang.Thread
   target:Runnable对象
   */
   @Override
       public void run() {
           if (target != null) {
               target.run();
           }
       }
   ```

   使用Runnable更容易与线程池等高级API配合，而且可以不用继承Thread

3. 通过Callable和Future创建线程

   + 创建Callable接口的实现类，并实现call()方法，该***call()方法将作为线程执行体，并且有返回值***
   + 创建Callable实现类的实例，**使用FutureTask类来包装Callable对象**，该FutureTask对象封装了该Callable对象的call()方法的返回值。（**FutureTask是一个包装器**，它通过接受Callable来创建，它***同时实现了Future和Runnable接口***。）
   + 使用FutureTask对象作为Thread对象的target创建并启动新线程
   + 调用FutureTask对象的get()方法来获得子线程执行结束后的返回值

   ```java
   //FutureTask间接继承Runnable接口
   FutureTask<Integer> task = new FutureTask<Integer>(new Callable<Integer>() {
       public Integer call() throws Exception {
           return 100;
       }
   });
   Thread t = new Thread(task,"mythrad");
   t.start();
   //获取返回结果
   System.out.println(task.get());  //100
   ```

池化技术的思想主要是为了减少每次获取资源的消耗，提⾼对资源的利⽤率。线程池提供了⼀种限制和管理资源（包括执⾏⼀个任务）。 每个线程池还维护⼀些基本统计信息，例如已完成任务的数量。在实际使用中，线程是很占用系统资源的，如果对线程管理不善很容易导致系统问题。因此，在大多数并发框架中都会使用**线程池**来管理线程，使用线程池管理线程主要有如下好处

+ 降低资源消耗：通过重复利⽤已创建的线程降低线程创建和销毁造成的消耗
+ 提升系统响应速度：当任务到达时，任务可以不需要的等到线程创建就能⽴即执⾏。
+ 提高线程的可管理性：线程是稀缺资源，如果⽆限制的创建，不仅会消耗系统资源，还会降低系 统的稳定性，使⽤线程池可以进⾏统⼀的分配，调优和监控。

#### 创建线程池的方法

1. 通过new ThreadPoolExceutor()自定义创建

   ```java
   public ThreadPoolExecutor(int corePoolSize,
                                 int maximumPoolSize,
                                 long keepAliveTime,
                                 TimeUnit unit,
                                 BlockingQueue<Runnable> workQueue,
                                 ThreadFactory threadFactory,
                                 RejectedExecutionHandler handler) {}
   ```

   + corePoolSize：表示核心线程池的大小。当提交一个任务时，如果当前核心线程池的线程个数没有达到corePoolSize，则会创建新的线程来执行所提交的任务，**即使当前核心线程池有空闲的线程**。如果当前核心线程池的线程个数已经达到了corePoolSize，则不再重新创建线程。（最多保留的线程数 ）
   + maximumPoolSize：表示线程池能创建线程的最大个数。如果当阻塞队列已满时，并且当前线程池线程个数没有超过maximumPoolSize的话，就会创建新的线程来执行任务。(核心线程数目 + 救急线程数)
   + keepAliveTime：空闲线程存活时间。如果当前线程池的线程个数已经超过了corePoolSize，核心线程之外的线程空闲时间超过了keepAliveTime的话，就会将这些空闲线程销毁，这样可以尽可能降低系统资源消耗（针对救急线程）
   + unit：时间单位。为keepAliveTime指定时间单位。（针对救急线程）
   + workQueue：阻塞队列。用于保存任务的阻塞队列，当新任务来的时候会先判断当前运⾏的线程数量是否达到核⼼线程数，如果达到的话，新任务就会被存放在队列中。可以使用**ArrayBlockingQueue, LinkedBlockingQueue, SynchronousQueue, PriorityBlockingQueue**。一般使用LinkedBlockingQueue和SynchronousQueue。线程池的排队策略与BlockingQueue有关。
   + threadFactory：创建线程的工程类。可以通过指定线程工厂为每个创建出来的线程设置更有意义的名字，如果出现并发问题，也方便查找问题原因。
   + handler：饱和策略。当线程池的阻塞队列已满和指定的线程都已经开启，说明当前线程池已经处于饱和状态了，那么就需要采用一种策略来处理这种情况。
     + AbortPolicy：直接拒绝所提交的任务，并抛出**RejectedExecutionException**异常
     
       ```java
       public static class AbortPolicy implements RejectedExecutionHandler {
           public AbortPolicy() { }
           public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
               throw new RejectedExecutionException("Task " + r.toString() +
                                                    " rejected from " +
                                                    e.toString());
           }
       }
       ```
     
     + CallerRunsPolicy：让调用者所在的线程来执行任务，会调用当前线程池的所在的线程去执行被拒绝的任务(及run方法是在当前进程执行的，并不会新建一个线程)（这个策略的缺点就是可能会阻塞主线程）
     
       ```java
       public static class CallerRunsPolicy implements RejectedExecutionHandler {
           public CallerRunsPolicy() { }
           public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
               if (!e.isShutdown()) {
                   r.run();
               }
           }
       }
       ```
     
     + DiscardPolicy：不处理直接丢弃掉任务，什么没干，会让被线程池拒绝的任务直接抛弃，不会抛异常也不会执行（即后面的线程的run方法不会被执行）
     
       ```java
       public static class DiscardPolicy implements RejectedExecutionHandler {
           public DiscardPolicy() { }
           public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
           }
       }
       ```
     
     + DiscardOldestPolicy：丢弃掉阻塞队列中存放时间最久的任务，会抛弃任务队列中最旧的任务也就是最先加入队列的，空出一个位置，然后再次执行execute方法把任务加入队列。
     
       ```java
       public static class DiscardOldestPolicy implements RejectedExecutionHandler {
           public DiscardOldestPolicy() { }
           public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
                   if (!e.isShutdown()) {
                       e.getQueue().poll();
                       e.execute(r);
                   }
               }
       }
       ```
     
     + 也可以根据应用场景需要来实现RejectedExecutionHandler接口自定义策略。如记录日志或持久化不能处理的任务。例如新建一个线程去执行
     
       ```java
       static class MyRejectedExecutionHandler implements RejectedExecutionHandler {
           @Override
           public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
               new Thread(r,"新线程"+new Random().nextInt(10)).start();
           }
       }
       ```

    线程池中一开始没有线程，当一个任务提交给线程池后，线程池会创建一个新线程来执行任务；当线程数达到corePoolSize并没有线程空闲，这时再加入任务，新加的任务会被加入workQueue队列排队，直到有空闲的线程。如果队列选择的是**有界队列**，当有界队列超过队列大小时，会创建maximumPoolSize - corePoolSize数目的线程来救急。如果线程到达maximumPoolSize仍然有新任务这时会执行拒绝策略，拒绝策略jdk提供了4种实现（AbortPolicy、CallerRunsPolicy、DiscardPolicy、DiscardOldesPolicy）。当高峰过去后，超过corePoolSize的救急线程如果一段时间没有任务做，需要结束节省资源，这个时间由keepAliveTime和unit来控制

2. 通过Executor的工厂方法创建

   > FixedThreadPool 和 SingleThreadExecutor ： 允许请求的队列⻓度为 Integer.MAX_VALUE ，可能堆积⼤量的请求，从⽽导致OOM。 CachedThreadPool 和 ScheduledThreadPool ： 允许创建的线程数量为 Integer.MAX_VALUE ，可能会创建⼤量线程，从⽽导致OOM。

   + FixedThreadPool ： 该方法返回⼀个固定线程数量的线程池。该线程池中的线程数量始终不 变。当有⼀个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂存在⼀个任务队列中，待有线程空闲时，便处理在任务队列中的任务（定长线程池，可控制线程最大并发数，超出的线程会在队列中等待）
   
      ```java
     ExecutorService es1 =  Executors.newFixedThreadPool(3);
     for (int i = 0; i < 10; i++) {
         es1.submit(new Runnable() {
             @Override
             public void run() {
               System.out.println(Thread.currentThread().getName()+"RUN");
             }
          });
    }
     ```

   + SingleThreadExecutor： ⽅法返回⼀个只有⼀个线程的线程池。若多余⼀个任务被提交到该线 程池，任务会被保存在⼀个任务队列中，待线程空闲，按先⼊先出的顺序执⾏队列中的任务（单线程化的线程池，它只会唯一的工作线程来执行任务，保证所有任务按照指定顺序（FIFO，LIFO，优先级）执行）
   
     ```
     ExecutorService es2 = Executors.newSingleThreadExecutor();
     for (int i = 0; i < 10; i++) {
         es2.submit(new Runnable() {
             @Override
             public void run() {
               System.out.println(Thread.currentThread().getName()+"RUN");
             }
          });
    }
     ```

   + CachedThreadPool： 该⽅法返回⼀个可根据实际情况调整线程数量的线程池。线程池的线程数 量不确定，但若有空闲线程可以复⽤，则会优先使⽤可复⽤的线程。若所有线程均在⼯作，⼜有 新的任务提交，则会创建新的线程处理任务。所有线程在当前任务执⾏完毕后，将返回线程池进行复⽤(可缓存的线程池，如果线程池长度超过处理需求，可灵活回收空闲线程，若无可回收，则新建线程)
   
     ```java
     ExecutorService es3 = Executors.newCachedThreadPool();
     for (int i = 0; i < 20; i++) {
         es3.submit(new Runnable() {
             @Override
             public void run() {
               System.out.println(Thread.currentThread().getName()+"RUN");
             }
          });
    }
     ```

   + ScheduledThreadPool：拥有固定线程数量的定时线程任务的线程池
   
     ```java
     ScheduledExecutorService es4 = Executors.newScheduledThreadPool(2);
     System.out.println("时间：" + System.currentTimeMillis());
     for (int i = 0; i < 5; i++) {
         es4.schedule(new Runnable() {
             @Override
             public void run() {
                 System.out.println("时间："+System.currentTimeMillis()+"--"+Thread.currentThread().getName() + "正在执行任务");
             }
         },3, TimeUnit.SECONDS);
    }
     ```

   + SingleThreadScheduledExecutor：只有一个线程的定时线程任务的线程池
   
     ```java
     ScheduledExecutorService es5=Executors.newSingleThreadScheduledExecutor();
     System.out.println("时间：" + System.currentTimeMillis());
     for (int i = 0; i < 5; i++) {
         es5.schedule(new Runnable() {
             @Override
             public void run() {
                 System.out.println("时间："+System.currentTimeMillis()+"--"+Thread.currentThread().getName() + "正在执行任务");
             }
         },3, TimeUnit.SECONDS);
     }
     ```

#### 自定义线程池

+ Thread Pool

```java
public class CustomThreadPool {

    //任务队列
    private BlockingQueue<Runnable> taskQueue;

    //线程集合
    private HashSet<Worker> workers = new HashSet<>();

    //核心线程数
    private int coreSize;

    //获取任务的超时时间
    private long timeout;
    
    //时间单位
    private TimeUnit timeUnit;

    private RejectPolicy<Runnable> rejectPolicy;

    public CustomThreadPool(int coreSize, long timeout, TimeUnit timeUnit, int queueSize,RejectPolicy<Runnable> rejectPolicy) {
        this.coreSize = coreSize;
        this.timeout = timeout;
        this.timeUnit = timeUnit;
        this.taskQueue = new BlockingQueue<>(queueSize);
        this.rejectPolicy =rejectPolicy;
    }

    //执行任务
    public void execute(Runnable task) {
        synchronized (workers) {
            if (workers.size() < coreSize) {
                //新增worker
                Worker worker = new Worker(task);
                workers.add(worker);
                worker.start();

            } else {
                //worker已到达上限，任务进入阻塞队列
                taskQueue.add(task);
                /*
                如果出现待执行任务数量大于任务队列容量大小，可以使用如下方法解决
                1.死等
                2.待超时等待
                3.放弃任务执行
                4.抛出异常
                5.自己创建新线程执行任务
                * */
                taskQueue.tryPut(rejectPolicy,task);
            }
        }

    }

    class Worker extends Thread {
        private Runnable task;

        public Worker(Runnable task) {
            this.task = task;
        }

        @Override
        public void run() {
            /*当task任务不为空，执行任务；当task执行完毕，接着从任务队列中获取任务并执行,当任务队列中的任务执行完毕后，taskQueue.get()会一直等待新任务的加入
            while (task != null || (task = taskQueue.get()) != null) {*/
            System.out.println("==PULL==");
            /*当任务队列中的任务执行完毕后，taskQueue.get()会限时等待新任务的加入，超时后该worker将从线程集合中移除*/
            while (task != null || (task = taskQueue.pull(timeout,timeUnit)) != null) {
                try {
                    task.run();
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    task = null;
                }
            }
            synchronized (workers) {
                //worker执行任务指导任务队列中没有任务之后被移除
                System.out.println("---------");
                workers.remove(this);
            }
        }
    }
}
```

+ BlockingQueue

```java
public class BlockingQueue<T> {
    //任务队列
    private Deque<T> deque = new ArrayDeque<>();

    //锁
    private ReentrantLock lock = new ReentrantLock();


    //生产者条件变量
    private Condition fullWait = lock.newCondition();

    //消费者条件变量
    private Condition emptyWait = lock.newCondition();

    //容量
    private int capcity;

    public BlockingQueue(int capcity) {
        this.capcity = capcity;
    }

    //阻塞获取队列中任务
    public T get(){
        lock.lock();
        try{
            while (deque.isEmpty()){
                try{
                    emptyWait.await();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
            T t= deque.removeFirst();
            fullWait.signal();
            return t;

        }finally {
            lock.unlock();
        }
    }


    //超时阻塞获取队列中任务
    public T pull(long timeout, TimeUnit unit){
        lock.lock();
        try{
            //将timeout转换为纳秒
            long nanos =unit.toNanos(timeout);
            while (deque.isEmpty()){
                try{
                    //awaitNanos返回的是剩余时间
                    System.out.println("----------"+nanos);
                    if(nanos<=0){
                        //超时返回null
                        return null;
                    }
                    nanos=emptyWait.awaitNanos(nanos);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
            T t= deque.removeFirst();
            fullWait.signal();
            return t;

        }finally {
            lock.unlock();
        }
    }


    //阻塞添加任务
    public void add(T element){
        lock.lock();
        try{
            while (deque.size()==capcity){
                try{
                    System.out.println("任务队列已满");
                    //任务队列已满，等待加入任务队列
                    fullWait.await();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
            deque.addLast(element);
            emptyWait.signal();

        }finally {
            lock.unlock();
        }

    }


    //超时阻塞添加任务
    public boolean put(T element,long timeout,TimeUnit timeUnit){
        lock.lock();
        try{
            //将timeout转换为纳秒
            long nanos =timeUnit.toNanos(timeout);
            while (deque.size()==capcity){
                try{
                    System.out.println("任务队列已满");
                    //任务队列已满，等待加入任务队列
                    nanos = fullWait.awaitNanos(nanos);
                    if (nanos<=0){
                        //阻塞添加超时
                        return false;
                    }
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
            deque.addLast(element);
            emptyWait.signal();
            return true;

        }finally {
            lock.unlock();
        }

    }

    //获取队列中的任务数
    public int size(){
        lock.lock();
                //当队列已经满时，执行拒绝策略
                rejectPolicy.reject(this,task);

            }else {
                deque.addLast(task);
                emptyWait.signal();
            }

        }finally {
            lock.unlock();
        }
    }
}
```

+ RejectPolicy

```java
interface RejectPolicy<T> {
    void reject(BlockingQueue<T> queue,T task);
}
```

#### 工作线程

### 实现Runnable接⼝和Callable接⼝的区别

Runnable ⾃Java 1.0以来⼀直存在，但 Callable 仅在Java 1.5中引⼊,⽬的就是为了来处
理 Runnable 不⽀持的⽤例。 Runnable 接⼝不会返回结果或抛出检查异常，但是 Callable 接⼝可以。所以，如果任务不需要返回结果或抛出异常推荐使⽤ Runnable 接⼝，这样代码看起来会更加简洁。⼯具类 Executors 可以实现 Runnable 对象和 Callable 对象之间的相互转换。（ Executors.callable（Runnable task ）或 Executors.callable（Runnable task，
Object resule） ）

+ Runnable

  ```java
  @FunctionalInterface
  public interface Runnable {
   /**
   * 被线程执⾏，没有返回值也⽆法抛出异常
   */
   public abstract void run();
  }
  ```

+ Callable

  ```java
  @FunctionalInterface
  public interface Callable<V> {
   /**
   * 计算结果，或在⽆法这样做时抛出异常。
   * @return 计算得出的结果
   * @throws 如果⽆法计算结果，则抛出异常
   */
   V call() throws Exception;
  }
  ```

### 执⾏execute()⽅法和submit()⽅法的区别？

+  execute() ⽅法⽤于提交不需要返回值的任务，所以⽆法判断任务是否被线程池执⾏成功与 否

  ```java
  public void execute(Runnable command) {
   ...
  }
  ```

+ submit() ⽅法⽤于提交需要返回值的任务。线程池会返回⼀个 Future 类型的对象，通过这个 Future 对象可以判断任务是否执⾏成功，并且可以通过 Future 的 get() ⽅法来获取
  返回值， get() ⽅法会阻塞当前线程直到任务完成，⽽使⽤ get（long timeout，
  TimeUnit unit） ⽅法则会阻塞当前线程⼀段时间后⽴即返回，这时候有可能任务没有执⾏完

  ```java
  public Future<?> submit(Runnable task) {
      if (task WX null) throw new NullPointerException();
      RunnableFuture<Void> ftask = newTaskFor(task, null);
      execute(ftask);
      return ftask;
  }
  
  
  protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T
  value) {
      return new FutureTask<T>(runnable, value);
  }
  ```

### mysql中有哪些索引？聚集索引与非聚集索引解释下？

+ B+树索引：非叶子节点只存储键值和指针，数据记录都存放在叶子节点中，由于一个节点存储了更多的键值和指针，所以同样多的内容可以降低树的高度，减少磁盘io次数，从而提高效率。在B+树中因为叶子节点的键值是按顺序排列的所以进行键值的范围查找效率非常高。
+ hash索引：基于hash表实现，通过对每一行的数据中的索引计算哈希值，存储在hash表中，同时在哈希表中保存指向每行数据的指针，检索效率高，一次定位，只有精确查找时才有效，仅仅使用于"="、“in”，不能使用范围索引、不支持排序、不支持联合索引部分匹配
+ 全文索引：文本数据检索，基于相似度搜索，只有char、varchar、text列才可以创建全文索引

聚集索引与非聚集索引都是基于B+树实现的

+ 聚集索引时基于主键构建，如果没有显式定义主键，会先选择一列不包含null值的唯一索引作为主键索引，如果连这样的唯一索引也没有，InnoDB会选择每行数据中隐藏的6字节的ROWID作为主键索引，它会随着行记录的写入而主键递增。在InnoDB中，每张表只有一个聚集索引
+ 非聚集索引是按照你建的索引构建的，叶子节点并不包含整行记录，但除了索引数值还会包含该行的主键值，当通过非聚集索引来寻找数据时，InnoDB会先遍历非聚集索引找到叶子节点获取主键索引的键值，然后再通过主键索引到聚集索引查找相应的完整行数据。每个表可以有多个辅助索引

### 为什么索引能加快查询速度？

索引(index)是一种高效获取数据的数据结构（排好序的快速查找数据结构），以某种方式指向数据。数据存放于磁盘上，通过索引可以快速定位数据所在的位置，避免多次查找，减少磁盘IO，并且通过索引列队数据进行排序，降低数据排序的成本，降低CPU的消耗。在B+树索引中，IO次数取决于b+数的高度h，假设当前数据表的数据为N，每个磁盘块的数据项的数量是m，则有h=㏒(m+1)N，当数据量N一定的情况下，m越大，h越小；而m = 磁盘块的大小 / 数据项的大小，磁盘块的大小也就是一个数据页的大小，是固定的(每个磁盘块大小为4K)，如果数据项(索引数目及大小)占的空间越小，数据项的数量越多，树的高度越低。这就是为什么每个数据项，即索引字段要尽量的小，比如int占4字节，要比bigint8字节少一半。

### 进程与线程的区别

通常一个进程都有若干个线程，至少包含一个线程

+ 进程是操作系统资源分配的基本单位，而线程是处理器任务调度和执行的基本单位
+ 进程有自己独立的地址空间，每启动一个进程，系统都会为其分配地址空间，建立数据表来维护代码段、堆栈段和数据段，线程没有独立的地址空间，它使用相同的地址空间共享数据，同一进程的线程共享本进程的地址空间和资源，而进程之间的地址空间和资源是相互独立的
+ CPU切换一个线程比切换进程花费小；创建一个线程比进程开销小；线程占用的资源要⽐进程少很多。
+ 线程之间通信更方便，同一个进程下，线程共享全局变量，静态变量等数据，进程之间的通信需要以通信的方式（IPC）进行；（但多线程程序处理好同步与互斥是个难点）
+ 多进程程序更安全，生命力更强，一个进程死掉不会对另一个进程造成影响（源于有独立的地址空间），多线程程序更不易维护，一个线程死掉，整个进程就死掉了（因为共享地址空间）；
+ 进程对资源保护要求高，开销大，效率相对较低，线程资源保护要求不高，但开销小，效率高，可频繁切换；
+ 每个独立的进程有程序运行的入口、顺序执行序列和程序出口。但是线程不能独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制，两者均可并发执行

从JVM角度，一个进程中可以有多个线程，多个线程共享进程的**堆**和**方法区 (JDK1.8 之后的元空间)**资源，但是每个线程有自己的**程序计数器、虚拟机栈和 本地方法栈**



### 数据库的隔离级别，分别有什么问题

+ READ UNCOMMITTED：读未提交，即一个事务可以读取另一个未提交事务的数据。脏读、不可重复读和幻读的问题都会出现

  ```
  不加锁，不同事务之间的操作时彼此可见且互相影响的
  ```

+ READ COMMIITTED：读提交，即一个事务要等另一个事务提交后才能读取最新数据。只允许事务读取其它事务已提交的变更，可以避免脏读，但不可重复读和幻读问题仍然可能出现

  ```
  RC隔离级别下，普通的select都是快照读，使用MVCC实现
  
  加锁的select都使用Record Lock，因为没有Gap Lock
  
  外键约束检查(foreign-key constraint checking)以及重复键检查(duplicate-key checking)时会使用间隙锁封锁区间；所以 RC 会出现幻读的问题。
  ```

+ REPEATABLE READ：重复读，确保事务可以多次从一个字段中读取相同的值(但可能不是最新值)，即事务不会读到其他事务对已有数据的修改，即使其他事务已提交，也就是说，事务开始时读到的已有数据是什么，在事务提交前的任意时刻，这些数据的值都是一样的。可以避免脏读和不可重复读，但幻读问题仍存在。在SQL标准中，RR是无法避免幻读问题的，但是InnoDB实现的RR避免了幻读问题。

  ```
  RR隔离级别下，普通的select使用快照读(snapshot read)，底层使用MVCC来实现
  
  加锁的select(select ... in share mode / select ... for update)以及更新操作update, delete 等语句使用当前读（current read），均使用next-key lock进行加锁。Next-Key Lock是行锁和间隙锁的组合，当InnoDB扫描索引记录的时候，会首先对索引记录加上行锁（Record Lock），再对索引记录两边的间隙加上间隙锁（Gap Lock）。加上间隙锁之后，其他事务就不能在这个间隙修改或者插入记录。
  
  当查询的索引含有唯一属性（唯一索引，主键索引）时，Innodb存储引擎会对next-key lock进行优化，将其降为record lock,即仅锁住索引本身，而不是范围。
  delete from user where id=15 and id=20; //record lock
  delete from user where id in(15,20);  //record lock
  delete from user where id between 5 and 7; //next-key
  ```

+ SERIALIZABLE：序列化，事务串行化顺序执行。确保事务可以从一个表中读取相同的行，在这个事务持续期间，禁止其它事务对该表的插入、更行和删除操作。可以避免脏读、不可重复读、幻读问题但性能低下

  ```java
  Serializable所有的select语句都会被隐式的转化为select ... in share mode，会和update、delete 互斥。
  ```



### Bean作用域

+ singleton：单例模式，Spring IoC容器中只会存在一个共享的Bean实例，无论有多少个Bean引用它，始终指向同一对象。Singleton作用域是Spring中的缺省作用域
+ prototype：原型模式，每次通过Spring容器获取prototype定义的bean时，容器都将创建一个新的Bean实例，每个Bean实例都有自己的属性和状态，而singleton全局只有一个对象。根据经验，对有状态的bean使用prototype作用域，而对无状态的bean使用singleton作用域
+ request：在一次Http请求中，容器会返回该Bean的同一实例。而对不同的Http请求则会产生新的Bean，而且该bean仅在当前Http Request内有效。
+ session：在一次Http Session中，容器会返回该Bean的同一实例。而对不同的Session请求则会创建新的实例，该bean实例仅在当前Session内有效。
+ global Session：在一个全局的Http Session中，容器会返回该Bean的同一个实例，仅在使用portlet context时有效。

```
controller是单例的，也就是说并发请求调用Controller生成的是同一个对象。从线程安全的角度来说，这些线程共享Controller的实例对象。controller是非线程安全的
避免定义全局变量。局部变量不存在线程安全问题
使用ThreadLocal来进行线程隔离
万一必须要定义一个非静态成员变量时候，则通过注解@Scope(“prototype”)，将其设置为多例模式。
```



**其中request、session、global session三种作用域仅适用于web环境**

### Bean的生命周期

![Spring-Bean生命周期](..\image\Spring-Bean生命周期.png)

1. **Bean定义**：通过各种BeanDefinitionReader读取各种配置来源信息（比如读取xml文件、注解等），并将其转化为BeanDefintion的过程。这里要理解一下BeanDefinition的作用。众所周知，Spring提供了多种多样的注册Bean的方法，**BeanDefinition的作用就是去定义并描述一个Spring Bean，方便后续解析实例化等操作**

2. **Bean的初始化**：

   + 实例化bean

   + 设置属性

     ```
     如果Bean实现了BeanNameAware接口，会回调该接口的setBeanName()方法，传入该Bean的id，此时该Bean就获得了自己在配置文件中的id，
     
     如果Bean实现了BeanFactoryAware接口,会回调该接口的setBeanFactory()方法，传入该Bean的BeanFactory，这样该Bean就获得了自己所在的BeanFactory，
     
     如果Bean实现了ApplicationContextAware接口,会回调该接口的setApplicationContext()方法，传入该Bean的ApplicationContext，这样该Bean就获得了自己所在的ApplicationContext，
     ```

   + BeanPostProcessor.postProcessBeforeInitialzation
   + 如果Bean实现了InitializingBean接口，则会回调该接口的afterPropertiesSet()方法
   + 如果Bean配置了init-method方法，则会执行init-method配置的方法
   + BeanPostProcessor.postProcessAfterInitialization()

3. **Bean使用**：对于scope为singleton的Bean,Spring的ioc容器中会缓存一份该bean的实例，而对于scope为prototype的Bean,每次被调用都会new一个新的对象，期生命周期就交给调用方管理了，不再是Spring容器进行管理了

4. **Bean销毁**

   + 容器关闭后，如果Bean实现了DisposableBean接口，则会回调该接口的destroy()方法
   + 如果Bean配置了destroy-method方法，则会执行destroy-method配置的方法，至此，整个Bean的生命周期结束

### 泛型

在创建对象或调用方法的时候才明确具体的类型，使用泛型的好处就是是代码更加简洁(不需要强制转换)，程序更加健壮(在编译期间没有警告，在运行期就不会出现ClassCastException异常)，在使用集合的时候用得挺多的，另外在写基础组件的时候可能会用到，为了通用性，用户传过来的参数可能是类型多样的，这种情况下使用泛型就很不错了，在加上反射在实际需要时获取其具体类型做转换

​		参数化类型，将类型由原来的具体的类型参数化，类似于方法中的变量参数，此时类型也定义成参数形式（可以称之为类型形参），然后在使用/调用时传入具体的类型（类型实参）。泛型指定编译时的类型，减少运行时由于对象类型不匹配引发的异常。其主要用途是提高我们的代码的复用率。实现泛型的目的是在编译时而不是运行时中发现错误。在编译时查找错误可以节省调试Java程序的时间，因为编译时错误更容易查找和修复。泛型类型仅在编译时存在。这是学习Java泛型时要记住的最重要的事情。

​		泛型的本质是为了参数化类型（在不创建新的类型的情况下，通过泛型指定的不同类型来控制形参具体限制的类型）。也就是说在泛型使用过程中，操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中，分别被称为泛型类、泛型接口、泛型方法。

适用于多种数据类型执行相同的代码（代码复用）

泛型中的类型在使用时指定，不需要强制类型转换（类型安全，编译器会检查类型）

泛型提供了编译时类型安全检测机制，该机制允许程序员在编译时检测到非法的类型。泛型主要使用在集合中

+ 泛型类：只能用在成员变量上，只能使用引用类型
+ 泛型接口：只能用在抽象方法上
+ 泛型方法：返回值前面加上 &ltT&gt

```
写一个排序方法，能够对整形数组、字符串数组甚至其他任何类型的数组进行排序，该如何实现？
可以使用 Java 泛型的概念，我们可以写一个泛型方法来对一个对象数组排序。然后，调用该泛型方法来对整型数组、浮点数数组、字符串数组等进行排序。
```

泛型可以在编译的时候检查类型安全，并且所有的强制转换都是自动和隐式的，提高了代码的重用率，只要在编译时期没有出现警告，那么运行时期就不会出ClassCastException异常，把类型明确的工作推迟到创建对象或调用方法的时候才去明确的特殊的类型，在编译时进行更强大的类型检查，消除显式转换。实现更好的代码可重用性，例如通用算法的实现。

写一个泛型方法，该方法在调用时可以接收不同类型的参数。根据传递给泛型方法的参数类型，编译器适当地处理每一个方法调用

泛型可以把一些错误提前到了编译时期，方便了调试。而且在迭代时，不再需要强制转化。ArrayList al=new ArrayList();ArrayList可以添加任何类型的元素，该方法在程序编译时并不会出现问题。但是在迭代取值时，Integer类型的值无法转换为String，会出现ClassCastException异常。而避免这种问题的，就是指定ArrayList的元素的类型，把类型参数化。ArrayList<String> al=new ArrayList<String>();

### HashTable和HashMap的区别

1. 继承父类不同

```java
public class HashMap<K,V> 
    extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable 
```

```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, Serializable 
```

2. 对NULL的支持不同

+ Hashtable既不支持Null key也不支持Null value

  ```java
  public synchronized V put(K key, V value) {
          // Make sure the value is not null
          //如果值为空那么会报空指针异常
          if (value == null) {
              throw new NullPointerException();
          }
  
          // Makes sure the key is not already in the hashtable.
          Entry<?,?> tab[] = table;
          //如果key为空，调用hashCode()方法会报空指针异常
          int hash = key.hashCode();
  ```

+ HashMap中，key可以为null（但这样的键只有一个）可以有一个或多个键所对应的值为null。当get()方法返回null值时，可能是 HashMap中没有该键，也可能使该键所对应的值为null。因此，在HashMap中不能由get()方法来判断HashMap中是否存在某个键， 而应该用containsKey()方法来判断。

3. 线程安全性不同

   Hashtable是线程安全的，它的每个方法中都加入了Synchronize方法。在多线程并发的环境下，可以直接使用Hashtable，不需要自己为它的方法实现同步

   HashMap不是线程安全的，在多线程并发的环境下，可能会产生死锁等问题。具体的原因在下一篇文章中会详细进行分析。使用HashMap时就必须要自己增加同步处理，虽然HashMap不是线程安全的，但是它的效率会比Hashtable要好很多


4. 初始容量大小和每次扩充容量大小的不同

   Hashtable默认的初始大小为11，之后每次扩充，容量变为原来的2n+1。HashMap默认的初始化大小为16。之后每次扩充，容量变为原来的2倍。

   创建时，如果给定了容量初始值，那么Hashtable会直接使用你给定的大小，而HashMap会将其扩充为2的幂次方大小。也就是说Hashtable会尽量使用素数、奇数。而HashMap则总是使用2的幂作为哈希表的大小。

### synchronized关键字的作用和底层实现

synchronized采用互斥的方式让同一时刻至多只有一个线程持有对象锁，其它线程在想要获取这个对象锁时就会被阻塞住，这样就能保证拥有锁的线程可以安全地执行临界区内的代码，不用担心线程上下文切换。在java中互斥和同步都可以采用synchronized来实现，但它们还是有区别的

 synchronized是基于进入和退出montior对象来实现，包括显示同步(synchronized修饰同步代码块时，同步代码块的字节码反编译之后会有明确的monitorenter和monitoexit指令)，还有隐式同步(synchronized修饰方法，同步方法的字节码反编译之后会有个标志位ACC_SYNCHRONIZED,线程在执行方法前会先去获取对象的monitor对象，如果获取成功则执行方法代码，执行完毕后释放monitor对象，如果monitor对象已经被其它线程获取，那么当前线程被阻塞)

### volatile

volatile的可以保证变量内存的可见性且并且可以提供了一定的有序性，禁止指令重排，但是无法保证原子性。 被volatile修饰的变量能够保证每个线程能够获取该变量的最新值，从而避免出现数据脏读的现象。volatile可以用来修饰成员变量和静态成员变量，可以避免线程从自己的工作缓存中查找变量值，必须到主存中获取它的值，线程操作volatile变量都是直接操作主存

​		Java内存模型(Java Memory Mode)规定所有的共享变量都是存在主存中，每个线程都有自己的工作内存。线程对变量的所有操作都是在自己的工作内存中进行，不能直接读写主内存中的变量，并且每个线程不能直接访问其他线程的工作内存，线程间变量的值的传递需要通过主内存中转来完成。这样就会造成可见性问题，可能会导致线程无法将工作内存中的修改值更新到主存中，也可能导致线程没有及时将主存中的最新值同步到工作内存中，总之，线程在工作内存中的共享变量值可能并不是最新的

​		在生成汇编代码时会在volatile修饰的共享变量进行写操作的时候会多出**Lock前缀的指令**，Lock前缀指令的影响主要有两方面：

**I.修改缓存行时会强制将数据写会主存**

**II.这个写回内存的操作会使得其他CPU里缓存了该内存地址的数据无效**

当处理器发现本地缓存失效后，就会从内存中重读该变量数据，即可以获取当前最新值

针对volatile变量通过这样的机制就使得每个线程都能获得该变量的最新值

### JVM的内存模型 ，JVM堆中每一代采用的垃圾回收算法以及为什么使用这种算法

+ 程序计数器：存储当前线程正在执行的Java方法的jvm指令地址
+ 虚拟机栈：负责java程序的运行，保存方法的局部变量、部分结果，并参与方法的调用和返回，每个线程在创建时都会创建一个虚拟机栈，其内存保存着一个个栈帧（stack frame）,对应着一次次的java方法调用
+ 本地方法栈：本地方法栈则用于管理本地方法的调用（其内存大小可以设置成固定值也可以动态分配，在内存溢出方面和虚拟机栈是相同的）本地方法是使用C语言实现的，在本地方法栈登记的native方法在Execution Engine执行时加载本地方法库
+ 堆：几乎所有的对象实例和数组都在堆中分配内存（经逃逸分析后对象可能栈上分配）在方法结束后，堆中的对象不会马上移除，仅仅在垃圾收集的时候才会被移除
+ 方法区：方法区用于存储类型信息、运行时常量池、域信息、方法信息、JIT代码缓存等内容（在Metaspace中将静态变量与字符串常量池放入堆中）方法区在JVM启动的时候被创建，它的大小可以选择固定大小或可拓展。方法区的大小决定了系统可以保存多少个类，如果系统定义了太多的类会导致方法区溢出错误。

可达性分析算法(Tracing Garbage Collection)：以根对象集合(GC Roots)为起始点，按照从上至下的方式搜索被根对象集合所连接的目标对象是否可达。使用可达性分析后，内存中的存活对象都会被根对象集合直接或间接连接着，搜索所走过的路径称为引用链（Reference Chain）。如果目标对象没有任何引用链相连，则是不可达的，就意味着该对象已经死亡，可以标记为垃圾对象。在可达性分析算法中，只有能够被根对象集合直接或者间接连接的对象才是存活对象。**如果要使用可达性分析算法来判断内存是否可回收，那么分析工作必须在一个能保障一致性的快照中进行，必须满足这点才能保证分析结果的准确性，这点是导致GC进行时必须Stop The World的重要原因之一**



GC Roots元素(简单来说，如果指针保存了堆内存里面得对象，但是自己又不存放在堆里面，那么它就是一个Root)

+ 虚拟机栈中引用的对象（各线程被调用的方法中使用到的参数、局部变量）
+ 本地方法栈(本地方法)引用的对象
+ 方法区中类静态属性引用的对象(Java类的引用类型静态变量
+ 方法区中常量引用的对象(字符串常量池的引用)
+ 所有被同步锁synchronized持有的对象
+ Java虚拟机内部的引用(基本数据类型对应的Class对象)
+ 反映java虚拟机内部情况的JMXBean、JVM中注册的回调、本地代码缓存等



+ 标记-清除算法(Mark-Sweep)：当堆中的有效内存空间(available memory)被耗尽的时候，就会停止整个程序(Stop the world)，然后进行两项工作，第一项是标记，第二项是清除

  + 标记：Collector从引用根节点开始遍历，标记所有**被引用**的对象，一般是在对象的Header中记录为可达对象
  + 清除：Conllector对堆内存从头到尾进行线性的遍历，如果发现某个对象在其Header中没有标记可达对象，则将其回收（这里所谓的清除并不是真的置空，而是把需要清除的对象地址保存在空闲的地址列表里，下次有新对象需要加载时，判断垃圾的位置空间是否够，如果够就存放）

  缺点：效率不算高；在进行GC的时候，需要停止整个应用程序，导致用户体验差；这种方式清理出来的空闲内存时不连续的，产生内存碎片，需要维护一个空闲列表

+ 复制算法(Copying)：将活着的内存空间分为两块，每次只使用其中一块，在垃圾回收时将正在使用的内存中的存活对象复制到未使用的内存块中，之后清除正在使用的内存块中的所有对象，交换两个内存的角色，最后完成垃圾回收。没有标记和清除过程，实现简单，运行高效；复制过去以后保证空间的连续性，不会出现“碎片”问题.在新生代，对常规应用的垃圾回收，一次通常可以回收70%-99%的内存空间，回收性价比很高。现在商业虚拟机都是用这种收集算法回收新生代（复制算法的高效性是建立在存活对象少、垃圾对象多的前提下的，适合新生代并不适合老年代）

+ 标记-压缩算法(Mark-Compact)：标记-压缩算法的最终结果等同于标记-清除算法执行完成后，再进行一次内存碎片整理。二者的本质差异在于标记-清除算法是一种非移动式的回收算法，标记-压缩是移动式的。

  + 标记：从根节点开始标记所有被引用的对象
  + 压缩：将所有的存活对象压缩到内存的一端，按顺序排放；之后，清理边界所有的空间

  优点：消除了标记-清除算法中内存区域分散的缺点，当需要给新对象分配内存时，JVM只需要持有一个内存的起始地址即可；消除了复制算法中内存减半的高额代价

  缺点：标记-整理算法的效率要低于复制算法；移动对象过程中需要STW；移动对象的同时，如果对象被其他对象引用则还需要调整引用的地址

  

  | GC           | 分类     | 工作区域  | 实现            | 特点         | 适用场景                           |
  | ------------ | -------- | --------- | --------------- | ------------ | ---------------------------------- |
  | Serial       | 串行     | 新生代    | 复制            | 响应速度优先 | 单CPU环境下的client模式            |
  | ParNew       | 并行     | 新生代    | 复制            | 响应速度优先 | 多CPU环境Server模式下雨CMS配合使用 |
  | Parallel     | 并行     | 新生代    | 复制            | 吞吐量优先   | 后台运算而不需要太多交互的场景     |
  | Serial Old   | 串行     | 老年代    | 标记-压缩       | 响应速度优先 | 单CPU环境下的Client模式            |
  | Parallel Old | 并行     | 老年代    | 标记-压缩       | 吞吐量优先   | 后台运算而不需要太多交互的场景     |
  | CMS          | 并发     | 老年代    | 标记-清除       | 响应速度优先 | 互联网或B/S业务                    |
  | G1           | 并发并行 | 新/老生代 | 标记-压缩、复制 | 响应速度优先 | 面向服务端应用                     |



### 对象创建过程

1. 判断对象对应的类是否加载、链接、初始化：当JVM在执行new指令的时，首先去检查这个指令的参数能否在Metaspace的常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已经被加载、解析和初始化（即【判断类元信息是否存在）如果没有，则在双亲委派模式下使用当前类加载器以ClassLoader+包名+类名为key查找对应的.class文件；如果没找到文件，则抛出ClassNotFoundException异常；如果找到则进行类加载，并生成对应的Class类对象
2. 为对象分配内存：首先计算对象占用空间大小，接着在堆种划分一块内存给对象。如果实例成员变量是引用变量，仅分配引用变量空间即可（4个字节大小）。内存分配的方式取决Java堆是否规整，而Java堆是否规整又取决于所采用的垃圾收集齐是否有压缩整理功能
   + 如果内存规整，JVM使用指针碰撞法（Bump The Pointer）来为对象分配内存。所有用过的内存在一边，空闲的内存在另一边，中间放着一个指针作为分界点的指示器，分配内存就仅仅是把指针向空闲那边挪动一段与对象大小相等的距离罢了。如果垃圾收集器选择的是Serial、ParNew这种基于压缩算法的，虚拟机采用这种分配方式。一般使用带有compact过程的收集器时，使用指针碰撞。
   + 如果内存不规整，JVM需要维护一个列表，使用空闲列表分配。如果内存是不规整的，已使用的内存和未使用的内存相互交错，那么虚拟机将采用的是空闲列表法来为对象分配内存。虚拟机维护一个列表，记录哪些内存块是可用的，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的内容

3. 处理并发安全问题：在分配内存空间时，需要及时保证new对象时候的线程并发安全性
   + CAS(Compare And Swqp)失败重试、区域加锁：保证指针更新操作的原子性
   + TLAB把内存分配的动作按照线程划分在不同的空间之中进行，即每个线程在Java堆中预先分配一小块内存，称为本地线程分配缓冲区。
4. 初始化分配到的空间：内存分配结束，虚拟机将分配到的内存空间都初始化为零值(不包括对象头)。这一步保证了对象的实例字段在Java代码可以不用赋值初始值就可以直接使用，程序能访问到这些字段的数据类型所对应的零值。
5. 设置对象的对象头：将对象的所属类（即类的元数据信息）、对象的HashCode和对象的GC信息、锁信息等数据存储在对象的对象头中这个过程的具体设置方式取决于JVM实现
6. 执行init方法进行进行初始化：在Java程序的视角看来，初始化才正式开始。初始化成员变量，执行实例化代码块，调用类的构造方法，并把堆内对象的首地址赋值给引用变量。因此一般来说（由字节码是否跟随invokespecial指令所决定），new指令之后会接着就是执行方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全创建出来。

