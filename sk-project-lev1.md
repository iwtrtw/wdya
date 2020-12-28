 

### DEMO

+ 数据库实体封装，隐藏数据真实性：先设计业务模型，然后根据业务模型分离出数据模型并设计数据库，在业务处理过程中使用View Object封装业务数据供前端展示

  + 数据模型(Data Object-M)：数据层，与数据库映射，通过ORM将数据库中的数据转换为Java Object；关注数据的CURD效率
  + 领域模型(Domain Model-M)：业务核心模型，拥有完成的生命周期(创建、查询、修改、删除)；关注核心业务处理
  + 展示聚合模型(View Object-C)：与前端对接模型，隐藏内部细节
  
+ 数据类型
  + 主业务数据：master data
  + 操作型数据：log data（记录状态，可以用于异步事件完成情况的判断）

+ 分库分表：根据id尾号分散到不同的数据库中

+ 统一返回结果：通用返回结果

+ 异常处理：全局异常处理，处理一切异常，返回给前端错误信息码，而不是springMVC自带错误页面，

+ 前端对输入数据进行校验可以将错误信息及时反馈给用户，增强用户体验；而后端对数据进行校验是一定的，因为数据是要进数据库的

+ 一般来说，密码是和主表分开的，是由另一个系统来实现的

+ spring @transaction实现原理

+ 日志不能打印敏感信息

+ **@Transactional**失效

+ 数据库表操作时尽量少使用null字段，设计表时尽量设为NOT NULL，使用-1或空字符串作为默认值，多数情况下null字段对于前端展示是没有意义的，而java中对空指针的处理是十分脆弱的

  而在使用唯一索引时，则不要设为NOT NULL，若对应的字段在某些情况下是null时，这是不影响使用索引的，但如果字段设置了NOT NULL并具有默认值，这样就违反了唯一性，无法使用唯一索引

  ### Q
  
  Contorller中的通过@Autowrite注入service是单例的吗？
  
  mysql date、datetime、time的区别？
  
  mysql 删除表数据的方法与区别？
  
  解决跨域与session共享问题
  
  ControllerAdvice的使用
  
  mysql远程调用端口开放规则
  
  scp 可用规则
  
  mysql锁机制？在创建订单时面对大量的并发请求，多次与mysql交互是否会产生死锁



pstree -p 5240 | wc -l 查看某个进程包含的线程数量

top -H 实时查看cpu运行情况

netstat -lnpt |grep 5672  检查端口被哪个进程占用

firewall-cmd --zone=public --add-port=5672/tcp --permanent   # 开放5672端口

firewall-cmd --zone=public --remove-port=5672/tcp --permanent  #关闭5672端口

firewall-cmd --reload   # 配置立即生效

firewall-cmd --zone=public --list-ports 查看防火墙所有开放的端口

firewall-cmd --state  查看防火墙状态

systemctl stop firewalld.service  

firewall-cmd --reload   # 配置立即生效

scp -r  /root/sk  root@192.168.232.130:/root   将sk目录复制到192.168.232.130的root目录下

telent 192.168.232.130 3306 测试是否可以连接到192.168.232.130

tail -f  access.log  动态查考access.log的尾部信息

### 瓶颈

##### 单Web容量上限(Spring Boot内置Tomcat)

内置Tomcat默认连接

+ 线程数量：4核8G内存单进程调度线程数800-1000以上之后，就会在CPU调度上耗费大量时间
+ 等待队列长度：队列用作缓冲池，但不能无限长，会消耗内存，出队入队也会消耗内存

##### MySql数据库QPS容量问题

+ 主键查询：千万级别数据（1-10毫秒）
+ 唯一索引查询：千万级别数据（10-100毫秒）
+ 非唯一索引查询：千万级别数据（100-1000毫秒）
+ 无索引：百万数据（1000毫秒+）
+ 非插入更新删除操作：与查询查不多
+ 插入操作：1w~10w tps（依赖配置优化）

##### TPS、QPS

+ PV（Page View）：页面访问量，每次用户访问或者刷新页面都会被计算在内。

+ QPS（Query Per Second）：每秒查询数，每秒系统能够处理的查询请求次数。

