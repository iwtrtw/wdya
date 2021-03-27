锁可以用来控制多个线程访问共享资源的方式，一般来说，一个锁能够防止多个线程同时访问共享资源。在Lock接口出现之前，java程序主要是靠synchronized关键字实现锁功能的，而java SE5之后，并发包中增加了lock接口，它提供了与synchronized一样的锁功能。**虽然它失去了像synchronize关键字隐式加锁解锁的便捷性，但是却拥有了锁获取和释放的可操作性，可中断的获取锁以及超时获取锁等多种synchronized关键字所不具备的同步特性。**通常使用显示使用lock的形式如下：

```java
Lock lock = new ReentrantLock();
lock.lock();
try{
	.......
}finally{
	lock.unlock();
}
```

与synchronized不同是，采用synchronized会自动释放锁，当synchronized方法或者synchronized代码块执行完之后，系统会自动让线程释放对锁的占用；而Lock则必须要用户去手动释放锁，否则可能出现死锁。

### Lock

```java
public interface Lock {
    //获取锁
    void lock();
    
    //获取锁的过程能够响应中断
    void lockInterruptibly() throws InterruptedException;

    //非阻塞式响应中断获取锁，立刻返回结果，获得锁放回true反之返回fasle
    boolean tryLock();
   
    //超时获取锁，在超时内或者未中断的情况下能够获取锁返回true反之返回false
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

    //释放锁
    void unlock();

    //获取与lock绑定的等待通知组件，当前线程必须获得了锁才能进行等待，进行等待时会先释放锁，当再次获取锁时才能从等待中返回
    Condition newCondition();
}
```

### AQS

AQS是一个同步器，设计模式是模板方法模式

核心数据结构：双向链表 + state(锁状态)
底层操作：CAS

AQS是一个用来构建锁和同步器的框架，它维护了一个共享资源 state 和一个 FIFO 的等待队列(即上文中管程的入口等待队列)，底层利用了 CAS 机制来保证操作的原子性。state 由于是多线程共享变量，所以必须定义成 volatile，以保证 state 的可见性, 同时虽然 volatile 能保证可见性，但不能保证原子性，所以 AQS 提供了对 state 的原子操作方法，保证了线程安全。

独占锁：其他线程只有在占有锁的线程释放后才能竞争锁，有且只有一个线程能竞争成功（ReentrantLock ）

共享锁：共享资源可以被多个线程同时占有，直到共享资源被占用完毕（ReadWriteLock, CountdownLatch）

同步器是用来构建锁和其他同步组件的基础框架，它的实现主要依赖一个int成员变量来表示同步状态以及通过一个FIFO队列构成等待队列。它的**子类必须重写AQS的几个protected修饰的用来改变同步状态的方法**，其他方法主要是实现了排队和阻塞机制。**状态的更新使用getState,setState以及compareAndSetState这三个方法**。

子类被**推荐定义为自定义同步组件的静态内部类**，同步器自身没有实现任何同步接口，它仅仅是定义了若干同步状态的获取和释放方法来供自定义同步组件的使用，同步器既支持独占式获取同步状态，也可以支持共享式获取同步状态，这样就可以方便的实现不同类型的同步组件。

同步器是实现锁（也可以是任意同步组件）的关键，在锁的实现中聚合同步器，利用同步器实现锁的语义。可以这样理解二者的关系：**锁是面向使用者，它定义了使用者与锁交互的接口，隐藏了实现细节；同步器是面向锁的实现者，它简化了锁的实现方式，屏蔽了同步状态的管理，线程的排队，等待和唤醒等底层操作**。锁和同步器很好的隔离了使用者和实现者所需关注的领域。

AQS的设计是使用模板方法设计模式，它将**一些方法开放给子类进行重写，而同步器给同步组件所提供模板方法又会重新调用被子类所重写的方法**。自定义同步器在实现时只需要实现共享资源state的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。

自定义同步器实现时主要需要重写以下几种方法：

