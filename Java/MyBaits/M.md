#### \#{}和${}的区别是什么？

+ ${} 是 Properties ⽂件中的变量占位符，它可以⽤于标签属性值和 sql 内部，属于静态⽂本 替换，⽐如${driver}会被静态替换为 com.mysql.jdbc.Driver 
+ #{} 是 sql 的参数占位符，Mybatis 会将 sql 中的 #{} 替换为?号，在 sql 执⾏前会使⽤ PreparedStatement 的参数设置⽅法，按序给 sql 的?号占位符设置参数值，⽐如 ps.setInt(0, parameterValue)， #{item.name} 的取值⽅式为使⽤反射从参数对象中获取 item 对象的 name 属性值，相当于 param.getItem().getName() 。

#### Xml 映射⽂件中常⻅的标签

select、insert、updae、delete 、resultMap、parameterMap、sql、include、selectKey加上动态 sql 的 9 个标签trim、where、set、foreach、if、choose、when、otherwise、bind 等，其中为 sql ⽚段标签，通过  标签引⼊ sql ⽚段，  为不⽀持⾃增的主键⽣成策略标签

#### 通常⼀个 Xml 映射⽂件，都会写⼀个 Dao 接口与之对应，请 问，这个 Dao 接⼝的⼯作原理是什么？Dao 接⼝⾥的方法，参数不同时，方法能重载吗？

Dao 接⼝，就是⼈们常说的 Mapper 接⼝，接⼝的全限名，就是映射⽂件中的 namespace 的值，接⼝的⽅法名，就是映射⽂件中 MappedStatement 的 id 值，接⼝⽅法内的参数，就是传递给 sql的参数。 Mapper 接⼝是没有实现类的，当调⽤接⼝⽅法时，接⼝全限名+⽅法名拼接字符串作为 key值，可唯⼀定位⼀个 MappedStatement ，举
例： com.mybatis3.mappers.StudentDao.findStudentById ，可以唯⼀找到 namespace为 com.mybatis3.mappers. StudentDao 下⾯ id = findStudentById 的MappedStatement 。在 Mybatis 中，每⼀个 <select> 、 <insert> 、 <update> 、 <delete> 标签，都会被解析为⼀个 MappedStatement 对象。

Dao 接⼝⾥的⽅法，是不能重载的，因为是全限名+⽅法名的保存和寻找策略。Dao 接⼝的⼯作原理是 JDK 动态代理，Mybatis 运⾏时会使⽤ JDK 动态代理为 Dao 接⼝⽣成代理proxy 对象，代理对象 proxy 会拦截接⼝⽅法，转⽽执⾏ MappedStatement 所代表的 sql，然后将sql 执⾏结果返回。

#### Mybatis 是如何进行分⻚的？分⻚插件的原理是什么？

Mybatis 使⽤ RowBounds 对象进⾏分⻚，它是针对 ResultSet 结果集执⾏的内存分⻚，⽽⾮物理分⻚，可以在 sql 内直接书写带有物理分⻚的参数来完成物理分⻚功能，也可以使⽤分⻚插件来完成物理分⻚。
分⻚插件的基本原理是使⽤ Mybatis 提供的插件接⼝，实现⾃定义插件，在插件的拦截⽅法内拦截待
执⾏的 sql，然后重写 sql，根据 dialect ⽅⾔，添加对应的物理分⻚语句和物理分⻚参数。

#### 简述 Mybatis 的插件运⾏原理，以及如何编写⼀个插件

Mybatis 仅可以编写针对ParameterHandler 、ResultSetHandler 、 StatementHandler 、 Executor 这 4 种接⼝的插件，Mybatis 使⽤ JDK 的动态代理，为需要拦截的接⼝⽣成代理对象以实现接⼝⽅法拦截功能，每当执⾏这 4 种接⼝对象的⽅法时，就会进⼊拦截⽅法，具体就是 InvocationHandler 的invoke() ⽅法，当然，只会拦截那些你指定需要拦截的⽅法。实现 Mybatis 的 Interceptor 接⼝并复写 intercept() ⽅法，然后在给插件编写注解，指定要拦截，哪⼀个接⼝的哪些⽅法即可，记住，别忘了在配置⽂件中配置你编写的插件。