+ TPS（Transactions Per Second）：每秒事务数，每秒系统能够处理的事务次数。

  TPS与QPS的区别的是：事务表示客户端发起请求到收到服务端最终响应的整个过程，这是一个TPS而在这个TPS中，为了处理第一次请求可能会引发后续多次对服务端的访问才能完成这次工作，每次访问都算一个QPS。所以，一个TPS可能包含多个QPS

### 优化

#### 分布式扩展

单机容量问题：CPU使用率增加，memory增加，网络带宽使用增加

> cpu us：用户空间的cpu使用情况(用户进程)
>
> cpu sy：内核空间的cpu使用情况(系统调用)
>
> load average：1、5、15分钟load平均值，0代表通常，1代表cup被占用满，1+代表cpu等待阻塞
>
> memory:free 空闲内存  used使用内存

######  数据与服务分离

mysql部署开放远程连接；服务进行水平扩展，对称部署；引入Nginx用作负载均衡（即需要四台服务器Nginx、Mysql、Server1、Server2）

![水平拓展](image\水平拓展.png)

+ mysql开发远程端口：在mysql数据库的mysql库user表中存放着用户信息

> use mysql;
> select host,user,password from user;
> grant all privileges on \*.\* to root@'%'  identified by 'root'   //授权所有数据库的所有表的访问权限给密码为root的root用户
> flush privileges;  //刷新权限

+ nginx：将静态页面放到html/resources目录下并修改配置文件中path

```java
//配置静态资源路径 alias 路径替换
server{
    listen 80;
    server_name localhost;
    ...
    location /resources/ {
        alias /usr/local/nginx/html/resources/;
        index index.html index.html;
    }
    ....
}
```

```java
//配置动态资源：反向代理到后端服务器集群并使用http 1.1开启keepalive
upstream backend_server{
    server 192.168.232.128:8090 weight=1;
    server 192.168.232.129:8090 weight=1;
    keepalive 30;  //设置长连接(秒)
}
server{
    listen 80;
    server_name localhost;
    ...
    location / {
        proxy_pass http://backend_server;
        proxy_set_header Host $http_host:$proxy_port;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1; //使用长连接
        proxy_set_header Connection "";
    }
    ....
}
```

+ 修改springboot内置tomcat默认配置并开启tomcat访问日志

```xml-dtd
# 开启tomcat访问日志
server.tomcat.accesslog.enabled=true
#server.tomcat.accesslog.directory=
#host - user 请求时间 请求行(请求方式、url、版本) 状态码 response大小(字节) 处理请求时间（毫秒）
server.tomcat.accesslog.pattern=%h %l %u %t "%r" %s %b %D
```

###### 分布式会话

+ 基于spring-session-data-redis实现会话共享，将seesion信息存入redis中共

+ 基于唯一token实现会话共享，将token存入redis中，客户端每次请求都带上token

  用户登录后生成token返回给客户端，服务端将token存入redis中，每次客户端请求必须带上token，服务端会判断token是否有效，有效才放行，否则通知客户端要先登录

#### 多级缓存

###### 缓存设计

+ 快速存取，使用内存
+ 将缓存推到离用户最近的地方
+ 缓存清理与同步，源数据变更需要同步到缓存中

###### 多级缓存

+ redis缓存：第一次从数据库查询数据时将结果存放到redis中，以后的查询先从redis中获取，如果没有则从数据库中查询，并将查询结果放入redis中
+ 本地缓存(程序内部)：使用缓存Map——**Guava cache** （对更新频繁地数据不适用而且受限于JVM内存大小）
+ openResty支持nginx先访问只读salve Redis，如果redis中没有再访问服务端（而服务端又可以将数据缓存到master Redis中，有master负责将数据同步的slave Redis中）

#### 页面静态化

###### CDN

静态资源CDN化：将静态资源缓存到CDN中，如果CDN中没有才回源静态资源服务器上获取数据

> css、js、img等元素使用摘要(只要文件不变，摘要就不变 )做文件名部署
>
> html文件可以设置no-cache或较短max age，以便于更新