> isHeldExclusively()：该线程是否正在独占资源。只有用到condition才需要去实现它。
>
> tryAcquire(int)：独占方式。尝试获取资源，成功则返回true，失败则返回false。
>
> tryRelease(int)：独占方式。尝试释放资源，成功则返回true，失败则返回false。
>
> tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
>
> tryReleaseShared(int)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false

AQS的核心包括了:**同步队列、独占式锁的获取和释放、共享锁的获取和释放以及可中断锁**

+ 同步队列：一个双向队列，AQS通过持有头尾指针管理同步队列，同时实现包括获取锁失败的线程进行入队，释放锁时对同步队列中的线程进行通知等核心方法。

```java
//等待队列头结点
private transient volatile Node head;
//等待队列尾节点
private transient volatile Node tail;

//静态内部类
static final class Node {
    	//共享模式
    	static final Node SHARED = new Node();
		
    	//互斥模式
        static final Node EXCLUSIVE = null;

    	//表示线程已被取消
        static final int CANCELLED =  1;
		
  //表示后继结点在等待当前结点唤醒。后继结点入队时，会将前继结点的状态更新为SIGNAL
        static final int SIGNAL    = -1;

    	//线程正在等待在Condition状态
        static final int CONDITION = -2;

    //共享模式下：前继结点不仅会唤醒其后继结点，同时也可能会唤醒后继的后继结点。
        static final int PROPAGATE = -3;

       	//节点状态
        volatile int waitStatus;
    	
    	//前驱节点
        volatile Node prev;
    	
    	//后继节点
        volatile Node next;
    	
    	//使该节点进入队列的线程
        volatile Thread thread;
    	
    	//等待队列中的下一个节点
        Node nextWaiter;

        final boolean isShared() {
            return nextWaiter == SHARED;
        }

   
        final Node predecessor() throws NullPointerException {
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }

        Node() {  
        }

        Node(Thread thread, Node mode) {     
            this.nextWaiter = mode;
            this.thread = thread;
        }

        Node(Thread thread, int waitStatus) { 
            this.waitStatus = waitStatus;
            this.thread = thread;
        }
    }
```

+ 独占式锁

> void acquire(int arg)：独占式获取同步状态，如果获取失败则插入同步队列进行等待
>
> void acquireInterruptibly(int arg)：与acquire方法相同，但在同步队列中进行等待的时候可以检测中断
>
> boolean tryAcquireNanos(int arg, long nanosTimeout)：在acquireInterruptibly基础上增加了超时等待功能，在超时时间内没有获得同步状态返回false; 
>
> boolean release(int arg)：释放同步状态，该方法会唤醒在同步队列中的下一个节点

+ 共享锁

> void acquireShared(int arg)：共享式获取同步状态，与独占式的区别在于同一时刻有多个线程获取同步状态
>
> void acquireSharedInterruptibly(int arg)：在acquireShared方法基础上增加了能响应中断的功能
>
> boolean tryAcquireSharedNanos(int arg, long nanosTimeout)：在acquireSharedInterruptibly基础上增加了超时等待的功能
>
> boolean releaseShared(int arg)：共享式释放同步状态

#### 独占锁

线程通过tryAcquire方法第一次获取锁，当获取锁失败时，线程会被封装成Node进入队列，主要是通过addWaiter()和enq()，enq()负责完成同步队里头节点的初始化以及尾插节点CAS操作失败的不断重试。在同步队列中，线程获取锁是自旋的，当且仅当当前节点是第二节点是头节点并成功获取锁时，线程才会被移除队列，否则，当不满足条件时就会调用LockSupport.park()方法使得线程阻塞，等待被唤醒。当释放锁时会唤醒后继节点

##### acquire

以独占方式获取资源，如果获取到资源，线程直接返回，否则进入等待队列

```java
public final void acquire(int arg) {
    //先尝试获取资源，如果成功则直接返回
		//若失败则先调用addWaiter()方法再调用acquireQueued()方法
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
}
```

