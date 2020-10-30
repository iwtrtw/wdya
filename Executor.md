### Executor体系

在实际使用中，线程是很占用系统资源的，如果对线程管理不善很容易导致系统问题。因此，在大多数并发框架中都会使用**线程池**来管理线程，使用线程池管理线程主要有如下好处

+ 降低资源消耗：通过复用已存在的线程和降低线程的关闭次数来尽可能降低系统性能损耗
+ 提升系统响应速度：通过复用线程，省去创建线程的过程，因此整体上可以提升系统的响应速度
+ 提高线程的可管理性：线程是比较消耗系统资源，还会降低系统的稳定性，因此，需要使用线程池来管理线程。

线程池：任务队列、线程集合、核心线程数、获取任务的超时时间、拒绝策略

线程池执行所提交的任务过程主要有这样几个阶段：

1. 先判断线程池中**核心线程池**所有的线程是否都在执行任务。如果不是，则新创建一个线程执行刚提交的任务，否则，核心线程池中所有的线程都在执行任务，则进入第2步
2. 判断当前**阻塞队列**是否已满，如果未满，则将提交的任务放置在阻塞队列中；否则，则进入第3步
3. 判断**线程池中所有的线程**是否都在执行任务，如果没有，则创建一个新的线程来执行任务，否则，则交给饱和策略进行处理

#### ThreadPoolExecutor

```
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
```

+ corePoolSize：表示核心线程池的大小。当提交一个任务时，如果当前核心线程池的线程个数没有达到corePoolSize，则会创建新的线程来执行所提交的任务，**即使当前核心线程池有空闲的线程**。如果当前核心线程池的线程个数已经达到了corePoolSize，则不再重新创建线程。
+ maximumPoolSize：表示线程池能创建线程的最大个数。如果当阻塞队列已满时，并且当前线程池线程个数没有超过maximumPoolSize的话，就会创建新的线程来执行任务。
+ keepAliveTime：空闲线程存活时间。如果当前线程池的线程个数已经超过了corePoolSize，并且线程空闲时间超过了keepAliveTime的话，就会将这些空闲线程销毁，这样可以尽可能降低系统资源消耗
+ unit：时间单位。为keepAliveTime指定时间单位。
+ workQueue：阻塞队列。用于保存任务的阻塞队列，可以使用**ArrayBlockingQueue, LinkedBlockingQueue, SynchronousQueue, PriorityBlockingQueue**。
+ threadFactory：创建线程的工程类。可以通过指定线程工厂为每个创建出来的线程设置更有意义的名字，如果出现并发问题，也方便查找问题原因。
+ handler：饱和策略。当线程池的阻塞队列已满和指定的线程都已经开启，说明当前线程池已经处于饱和状态了，那么就需要采用一种策略来处理这种情况。
  + AbortPolicy：直接拒绝所提交的任务，并抛出**RejectedExecutionException**异常
  + CallerRunsPolicy：让调用者所在的线程来执行任务
  + DiscardPolicy：不处理直接丢弃掉任务
  + DiscardOldestPolicy：丢弃掉阻塞队列中存放时间最久的任务，执行当前任务

execute

1. 如果当前运行的线程少于corePoolSize，则会创建新的线程来执行新的任务
2. 如果运行的线程个数等于或者大于corePoolSize，则会将提交的任务存放到阻塞队列workQueue中
3. 如果当前workQueue队列已满的话，则会创建新的线程来执行任务
4. 如果线程个数已经超过了maximumPoolSize，则会使用饱和策略RejectedExecutionHandler来进行处理。

```java
public void execute(Runnable command) {
        if (command == null)
        int c = ctl.get();
        //如果线程池的线程个数少于corePoolSize则创建新线程执行当前任务
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        //如果线程个数大于corePoolSize或者创建线程失败，则将任务存放在阻塞队列workQueue中
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
    	//如果当前任务无法放进阻塞队列中，则创建新的线程来执行任务
        else if (!addWorker(command, false))
            reject(command);
    }
```

#### ScheduledThreadPoolExecutor