动态资源CDN化：将ajax动态请求返回的数据静态化成json资源推送到CDN上，通过版本控制异步请求后端更新json数据（但会造成前端展示数据不同步问题）

全页面静态化：在服务端完成html、css，甚至js的load渲染成纯html文件后直接以静态资源的方式部署到CDN上

+ HTTP cache-control响应头

  + private：客户端可以缓存
  + public：客户端和代理服务器(客户端与服务端之间可能通过多个中间服务器)都可以缓存
  + max-age=xxx：缓存的内容将在xxx秒后失效，迫使客户端重新向服务端发起新请求
  + no-cache：使用缓存前强制向服务端再验证一次

  > ETag：资源唯一标识  （服务端返回给浏览器，客户端使用缓存前先想服务端发送带ETag的http请求，若资源没有改变，服务端会返回304 NOT MODIFIED，若资源改变，服务端会返回200和最新的资源）
  >
  > If-None-Match：客户端发送ETag标识符
  >
  > Last-modified：服务端返回资源最后被修改的时间
  >
  > If-Modified-Since：客户端发送匹配资源最后修改时间的标识符

  + no-store：客户端不可以缓存返回的任何内容（一般动态请求使用这个）

+ 服务设置cache-control

  ![cache-control](image\cache-control.png)

+ 客户端请求资源

![客户端请求资源](image\客户端请求资源.png)

+ 浏览器刷新方式

  > 缓存协商：比较ETag和Last-modified到服务端，若服务端判断没有变化则返回304，否则返回200带上最新数据

  + 回车或a链接跳转：先查看cache-control对应的max-age判断cache是否有效，有效则使用本地缓存；若cache-control为no-cache则一定要进入缓存协商(即需要向服务端发起请求判断ETag、Last-Modified)
  + F5刷新：去掉cache-conrol中的max-age或设置max-age为0，然后进入缓存协商
  + 强制刷新(Ctrl+F5)：去掉cache-control和协商头(不带ETag、Last-Modified)，强制重新向服务端请求

简单来说，服务端一般会返回ETag、Last-Modified、Cache-control等内容

> 当服务端返回cache-control中为no store时，客户端不缓存任何内容，下次重新请求

> 当服务端返回cache-control为no cache时，客户端缓存内容，但下次请求时需要先询问服务端(带上if-none-match/if-modified-since)

> 当服务端cache-control没有no cache/no store并且有max-age，若在max-age的时间内，客户端无需访问服务端，可以直接使用本地缓存；若过期了，则需重新向服务端发起请求(带上之前的Etag)

> 当客户端不想直接使用本地缓存时，想向服务器确认一下时，可以设置请求头中cache-control:max-age=0，并带上if-none-match/If-Modified-Since，发起请求向服务端确认 （F5）

> 当客户端不想直接使用本地缓存时，也不想向服务器确认时，只想要最新的内容时，可以设置请求头中cache-control:no cache，并且不带上if-none-match/If-Modified-Since，发起请求向服务端获取最新数据 （Ctrl+F5）

Http缓存控制 https://zhuanlan.zhihu.com/p/55623075

浅谈http中的Cache-Control  https://blog.csdn.net/u012375924/article/details/82806617

HTTP总结（四）：Cache-Control https://zhuanlan.zhihu.com/p/57100556

#### 缓存热点数据

订单创建流程(8次与mysql交互)：校验用户（查询用户(**user_info**)、查询密码(**user_password**)）、校验商品（查询商品(**item**)、查询库存(**item_stock**)、查询活动信息(**promo**)）、下单减库存（更新库存(**item_stock**)——行锁）、订单入库（创建订单(**order_info**)——）、更改商品销量（更新销量(**item**)——行锁）

![交易流程](image\交易流程.png)

+ 用户校验——使用缓存：用户风控策略缓存模型化（判断用户是否存在、账号是否异常、最近异地登录、最近修改密码）
+ 活动校验——使用缓存：引入活动发布流程，模型缓存化，紧急下线（删除redis缓存）
+ 库存扣减（行锁前提是item_id需要加索引，否则会锁表）：库存缓存化（设置定时任务在活动快要开始前将库存缓存到redis中）、库存异步同步到数据、库存数据库最终一致性保证