> tryAcquire()尝试直接去获取资源，如果成功则直接返回[需要实现]
>
> addWaiter()将该线程加入等待队列的尾部，并标记为独占模式；
>
> acquireQueued()使线程在等待队列中获取资源，一直获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。
>
> 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断selfInterrupt()，将中断补上。

+ tryAcquire(int)：尝试以独占的方式获取资源，如果获取成功，则直接返回true，否则直接返回false。该方法的默认实现是抛出异常，具体实现由自定义的扩展了AQS的同步类来实现。

```java
protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
}
```

+ addWaiter(Node)：将当前线程根据不同的模式（`Node.EXCLUSIVE`互斥模式、`Node.SHARED`共享模式）加入到等待队列的队尾，并返回当前线程所在的结点。

```java
private Node addWaiter(Node mode) {
    	//构建当前线程成Node
        Node node = new Node(Thread.currentThread(), mode);
        Node pred = tail;
    	//判断队列是否为空
        if (pred != null) {
      //通过CAS方式将当前节点插入等待队列末尾,如果失败会交给enq(Node)死循环重试
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
    	//队列为空，或者compareAndSetTail失败会进入enq(node)
        enq(node);
        return node;
    }
```

+ + enq(Node)：在当前线程是第一个加入同步队列时，初始化头结点；然后自旋不断尝试CAS尾插入节点直至成功为止

```java
private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) {
                //初始化头结点
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                //以CAS自旋的方式进行，直到成功加入队尾为止。
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
}
```

+ acquireQueued：排队获取锁，如果当前节点的前驱节点是头节点，并且能够获得同步状态的话，当前线程能够获得锁该方法执行结束退出；获取锁失败的话，先将节点状态设置成SIGNAL，然后调用LookSupport. park方法使得当前线程阻塞。

```java
final boolean acquireQueued(final Node node, int arg) {
    	 //标记是否成功拿到资源，默认false
        boolean failed = true;
        try {
            //标记等待过程中是否被中断过
            boolean interrupted = false;
            for (;;) {
                //获得当前节点的先驱节点
                final Node p = node.predecessor();
               //如果当前节点的先驱节点是头结点则尝试继续获取独占式锁
                if (p == head && tryAcquire(arg)) {
                    //获取锁成功，出队操作
                    
                    //队列头结点引用指向当前节点,将当前节点信息清空
                    setHead(node);             
//之前的头结点p的next域设置为null与队列断开，无任何引用方便GC时能够将内存进行回收
                    p.next = null; 
                    failed = false;
                    return interrupted;
                }
                //获取锁失败,检查当前节点是否应该被park
                if (shouldParkAfterFailedAcquire(p, node) &&
                    //阻塞当前线程
                    parkAndCheckInterrupt())
        //如果等待过程中被中断过，哪怕只有那么一次，就将interrupted标记为true
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
}
```

+ + shouldParkAfterFailedAcquire：使用CAS将节点状态由INITIAL设置成SIGNAL，表示当前线程阻塞。当compareAndSetWaitStatus设置失败则说明shouldParkAfterFailedAcquire方法返回false，然后会在acquireQueued()方法中for (;;)死循环中会继续重试，直至compareAndSetWaitStatus设置节点状态位为SIGNAL时，shouldParkAfterFailedAcquire返回true时才会执行方法parkAndCheckInterrupt()方法

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)
            return true;
        if (ws > 0) {
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
        //使用CAS将节点状态由INITIAL设置成SIGNAL，表示当前线程阻塞
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
}
```

+ + parkAndCheckInterrupt

```java
private final boolean parkAndCheckInterrupt() {
    	//阻塞当前节点
        LockSupport.park(this);
        return Thread.interrupted();
}
```

##### release

独占模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果彻底释放了（即state=0）,它会唤醒等待队列里的其他线程来获取资源。

```java
public final boolean release(int arg) {
    	//释放锁
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
}
```

+ unparkSuccessor：首先获取头节点的后继节点，然后调用LookSupport.unpark()方法唤醒后继节点所包装的线程。

```java
private void unparkSuccessor(Node node) {
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);
    	//头节点的后继节点
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            //后继节点不为null时唤醒该线程
            LockSupport.unpark(s.thread);
    }
