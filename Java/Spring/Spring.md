

### 事务

编程式事务

声明式事务

##### 传播特性

当一个事务方法被另一个事务方法调用时，这个事务方法应该如何进行

+ PROPAGATION_NEVER：当前方法以非事务方式执行(失败不回滚)，当上一级存在事务时则抛出异常
+ PROPAGATION_NOT_SUPPORTED：当前方法以非事务方式执行(失败不回滚)，当上一级存在事务时就将它挂起直到当前方法执行完毕（这可能会造成死锁，被挂起的事务先锁数据然后当前方法需要访问同一数据）



+ PROPAGATION_SUPPORTS：如果上级有事务，当前方法就和上级使用同一事务（只要有一个发生错误回滚，另一个也会回滚）；如果上级没有事务，当前方法就以非事务方式执行。



+ PROPAGATION_REQUIRED_NEW：表示当前方法运行在自己创建的新事务中。如果上级存在当前事务，那么当前方法执行期间，上级事务会被挂起。两者互不影响，完全隔离

+ PROPAGATION_REQUIRED(默认)：如果上级没有事务，当前方法就新建一个事务；如果上级有事务，则加入到上级事务中；两者互相影响，任意一回滚都导致另一个回滚
+ PROPAGATION_NESTED：如果上级有事务，则当前方法的事务会嵌套在上级事务中执行（上级影响当前方法，但当前方法不影响上级）。如果上级没有事务，则当前方法会新建自己的事务
+ PROPAGATION_MANDATORY：如果上级没有事务，则会抛出一个异常；如果上级有事务则使用同一个事务；两者互相影响