RocketMQ

![RocketMQ](image\RocketMQ.png)

> unzip rocketmq-all-4.7.1-source-release.zip

```
使用vim xxx.sh命令进行JVM参数配置修改：
将runbroker.sh的第一个JAVA_OPT改成JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m"
将runserver.sh的第一个JAVA_OPT改成JAVA_OPT="${JAVA_OPT} -server -Xms128m -Xmx256m -Xmn256m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```

> nohup sh bin/mqnamesrv &   //启动nameserver(默认9876端口)
>
> netstat -anp | grep 9876
>
> tail -f ~/logs/rocketmqlogs/namesrv.log    //查看namesrv日志
>
> nohup sh bin/mqbroker -n localhost:9876 &  //启动broker
>
> tail -f ~/logs/rocketmqlogs/broker.log   //查看broker日志

> export NAMESRV_ADDR=localhost:9876
>
> //启动producer
>
> sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer  
>
> //启动consumer
>
> sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer

> sh bin/mqshutdown broker  //停止broker
>
> sh bin/mqshutdown namesrv  //停止nameServer

> //新建topic——stock
>
> ./mqadmin updateTopic -n localhost:9876 -t stock -c DefaultCluster

+ 下单减库存，事先将活动库存缓存到redis中，然后直接对redis中的库存进行修改，然后通过发送异步同步数据库的消息

  + 分布式事务问题：如何保证redis事务与其他数据库的事务一致，若redis库存扣减成功，数据也同步了，但在后续订单入库等操作失败事务回滚咋办（**Spring  @Transactional标签只有在方法返回时才会commit**）
  + 数据一致性问题：redis库存扣除成功，但数据同步消息发送失败怎么办

  ```java
  //方法一：利用spring提供的Transaction：确保最近事务在提交后才发送数据同步消息（前提是需要保证消息发送一定会成功）
  @Transactional
  public OrderVo createOrder(Integer userId, Integer itemId, Integer amount,Integer promoId) throws BusinessException {
  	TransactionSynchronizationManager.registerSynchronization(new TransactionSynchronizationAdapter() {
  	@Override
  	public void afterCommit() {
  		boolean mqResult = itemStockService.asyncDecreaseStock(itemId, amount);
  		}
      });
  }
  
  
  ```

  ```java
  //方法二：使用rocketmq的事务型消息确保下单流程走完后消息也被消费掉（引入库存流水记录订单的状态，虽然需要多一个关于订单的数据库操作，但这个和从数据库直接减库存是不同，减库存是多笔交易针对itemid的行锁，对数据库会造成锁争夺、等待的压力，等待而库存流水这是每单交易自己的行锁）
  
  //下单前初始化库存流水
  
  //发送事务型消息——异步同步库存到数据库并监听下面的下单操作
  
  //执行下单操作，下单操作成功时更新库存流水状态为成功
  
  //根据下单操作的成功与否、订单状态决定消息是否被消费
  ```
  

#### 事务型消息

将商品数据修改放到redis中，会出现数据库数据不同步问题，而且无法回滚，需要保证数据的最终一致性

