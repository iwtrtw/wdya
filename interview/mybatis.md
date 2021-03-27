#### ${}和#{}的区别

+ #{}: 解析为一个 JDBC 预编译语句（prepared statement）的参数标记符,一个 #{ } 被解析为一个参数占位符 
+ $ {}: 仅仅为一个纯碎的 string 替换，在动态 SQL 解析阶段将会进行变量替换。在使用order by 时，就需要使用$

#### MyBatis中xml文件的标签

+ insert：映射插入语句

+ update：映射更新语句

+ delete：映射删除语句

+ select：映射查询语句

+ resultMap：

  ```xml
  <resultMap id="userResultMap" type="User">
      <id property="id" column="user_id" />
      <result property="username" column="user_name"/>
     <result property="password" column="hashed_password"/>
  </resultMap>
  
  <select id="selectUsers" resultMap="userResultMap">
      select user_id, user_name, hashed_password
      from some_table
      where id = #{id}
  </select>
  ```

+ < sql >：定义复用的sql语句片段，在执行的sql语句标签直接引用即可。可以提高编码效率、简化代码和提高可读性。需要配置id标识，表示该sql片段的唯一标识，通过<include refid=" " / >标签引用，refid的值就是< sql>的id属性的值

  ```xml
  <sql id="Base_Column_List">
      id, question, answer 
  </sql>
  
  <select id="selectByPrimaryKey" parameterType="java.lang.Long" resultMap="BaseResultMap">
      select 
      <include refid="Base_Column_List" />
      from java
      where id = #{id,jdbcType=BIGINT}
  </select>
  ```

+ < where >：用来替换sql语句中的where字段，用来简化sql语句中where条件判断的书写的

+ < if > ：条件判断标签，配置属性test=" 条件字符串 "，判断是否满足条件，满足则执行，不满足则跳过

  ```xml
  <select id="selectByParams" parameterType="map" resultType="user">
  　　　　select * from user
      <where>
          <if test="id != null ">id=#{id}</if>
          <if test="name != null and name.length()>0" >and name=#{name}</if>
          <if test="age != null and age.length()>0">and age = #{age}</if>
      </where>
  </select>　
  ```

  如果当id值为空时，此时打印的sql应是：select * from user where name=“xx” and age=“xx”。**where 标记会自动将其后第一个条件的and或者是or给忽略掉**

+ < set > ：替换sql中的set字段，一般用于update。 **set 标记会自动将其后第一个条件后的逗号忽略掉**

  ```xml
  <update>
      update user 
      <set>
          <if test="name != null and name.length()>0">name = #{name},</if>
          <if test="age != null and age .length()>0">age = #{age },</if>
      </set>
      where id = #{id}
  </update>　
  ```

+ < trim >：格式化的标记，可以完成set或者是where标记的功能

  + prefix：前缀
  + prefixoverride：去掉第一个and或者是or

  ```xml
  select * from user 
  <trim prefix="WHERE" prefixoverride="AND |OR">
      <if test="name != null and name.length()>0"> AND name=#{name}</if>
      <if test="age != null and age.length()>0"> AND age=#{age}</if>
  </trim>
  ```

  + suffixoverride：去掉最后一个逗号（也可以是其他的标记，就像是上面前缀中的and一样）
  + suffix：后缀

  ```xml
  update user
  <trim prefix="set" suffixoverride="," suffix=" where id = #{id} ">
      <if test="name != null and name.length()>0">name=#{name} , </if>
      <if test="age!= null and age.length()>0">age=#{age} , </if>
  </trim>
  ```

+ < choose >：按顺序判断其内部when标签中的test条件出否成立，如果有一个成立，则 choose 结束。当 choose 中所有 when 的条件都不满则时，则执行 otherwise 中的sql。类似于Java 的 switch 语句，choose 为 switch，when 为 case，otherwise 则为 default

  ```xml
  <select id="selectByParams" parameterType="map" resultType="user">
      select * from user where 1 = 1
      <choose>
          <when test="id !=null ">
              AND id = #{id}
          </when >
          <when test="username != null and username !=''">
              AND username = #{username}
          </when >
          <when test="age != null and age !=''">
              AND age = #{age}
          </when >
          <otherwise>
          </otherwise>
      </choose>
  </select>　　　
  ```

  