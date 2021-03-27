nginx应用

+ 用作web服务器
+ 用作动静分离服务器
+ 用作反向代理服务器

nginx高性能原因

+ epoll多路复用：非阻塞IO，给客户端设置回调，无需轮询，有事件发生后直接执行回调函数，理论上无连接上限（java nio中select在linux 2.6以上会转为epoll模型 ）

  > select变更轮询查找（1024上限），可能需要轮询许多遍才可以找到变更的线程

+ master worker进程模型：平滑重启与加载配置文件，不会断开与服务端的连接

  + master与worker之间是父子进程关系，master用于管理worker进程，worker用于真正处理客户端连接，worker会去抢占一个锁，获得锁的worker就会处理连接(使用epoll)，往后关于该连接的处理都会交给该worker。当某个worker进程down掉，master可以感知到，会接管该worker的权限，重新new一个worker 

    > sbin/nginx  -c  conf/nginx.conf   
    >
    > 使用nginx重启命令时，master会接管worker的工作权限并new一个新的worker并把工作交给他，然后替换掉原来的worker，所以每次重启worker会变成一个新的进程(进程id是信息)

    worker进程是单线程的（如果线程内部没有阻塞操作的，使用单线程比多线程会高效很多），worker把阻塞的操作交给了epoll，有事件直接调用socket的read、write只是用户空间与内核空间之间的数据拷贝而已
    
    ![nginx-master-worker](.\image\nginx-master-worker.png)

+ 协程机制：将每个请求分给对应的协程，协程是依附线程的内存模型，切换开销更小；一个线程可以有多个协程，协程切换时不需要CPU切换开销，只需内存开销即可，当协程遇到阻塞时，执行权会切换到其他协程，保证整个线程在高速前进 ，当阻塞的协程执行完毕后return返回即可这是epoll就会回调

#### nginx处理阶段

![nginx-处理阶段](.\image\nginx-处理阶段.png)