```java
@Transactional(rollbackFor=Exception.class)
public OrderVo asynCreateOrder(Integer userId, Integer itemId, Integer amount, Integer promoId, String stockLogId) throws BusinessException {
        //校验下单状态：用户是否合法？下单的商品是否存在？库存是否剩余？
        //校验用户是否合法(缓存)

        System.out.println("校验用户是否合法");
        UserModel user = userInfoService.getUserByIdInCache(userId);
        if(user==null){
            throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,"用户信息不存在");
        }


        //校验商品是否存在(缓存)
        System.out.println("校验商品是否存在");
        ItemVo itemVo = itemService.getItemByIdInCache(itemId);
        if(itemVo==null){
            throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,"商品信息不存在");
        }

        //校验数量是否合法
        System.out.println("校验购买商品数量是否合法");
        if(amount<=0 ||amount>99){
            throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,"购买数量信息不存在");
        }

        //校验活动信息
        System.out.println("校验是否为促销活动");
        if(promoId!=null){
            if(promoId.intValue()!=itemVo.getPromoId()){
                throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,"活动信息不正确");
            }else if(itemVo.getPromoStatus().intValue()!=2){
                throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,"活动还未开始");

            }
        }

        //落单减库存(此处修改的是活动发布时的缓存，并不是item详情页的缓存)
        System.out.println("落单减库存，扣除redis中库存");
        boolean flag = itemStockService.decreaseStockInCache(itemId, amount);
        if(!flag){
            throw new BusinessException(EmBusinessError.STOCK_NOT_ENOUGH);
        }

        //订单入库
        System.out.println("订单入库：创建订单写入数据库");
        OrderInfo orderInfo = new OrderInfo();
        orderInfo.setUserId(userId);
        orderInfo.setItemId(itemId);
        orderInfo.setAmount(amount);
        orderInfo.setPromoId(promoId);
        if(promoId!=null){
            orderInfo.setItemPrice(itemVo.getPromoPrice());
        }else{
            orderInfo.setItemPrice(itemVo.getPrice());

        }
        orderInfo.setOrderPrice(orderInfo.getItemPrice().multiply(new BigDecimal(amount)));
        orderInfo.setId(generateOrderId());
        orderInfoMapper.insert(orderInfo);

        //商品销量增加
        System.out.println("增加商品的销量");
        itemService.increaseSales(itemId, amount);

        //设置库存流水状态为成功
        System.out.println("设置库存流水状态为成功");
        stockLogService.updateStockLogStatus(stockLogId,2);


        OrderVo orderVo = new OrderVo();
        BeanUtils.copyProperties(orderInfo,orderVo);

        return orderVo;
    }
```

+ 问题

  + redis不可用：直接访问数据库（流量高时数据库抗不住，而异步消息是否已经发送完成）
  + 扣除流水出错

  在分布式高可用场景很难保证所有状态都正常，由于最终一致性问题（是否需要让用户等待），业务场景决定高可用（需要考虑极端情况下）

  + 宁可少买，不能超卖：redis可以比实际数据库中少、超时释放

  宁可超卖，不能少买

+ 下单前直接先添加库存流水：由于库存数量有限，在真正查看库存前就会生成大量库存流水，而后面库存售罄，导致可能到后面许多库存流水是无效的

  添加库存售罄标识，先判断库存是否售罄，如果售罄直接返回并通知各系统售罄（告诉其他系统缓存的内容失效），后面什么操作都不做；如果有回补上新，则需要把售罄标识去除

#### 流量削峰

秒杀下单接口会被脚本刷；秒杀验证逻辑和秒杀下单接口强关联，耦合度高

第一秒大流量涌入，将第一秒的流量高峰削平

###### 秒杀令牌

秒杀接口需要依靠令牌才能进入，令牌是有秒杀活动模块负责生成的（与交易系统无关），秒杀活动模块对秒杀令牌只做校验；令牌的生成、消亡有活动模块全权负责

###### 秒杀大闸

根据商品初始库存颁发对应数量令牌（但在多库存情况下，令牌限制后的剩下流量还是很大）；将用户验证、库存售罄前置到令牌发放前

###### 队列泄洪

排队有时比并发更高效,并发高时锁竞争是十分消耗的CPU上下文切换的（例如redis单线程 innodb mutex key）；依靠排队去限制并发流量，依靠排队和下游拥塞窗口程度调整队列释放流量的大小（一次从队列取多个，即拥塞窗口大小）

+ 将队列维护在本地（使用线程池）
+ 分布式：将队列设置到redis中

使用分布式队列，当出现问题时降级到本地队列

#### 防刷限流

###### 验证码

错峰用户的请求流量，是用户的请求分散

###### 限流

+ 限并发：在controller层面入口处添加计数器，限制访问数量，出口处把计数器重置 （不推荐）
+ 令牌桶算法：限TPS，每次只允许一定数量的请求真正执行业务操作，
+ 漏桶算法





当消息队列宕机如何降级，或者某一环节宕掉，如何降级，保证系统还是可运行的呢？