```

#### 共享锁

##### acquireShared

获取指定量的资源，获取成功则直接返回，获取失败则进入等待队列，直到获取到资源为止，整个过程忽略中断。tryAcquireShared返回值是一个int类型，当返回值为大于等于0的时候方法结束说明获得成功获取锁，否则，表明获取同步状态失败即所引用的线程获取锁失败，会执行doAcquireShared方法。

```java
public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
}
```

+ doAcquireShared：将当前线程加入等待队列尾部休息，直到其他线程释放资源唤醒自己，自己成功拿到相应量的资源后才返回。自旋过程中能够退出的条件是当前节点的前驱节点是头结点并且tryAcquireShared(arg)返回值大于等于0即能成功获得同步状态。

```java
private void doAcquireShared(int arg) {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        // 当该节点的前驱节点是头结点且成功获取同步状态
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

##### releaseShared

释放指定量的资源，如果成功释放且允许唤醒等待线程，它会唤醒等待队列里的其他线程来获取资源。

```java
public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
}
```

+ doReleaseShared：在共享式锁的释放过程中，对于能够支持多个线程同时访问的并发组件，必须保证多个线程能够安全的释放同步状态，这里采用的CAS保证，当CAS操作失败continue，在下一次循环中进行重试。

```java
private void doReleaseShared() {
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
                    unparkSuccessor(h);
                }
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }
```

### ReentrantLock

ReentrantLock支持重入性、支持公平锁和非公平锁、支持多条件变量。可重入是指能够对共享资源能够重复加锁，即当前线程以获取某锁当再次获取的时候不会被阻塞。所谓公平与不公平，是针对获取锁而言的，如果一个锁是公平的，那么锁的获取顺序就应该符合请求上的绝对时间顺序，满足FIFO，先到先得。公平锁与非公平锁的区别在于它的tryAcquires，公平锁每次获取到锁为同步队列中的第一个节点，**保证请求资源时间上的绝对顺序**，而非公平锁有可能刚释放锁的线程下次继续获取该锁，则有可能导致其他线程永远无法获取到锁，**造成“饥饿”现象**。

​		公平锁为了保证时间上的绝对顺序，需要频繁的上下文切换，而非公平锁会降低一定的上下文切换，降低性能开销。因此，ReentrantLock默认选择的是非公平锁，则是为了减少一部分上下文切换，**保证了系统更大的吞吐量**。

```java
public class ReentrantLock implements Lock,java.io. Serializable{
    private static final long serialVersionUID = 7373984872572414699L;

    private final Sync sync;
    
    //同步工具，AQS静态内部类
    abstract static class Sync extends AbstractQueuedSynchronizer{
        ...
    }
    
    //非公平锁
    static final class NonfairSync extends Sync{
        ...
    }
    
    //公平锁
    static final class FairSync extends Sync{
        ...
    }
    
    //默认使用NonfairSync
    public ReentrantLock() {
        sync = new NonfairSync();
    }
    
    //传参指定是否公平
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }

    
    public void lock() {
        sync.lock();
    }
    
    public void unlock() {
        sync.release(1);
    }
```

##### Sync

实现可重入，每次重新获取都会对同步状态进行加一的操作；重入锁的释放必须得等到同步状态为0时锁才算成功释放，否则锁仍未释放。如果锁被获取n次，释放了n-1次，该锁未完全释放返回false，只有被释放n次才算成功释放，返回true。

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -5179523762034025860L;
    
        abstract void lock();
    
    	//非公平可重入（没有考虑同步队列中情况）
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            //判断state状态是否为0，如果该锁未被任何线程占有，该锁能被当前线程获取
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            //若被占有，检查占有线程是否是当前线程
            else if (current == getExclusiveOwnerThread()) {
                //如果是的话，同步状态加1返回true
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    
    	//重入锁释放
        protected final boolean tryRelease(int releases) {
            //同步状态减1
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            //只有当同步状态为0时，锁成功被释放，返回true
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            // 锁未被完全释放，返回false
            setState(c);
            return free;
        }

        protected final boolean isHeldExclusively() {
            return getExclusiveOwnerThread() == Thread.currentThread();
        }

        final ConditionObject newCondition() {
            return new ConditionObject();
        }

        final Thread getOwner() {
            return getState() == 0 ? null : getExclusiveOwnerThread();
        }

        final int getHoldCount() {
            return isHeldExclusively() ? getState() : 0;
        }

        final boolean isLocked() {
            return getState() != 0;
        }

        private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
            s.defaultReadObject();
            setState(0); // reset to unlocked state
        }
    }