#### Mybatis 执⾏批量插⼊，能返回数据库主键列表吗？

能，JDBC 都能，Mybatis 当然也能。

#### Mybatis 动态 sql 是做什么的？都有哪些动态 sql？能简述⼀下动态 sql 的执⾏ 原理不？

Mybatis 动态 sql 可以让我们在 Xml 映射⽂件内，以标签的形式编写动态 sql，完成逻辑判断和动态拼接 sql 的功能，Mybatis 提供了 9 种动态 sql 标签trim、where、set、foreach、if、choose、when、otherwise、bind 。其执⾏原理为，使⽤ OGNL 从 sql 参数对象中计算表达式的值，根据表达式的值动态拼接 sql，以此来完成动态 sql 的功能。

#### Mybatis 是如何将 sql 执⾏结果封装为⽬标对象并返回的？都有哪些映射形式？

第⼀种是使⽤ <resultMap> 标签，逐⼀定义列名和对象属性名之间的映射关系。

第⼆种是使⽤sql 列的别名功能，将列别名书写为对象属性名，⽐如 T_NAME AS NAME，对象属性名⼀般是 name，⼩写，但是列名不区分⼤⼩写，Mybatis 会忽略列名⼤⼩写，智能找到与之对应对象属性名，你甚⾄可以写成 T_NAME AS NaMe，Mybatis ⼀样可以正常⼯作。有了列名与属性名的映射关系后，Mybatis 通过反射创建对象，同时使⽤反射给对象的属性逐⼀赋值并返回，那些找不到映射关系的属性，是⽆法完成赋值的。

#### Mybatis 能执⾏⼀对⼀、⼀对多的关联查询吗？都有哪些实现⽅式，以及它们 之间的区别

能，Mybatis 不仅可以执⾏⼀对⼀、⼀对多的关联查询，还可以执⾏多对⼀，多对多的关联查询，多对⼀查询，其实就是⼀对⼀查询，只需要把 selectOne() 修改为 selectList() 即可；多对多查询，其实就是⼀对多查询，只需要把 selectOne() 修改为 selectList() 即可。

关联对象查询，有两种实现⽅式，⼀种是单独发送⼀个 sql 去查询关联对象，赋给主对象，然后返回主对象。另⼀种是使⽤嵌套查询，嵌套查询的含义为使⽤ join 查询，⼀部分列是 A 对象的属性值，另外⼀部分列是关联对象 B 的属性值，好处是只发⼀个 sql 查询，就可以把主对象和其关联对象查出来。

那么问题来了，join 查询出来 100 条记录，如何确定主对象是 5 个，⽽不是 100 个？其去重复的原理是 <resultMap> 标签内的 <id> ⼦标签，指定了唯⼀确定⼀条记录的 id 列，Mybatis 根据列值来完成 100 条记录的去重复功能， <id> 可以有多个，代表了联合主键的语意。

同样主对象的关联对象，也是根据这个原理去重复的，尽管⼀般情况下，只有主对象会有重复记录，关
联对象⼀般不会重复。

#### Mybatis 是否⽀持延迟加载？如果⽀持，它的实现原理是什么？