```

##### NonfairSync

直接调用父类Sync中的nonfairTryAcquire()实现tryAcquire非公平可重入争夺锁，而释放锁也是使用父类Sync中的trytryRelease实现可重入锁的解锁

```java
static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

    	//加锁
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                //使用AQS提供模板方法获取锁
                acquire(1);
        }

    	//重写AQS中的tryAcquire方法，实现可重入非公平锁
        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
```

##### FairSync

tryAcquire实现公平可重，和NonfairSync不同的是添加一个AQS.hasQueuedPredecessors判断，用于判断当前线程可以去获取锁，如果同步队列为空则可以获取锁；如果不为空，只有当前线程第2个节点时才可以获取锁。这样可以保证每次是同步队列中第一个线程获取锁，保证了公平性。释放锁使用的是父类Sync中的trytryRelease实现可重入锁的解锁

```java
static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);
        }
    
    	//公平可重入锁
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```

AQS.hasQueuedPredecessors

```java
public final boolean hasQueuedPredecessors() {
        Node t = tail; 
        Node h = head;
        Node s;
    /*
    h!=t 表示队列中有Node
    s = h.next == null 队列中没有第二个节点
    s.thread != Thread.currentThread() 队列中第二个节点不是此线程
    */
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

### ReentrantReadWriteLock

读写所允许同一时刻被多个读线程访问，但是在写线程访问时，所有的读线程和其他的写线程都会被阻塞。

+ **公平性选择**：支持非公平性（默认）和公平的锁获取方式，吞吐量还是非公平优于公平
+ **重入性**：支持重入，读锁获取后能再次获取，写锁获取之后能够再次获取写锁，同时也能够获取读锁；
+ **锁降级**：遵循获取写锁，获取读锁再释放写锁的次序，写锁能够降级成为读锁

```java
public class ReentrantReadWriteLock
        implements ReadWriteLock, java.io.Serializable {
    private static final long serialVersionUID = -6992448646407690164L;
    
    //读锁
    private final ReentrantReadWriteLock.ReadLock readerLock;
    
    //写锁
    private final ReentrantReadWriteLock.WriteLock writerLock;
    
   
    final Sync sync;
    
    public ReentrantReadWriteLock() {
        this(false);
    }
    
    public ReentrantReadWriteLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
        readerLock = new ReadLock(this);
        writerLock = new WriteLock(this);
    }
```

##### Sync

同步状态的低16位用来表示写锁的获取次数，同步状态的高16位用来表示读锁被获取的次数，当读锁已经被读线程获取或者写锁已经被其他写线程获取，则写锁获取失败；否则，获取成功并支持重入，增加写状态

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 6317671515068378041L;

        static final int SHARED_SHIFT   = 16;
        static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
        static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
    	//EXCLUSIVE_MASK：0x0000FFFF
        static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
       	
    	//取state高16位，表示读锁的获取次数
        static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
    	
    	//取state低16位，表示写锁的获取次数
        static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }

        static final class HoldCounter {
            int count = 0;
            
            final long tid = getThreadId(Thread.currentThread());
        }

        static final class ThreadLocalHoldCounter
            extends ThreadLocal<HoldCounter> {
            public HoldCounter initialValue() {
                return new HoldCounter();
            }
        }

        private transient ThreadLocalHoldCounter readHolds;

        private transient HoldCounter cachedHoldCounter;

        private transient Thread firstReader = null;
    
        private transient int firstReaderHoldCount;

        Sync() {
            readHolds = new ThreadLocalHoldCounter();
            setState(getState()); // ensures visibility of readHolds
        }

        abstract boolean readerShouldBlock();

        abstract boolean writerShouldBlock();
    
    	//写锁释放
        protected final boolean tryRelease(int releases) {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            //同步状态-写状态
            int nextc = getState() - releases;
            //若当前写状态为0，则释放写锁
            boolean free = exclusiveCount(nextc) == 0;
            if (free)
                setExclusiveOwnerThread(null);
            //不为0则更新同步状态
            setState(nextc);
            return free;
        }

    	//写锁获取：当读锁已经被读线程获取或者写锁已经被其他写线程获取，则写锁获取失败；否则，获取成功并支持重入，增加写状态
        protected final boolean tryAcquire(int acquires) {
            Thread current = Thread.currentThread();
            //获取写锁当前的同步状态
            int c = getState();
            //获取写锁获取的次数
            int w = exclusiveCount(c);
            if (c != 0) {
     //当读锁已被读线程获取或者当前线程不是已经获取写锁的线程的话,当前线程获取写锁失败
                if (w == 0 || current != getExclusiveOwnerThread())
                    return false;
                
                if (w + exclusiveCount(acquires) > MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                //当前线程获取写锁，支持可重复加锁
                setState(c + acquires);
                return true;
            }
            //写锁未被任何线程获取，当前线程可获取写锁
            if (writerShouldBlock() ||
                !compareAndSetState(c, c + acquires))
                return false;
            setExclusiveOwnerThread(current);
            return true;
        }

    	//读锁释放
        protected final boolean tryReleaseShared(int unused) {
            Thread current = Thread.currentThread();
            if (firstReader == current) {
                // assert firstReaderHoldCount > 0;
                if (firstReaderHoldCount == 1)
                    firstReader = null;
                else
                    firstReaderHoldCount--;
            } else {
                HoldCounter rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    rh = readHolds.get();
                int count = rh.count;
                if (count <= 1) {
                    readHolds.remove();
                    if (count <= 0)
                        throw unmatchedUnlockException();
                }
                --rh.count;
            }
            for (;;) {
                int c = getState();
                //读锁释放 将同步状态减去读状态即可
                int nextc = c - SHARED_UNIT;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }

        private IllegalMonitorStateException unmatchedUnlockException() {
            return new IllegalMonitorStateException(
                "attempt to unlock read lock, not locked by current thread");
        }
		
    	//读锁获取:当写锁被其他线程获取后，读锁获取失败
        protected final int tryAcquireShared(int unused) {
            Thread current = Thread.currentThread();
            int c = getState();
            //如果写锁已经被获取并且获取写锁的线程不是当前线程的话,线程获取读锁失败
            if (exclusiveCount(c) != 0 &&
                getExclusiveOwnerThread() != current)
                return -1;
            int r = sharedCount(c);
            
            if (!readerShouldBlock() &&
                r < MAX_COUNT &&
                //当前线程获取读锁
                compareAndSetState(c, c + SHARED_UNIT)) {
                //返回当前获取读锁的次数
                if (r == 0) {
                    firstReader = current;
                    firstReaderHoldCount = 1;
                } else if (firstReader == current) {
                    firstReaderHoldCount++;
                } else {
                    HoldCounter rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current))
                        cachedHoldCounter = rh = readHolds.get();
                    else if (rh.count == 0)
                        readHolds.set(rh);
                    rh.count++;
                }
                return 1;
            }
            //处理在第二步中CAS操作失败的自旋已经实现重入性
            return fullTryAcquireShared(current);
        }

     
        final int fullTryAcquireShared(Thread current) {
            HoldCounter rh = null;
            for (;;) {
                int c = getState();
                if (exclusiveCount(c) != 0) {
                    if (getExclusiveOwnerThread() != current)
                        return -1;
                    // else we hold the exclusive lock; blocking here
                    // would cause deadlock.
                } else if (readerShouldBlock()) {
                    // Make sure we're not acquiring read lock reentrantly
                    if (firstReader == current) {
                        // assert firstReaderHoldCount > 0;
                    } else {
                        if (rh == null) {
                            rh = cachedHoldCounter;
                            if (rh == null || rh.tid != getThreadId(current)) {
                                rh = readHolds.get();
                                if (rh.count == 0)
                                    readHolds.remove();
                            }
                        }
                        if (rh.count == 0)
                            return -1;
                    }
                }
                if (sharedCount(c) == MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                if (compareAndSetState(c, c + SHARED_UNIT)) {
                    if (sharedCount(c) == 0) {
                        firstReader = current;
                        firstReaderHoldCount = 1;
                    } else if (firstReader == current) {
                        firstReaderHoldCount++;
                    } else {
                        if (rh == null)
                            rh = cachedHoldCounter;
                        if (rh == null || rh.tid != getThreadId(current))
                            rh = readHolds.get();
                        else if (rh.count == 0)
                            readHolds.set(rh);
                        rh.count++;
                        cachedHoldCounter = rh; // cache for release
                    }
                    return 1;
                }
            }
        }

        final boolean tryWriteLock() {
            Thread current = Thread.currentThread();
            int c = getState();
            if (c != 0) {
                int w = exclusiveCount(c);
                if (w == 0 || current != getExclusiveOwnerThread())
                    return false;
                if (w == MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
            }
            if (!compareAndSetState(c, c + 1))
                return false;
            setExclusiveOwnerThread(current);
            return true;
        }

        final boolean tryReadLock() {
            Thread current = Thread.currentThread();
            for (;;) {
                int c = getState();
                if (exclusiveCount(c) != 0 &&
                    getExclusiveOwnerThread() != current)
                    return false;
                int r = sharedCount(c);
                if (r == MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                if (compareAndSetState(c, c + SHARED_UNIT)) {
                    if (r == 0) {
                        firstReader = current;
                        firstReaderHoldCount = 1;
                    } else if (firstReader == current) {
                        firstReaderHoldCount++;
                    } else {
                        HoldCounter rh = cachedHoldCounter;
                        if (rh == null || rh.tid != getThreadId(current))
                            cachedHoldCounter = rh = readHolds.get();
                        else if (rh.count == 0)
                            readHolds.set(rh);
                        rh.count++;
                    }
                    return true;
                }
            }
        }

        protected final boolean isHeldExclusively() {
            // While we must in general read state before owner,
            // we don't need to do so to check if current thread is owner
            return getExclusiveOwnerThread() == Thread.currentThread();
        }

        // Methods relayed to outer class

        final ConditionObject newCondition() {
            return new ConditionObject();
        }

        final Thread getOwner() {
            // Must read state before owner to ensure memory consistency
            return ((exclusiveCount(getState()) == 0) ?
                    null :
                    getExclusiveOwnerThread());
        }

        final int getReadLockCount() {
            return sharedCount(getState());
        }

        final boolean isWriteLocked() {
            return exclusiveCount(getState()) != 0;
        }

        final int getWriteHoldCount() {
            return isHeldExclusively() ? exclusiveCount(getState()) : 0;
        }

        final int getReadHoldCount() {
            if (getReadLockCount() == 0)
                return 0;

            Thread current = Thread.currentThread();
            if (firstReader == current)
                return firstReaderHoldCount;

            HoldCounter rh = cachedHoldCounter;
            if (rh != null && rh.tid == getThreadId(current))
                return rh.count;

            int count = readHolds.get().count;
            if (count == 0) readHolds.remove();
            return count;
        }

        /**
         * Reconstitutes the instance from a stream (that is, deserializes it).
         */
        private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
            s.defaultReadObject();
            readHolds = new ThreadLocalHoldCounter();
            setState(0); // reset to unlocked state
        }

        final int getCount() { return getState(); }
    }
```

##### ReadLock

```java
public static class ReadLock implements Lock, java.io.Serializable {
        private static final long serialVersionUID = -5992448646407690164L;
        private final Sync sync;

        protected ReadLock(ReentrantReadWriteLock lock) {
            sync = lock.sync;
        }

       
        public void lock() {
            //获取共享锁
            sync.acquireShared(1);
        }

        public void lockInterruptibly() throws InterruptedException {
            sync.acquireSharedInterruptibly(1);
        }


        public boolean tryLock() {
            return sync.tryReadLock();
        }

        public boolean tryLock(long timeout, TimeUnit unit)
                throws InterruptedException {
            return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
        }

        public void unlock() {
            //释放共享锁
            sync.releaseShared(1);
        }
    }
```



##### WriteLock

### Semaphore

Semaphore也叫信号量,以用来控制同时访问特定资源的线程数量，通过协调各个线程，以保证合理的使用资源。Semaphore是用来保护一个或者多个共享资源的访问，Semaphore内部维护了一个计数器，其值为可以访问的共享资源的个数。一个线程要访问共享资源，先获得信号量，如果信号量的计数器值大于1，意味着有共享资源可以访问，则使其计数器值减去1，再访问共享资源。Semaphore可以用来做流量分流，特别是对公共资源有限的场景，比如数据库连接，可以控制最多只有n个线程能够拿到数据库连接进行操作。Semaphore和ReentrantLock类似，获取许可有公平策略和非公平许可策略，默认情况下使用非公平策略。

```java
public class SemaphoreTest {
    private static final int COUNT = 40;
    private static Executor executor = Executors.newFixedThreadPool(COUNT);
    private static Semaphore semaphore = new Semaphore(10);
    public static void main(String[] args) {
        for (int i=0; i< COUNT; i++) {
            executor.execute(new ThreadTest.Task());
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                //读取文件操作
                semaphore.acquire();
                // 存数据过程
                semaphore.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
            }
        }
    }
}
```

##### Sync

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 1192457210091910933L;

        Sync(int permits) {
            setState(permits);
        }

        final int getPermits() {
            return getState();
        }
    	
    	//非公平获取锁
        final int nonfairTryAcquireShared(int acquires) {
            for (;;) {
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }

    	//锁释放
        protected final boolean tryReleaseShared(int releases) {
            for (;;) {
                int current = getState();
                int next = current + releases;
                if (next < current) // overflow
                    throw new Error("Maximum permit count exceeded");
                if (compareAndSetState(current, next))
                    return true;
            }
        }

        final void reducePermits(int reductions) {
            for (;;) {
                int current = getState();
                int next = current - reductions;
                if (next > current) // underflow
                    throw new Error("Permit count underflow");
                if (compareAndSetState(current, next))
                    return;
            }
        }

        final int drainPermits() {
            for (;;) {
                int current = getState();
                if (current == 0 || compareAndSetState(current, 0))
                    return current;
            }
        }
    }
```

##### NonfairSync

```java
static final class NonfairSync extends Sync {
        private static final long serialVersionUID = -2694183684443567898L;

        NonfairSync(int permits) {
            super(permits);
        }

        protected int tryAcquireShared(int acquires) {
            return nonfairTryAcquireShared(acquires);
        }
    }
```

##### FairSync

```java
static final class FairSync extends Sync {
        private static final long serialVersionUID = 2014338818796000944L;

        FairSync(int permits) {
            super(permits);
        }
    	
    	//公平获取锁
        protected int tryAcquireShared(int acquires) {
            for (;;) {
                //判断阻塞队列是否有等待的线程
                if (hasQueuedPredecessors())
                    return -1;
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
    }
```

### CountDownLatch

它允许一个或多个线程一直等待，直到其他线程执行完后再执行。例如，应用程序的主线程希望在负责启动框架服务的线程已经启动所有框架服务之后执行。CountDownLatch是通过一个计数器来实现的，计数器的初始化值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就相应得减1。当计数器到达0时，表示所有的线程都已完成任务，然后在闭锁上等待的线程就可以恢复执行任务。