Mybatis 仅⽀持 association 关联对象和 collection 关联集合对象的延迟加载，association指的就是⼀对⼀，collection 指的就是⼀对多查询。在 Mybatis 配置⽂件中，可以配置是否启⽤延迟加载 lazyLoadingEnabled=true|false。
它的原理是，使⽤ CGLIB 创建⽬标对象的代理对象，当调⽤⽬标⽅法时，进⼊拦截器⽅法，⽐如调⽤a.getB().getName() ，拦截器 invoke() ⽅法发现 a.getB() 是 null 值，那么就会单独发送事先保存好的查询关联 B 对象的 sql，把 B 查询上来，然后调⽤ a.setB(b)，于是 a 的对象 b 属性就有值了，接着完成 a.getB().getName() ⽅法的调⽤。这就是延迟加载的基本原理。当然了，不光是 Mybatis，⼏乎所有的包括 Hibernate，⽀持延迟加载的原理都是⼀样的。

#### Mybatis 中如何执行批处理？

使⽤ BatchExecutor 完成批处理

 #### Mybatis 都有哪些 Executor 执⾏器？它们之间的区别是什么？

Mybatis 有三种基本的 Executor 执⾏器:SimpleExecutor 、ReuseExecutor、BatchExecutor

+ SimpleExecutor ：每执⾏⼀次 update 或 select，就开启⼀个 Statement 对象，⽤完⽴刻关闭Statement 对象。
+ ReuseExecutor ：执⾏ update 或 select，以 sql 作为 key 查找 Statement 对象，存在就使⽤，不存在就创建，⽤完后，不关闭 Statement 对象，⽽是放置于 Map<String, Statement>内，供下⼀次使⽤。简⾔之，就是重复使⽤ Statement 对象。
+ BatchExecutor ：执⾏ update（没有 select，JDBC 批处理不⽀持 select），将所有 sql 都添加到批处理中（addBatch()），等待统⼀执⾏（executeBatch()），它缓存了多个 Statement 对象，每个 Statement 对象都是 addBatch()完毕后，等待逐⼀执⾏executeBatch()批处理。与 JDBC 批处理相同。

Executor 的这些特点，都严格限制在 SqlSession ⽣命周期范围内

####  Mybatis 中如何指定使⽤哪⼀种 Executor 执⾏器？

在 Mybatis 配置⽂件中，可以指定默认的 ExecutorType 执⾏器类型，也可以⼿动给DefaultSqlSessionFactory 的创建 SqlSession 的⽅法传递 ExecutorType 类型参数。

 #### Mybatis 是否可以映射 Enum 枚举类？

Mybatis 可以映射枚举类，不单可以映射枚举类，Mybatis 可以映射任何对象到表的⼀列上。映射 ⽅式为⾃定义⼀个 TypeHandler ，实现 TypeHandler 的 setParameter() 和 getResult() 接⼝⽅法。 TypeHandler 有两个作⽤，⼀是完成从 javaType ⾄ jdbcType 的转换，⼆是完成 jdbcType ⾄ javaType 的转换，体现为 setParameter() 和 getResult() 两个⽅法，分别代表设 置 sql 问号占位符参数和获取列查询结果。

#### 简述 Mybatis 的 Xml 映射⽂件和 Mybatis 内部数据结构之间的映射关系？

Mybatis 将所有 Xml 配置信息都封装到 All-In-One 重量级对象 Configuration 内部。在 Xml映射⽂件中， <parameterMap> 标签会被解析为 ParameterMap 对象，其每个⼦元素会被解析为ParameterMapping 对象。 <resultMap> 标签会被解析为 ResultMap 对象，其每个⼦元素会被解析为 ResultMapping 对象。每⼀个<select>、<insert>、<update>、<delete> 标签均会被解析为 MappedStatement 对象，标签内的 sql 会被解析为 BoundSql 对象。

#### 说 Mybatis 是半⾃动 ORM 映射⼯具？它与全⾃动的区别在哪⾥？

Hibernate 属于全⾃动 ORM 映射⼯具，使⽤ Hibernate 查询关联对象或者关联集合对象时，可以 根据对象关系模型直接获取，所以它是全⾃动的。⽽ Mybatis 在查询关联对象或关联集合对象时，需 要⼿动编写 sql 来完成，所以，称之为半⾃动 ORM 映射⼯具。