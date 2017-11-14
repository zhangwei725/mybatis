# Mapper映射文件(一)

## 一、说明

> ​	MyBatis 的真正强大在于它的映射语句，也是Mybatis最核心的内容。由于它的异常强大，映射器的 XML 文件就显得相对简单。如果拿它跟具有相同功能的 JDBC 代码进行对比，你会立即发现省掉了将近 95% 的代码。MyBatis 就是针对 SQL 构建的，并且比普通的方法做的更好
>
> SQL 映射文件有很少的几个顶级元素（按照它们应该被定义的顺序）
>
> - `cache` – 给定命名空间的缓存配置。
> - `cache-ref` – 其他命名空间缓存配置的引用。
> - `resultMap` – 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。
> - `sql` – 可被其他语句引用的可重用语句块。
> - `insert` – 映射插入语句
> - `update` – 映射更新语句
> - `delete` – 映射删除语句

## 二、元素详解

### 1、完整的配置文件

1. xml配置(模板)

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"
           "http://ibatis.apache.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="">
       <cache type="PERPETUAL" eviction="LRU" flushInterval="60000"
              size="512" readOnly="true"/>
       <cache-ref namespace=""/>
       <sql id="">
       </sql>
       <resultMap id="" type="">
       </resultMap>
       <insert id=""
               parameterType=""
               flushCache="true"
               statementType="PREPARED"
               keyProperty=""
               keyColumn=""
               useGeneratedKeys="false"
               timeout="20">
           <selectKey
                   keyProperty="userId"
                   resultType=""
                   order="BEFORE"
                   databaseId=""
                   statementType="PREPARED">
           </selectKey>
       </insert>
       <update
               id=""
               parameterType=""
               flushCache="true"
               statementType="PREPARED"
               timeout="20">
       </update>
       <delete
               id=""
               parameterType=""
               flushCache="true"
               statementType="PREPARED"
               timeout="20">
       </delete>
       <select
               id=""
               parameterType="int"
               parameterMap=""
               resultType=""
               resultMap=""
               flushCache="false"
               useCache="true"
               timeout="10000"
               fetchSize="256"
               statementType="PREPARED"
               resultSetType="FORWARD_ONLY">
       </select>
   </mapper>
   ```

### 2、查询(select)

#### 1、完整的xml配置

> ```
> <select
>      <!--  1. id （必须配置）
>         说明 id是命名空间中的唯一标识符，可被用来代表这条语句。 对应Dao里的方法的名称,大小写敏感
>      -->
>      id="findUserById"
>      <!-- 2. parameterType （可选配置, 默认为mybatis自动选择处理）
>         说明 将要传入语句的参数的完全限定类名或别名， 如果不配置，会自动处理
>         parameterType 主要指定参数类型，可以是基本类型,也可以是复杂类型（如对象） -->
>      parameterType="User"
>      <!-- 3. resultType (resultType 与 resultMap 二选一配置)
>         说明 resultType用以指定返回类型，指定的类型可以是基本类型，可以是java容器，也可以是javabean -->
>      resultType="hashmap"
>      <!-- 4. resultMap (resultType 与 resultMap 二选一配置)
>         说明 resultMap用于引用我们通过 resultMap标签定义的映射类型，这也是mybatis组件高级复杂映射的关键-->
>      resultMap="userResultMap"
>      <!-- 5. flushCache (可选配置)
>          说明 将其设置为 true，表示语句一旦执行，都会导致本地缓存和二级缓存都会被清空，默认值：false -->
>      flushCache="false"
>      <!-- 6. useCache (可选配置)
>          说明 如果为true，结果将在二级缓存中缓存。select语句中默认为true -->
>      useCache="true"
>      <!-- 7. timeout (可选配置) 
>          说明 设置超时，若超时则抛出异常。默认值为 unset（依赖驱动）-->
>      timeout="20"
>      <!-- 8. fetchSize (可选配置) 
>        说明 这是尝试影响驱动程序每次批量返回的结果行数和这个设置值相等。默认值为 unset（依赖驱动)-->
>      fetchSize="100"
>      <!-- 9. statementType (可选配置) 
>          说明 STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED-->
>      statementType="PREPARED"
>      <!-- 10. resultSetType (可选配置) 
>        	 说明 FORWARD_ONLY，SCROLL_SENSITIVE 或 SCROLL_INSENSITIVE 中的一个，默认值为 unset （依赖驱动）-->
>      resultSetType="FORWARD_ONLY
>     <!-- 11. resultOrdered (可选配置)
>     	说明  默认是false ,嵌套查询时使用
>     	如果为 true，就是假设包含了嵌套结果集或是分组了，这样的话当返回一个主结果行的时候，就不会发生有对前面结果集的引用的情况。这就使得在获取嵌套的结果集的时候不至于导致内存不够用
>     -->
>     resultOrdered="true"
>     <!-- 12. resultSets (可选配置)
>     	说明 多结果集时使用。它将列出语句执行后返回的结果集并每个结果集给一个名称，名称是逗号分隔的
>     -->
>     resultSets=""
> </select>
> ```

#### 2、元素说明

> | 属性              | 描述                                       |
> | --------------- | ---------------------------------------- |
> | `id`            | 在命名空间中唯一的标识符，可以被用来引用这条语句。                |
> | `parameterType` | 将会传入这条语句的参数类的完全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过 TypeHandler 推断出具体传入语句的参数，默认值为 unset。 |
> | parameterMap    | 这是引用外部 parameterMap 的已经被废弃的方法。使用内联参数映射和 parameterType 属性。 |
> | `resultType`    | 从这条语句中返回的期望类型的类的完全限定名或别名。注意如果是集合情形，那应该是集合可以包含的类型，而不能是集合本身。使用 resultType 或 resultMap，但不能同时使用。 |
> | `resultMap`     | 外部 resultMap 的命名引用。结果集的映射是 MyBatis 最强大的特性，对其有一个很好的理解的话，许多复杂映射的情形都能迎刃而解。使用 resultMap 或 resultType，但不能同时使用。 |
> | `flushCache`    | 将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：false。 |
> | `useCache`      | 将其设置为 true，将会导致本条语句的结果被二级缓存，默认值：对 select 元素为 true。 |
> | `timeout`       | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）。 |
> | `fetchSize`     | 这是尝试影响驱动程序每次批量返回的结果行数和这个设置值相等。默认值为 unset（依赖驱动）。 |
> | `statementType` | STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
> | `resultSetType` | FORWARD_ONLY，SCROLL_SENSITIVE 或 SCROLL_INSENSITIVE 中的一个，默认值为 unset （依赖驱动）。 |
> | `databaseId`    | 如果配置了 databaseIdProvider，MyBatis 会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略。 |
> | `resultOrdered` | 这个设置仅针对嵌套结果 select 语句适用：如果为 true，就是假设包含了嵌套结果集或是分组了，这样的话当返回一个主结果行的时候，就不会发生有对前面结果集的引用的情况。这就使得在获取嵌套的结果集的时候不至于导致内存不够用。默认值：`false`。 |
> | `resultSets`    | 这个设置仅对多结果集的情况适用，它将列出语句执行后返回的结果集并每个结果集给一个名称，名称是逗号分隔的。 |

#### 3、示例代码

1. 简单查询对象

   ```xml
     
     
   ```

2. ​

### 3、增(insert) 改(update) 删(delete)

#### 1、完整配置(模板)

> ```
> <mapper namespace="com.werner.mb.mapper.UserMapper">
>        <insert id="saveUser"
>                 parameterType=""
>                 flushCache="true"
>                 statementType=""
>                 keyProperty=""
>                 useGeneratedKeys=""
>                 timeout="20"
>         >
>       	INSERT INTO tb_user(NAME) VALUES (#{name});
> 	</insert>
>     <update
>             id="update"
>             parameterType=""
>             flushCache=""
>             statementType=""
>             timeout="">
>     </update>
>     <delete
>             id="delete"
>             parameterType="User"
>             flushCache="true"
>             statementType="PREPARED"
>             timeout="20">
>     </delete>
> </mapper>
> ```

#### 2、参数详细说明

=============== insert、update、delete公共属性=============== 

##### 2.1、id (必要参数)

1. 说明

   对应Dao的里方法的名称,注意区分大小写


1. 其它

   1> 可以将要传入的语句的参数完全限定类名或者别名

   2> 参数的类型可以是 基本类型,也可以是复杂对象(User)

   3> 如果想省略包名或者使用别名需要在mybatis-config.xml中配置

##### 2.2、parameterType (可选参数)

1. 说明

   参数类型，默认的话mybatis会自动处理(包名+类名)

##### 2.3、flushCache (可选参数)

1. 说明

   默认是true

2. 其它

   1> 如果设置true 表示每次语句被调用时,都会清除一级缓存和二级缓存

   2> 只有增删改的操作有效        

##### 2.4、statementType (可选参数)

1. 说明

   默认 PREPARED 表示使用何种jdbc的Statement对象来执行sql,三种可选值

2. 其它

   1> 如果设置true 表示每次语句被调用时,都会清除一级缓存和二级缓存

   2> 只有增删改的操作有效      

##### 2.5、timeout (可选参数)

1. 说明

   默认为unset, 依赖驱动 单秒:秒

2. 其它

   在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）。

=============== insert、update公共属性===============

##### 2.6、keyProperty (可选参数)

1. 说明

   默认值  unset 

2. 其它

   很多时候，在向数据库插入数据时，需要保留插入数据的id，以便进行后续的update操作或者将id存入其他表作为外键。但是，在默认情况下，insert操作返回的是一个int值，并且不是表示主键id，而是表示当前SQL语句影响的行数

##### 2.7、useGeneratedKeys (可选参数)

1. 说明

   默认为false

2. 其它

   MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系数据库管理系统的自动递增字段）

2.8、keyColumn

1. 说明

   （仅对 insert 和 update 有用）通过生成的键值设置表中的列名，这个设置仅在某些数据库（像 PostgreSQL）是必须的，当主键列不是表中的第一列的时候需要设置。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。

#### 3、示例代码

1. 简单的保存操作

   ```xml
   <!-- mapper.xml-->
     <insert id="saveUser" parameterType="com.werner.mb.entity.User">
           INSERT  INTO  TB_USER (NAME)  VALUES (#{name})
     </insert>
   ```

   ```java
   /**
    *测试代码
    */
       @Before
       public void setUp() throws Exception {
           ssf = new SqlSessionFactoryBuilder()
                   .build(this.getClass()
                           .getClassLoader()
                           .getResourceAsStream("mybatis-config.xml"));
       }
   	
       @Test
       public void saveUser() throws Exception {
           SqlSession sqlSession = ssf.openSession();
           UserMapper mapper = sqlSession.getMapper(UserMapper.class);
           User user = new User("测试用户1");
           mapper.saveUser(user);
           System.out.println(user.getUserId());
           sqlSession.commit();
           sqlSession.close();
       } 
   /**
    *有返回值和没返回值的区别在于：
    *有返回值的只是对数据库只读模式访问数据库，对数据库数据不会有任何修改，比如各种方式的查询。
    *无返回值的则会以读写模式访问数据库，会对数据库中的数据进行修改，比如删除，增加。
    */
   ```

2. 更新操作

   ```xml
      <!-- mapper.xml-->
      <update id="update" parameterType="user">
           UPDATE TB_USER t
           SET t.NAME = #{name}
           WHERE t.USER_ID = #{userId}
       </update>
   ```

   ```java
       @Test
       public void update() throws Exception {
           SqlSession sqlSession = ssf.openSession();
           UserMapper mapper = sqlSession.getMapper(UserMapper.class);
           User user = new User(11, "修改用户");
           mapper.update(user);
           System.out.println(user.getUserId());
           sqlSession.commit();
           sqlSession.close();
       }

   ```

3. 删除操作

   ```xml
       <delete id="delete" >
           DELETE FROM TB_USER  WHERE USER_ID=#{userId}
       </delete>
   ```

   ```java
       @Test
       public void delete() throws Exception {
           SqlSession sqlSession = ssf.openSession();
           UserMapper mapper = sqlSession.getMapper(UserMapper.class);
           int count = mapper.delete(1);
           System.out.println(count);
           sqlSession.commit();
           sqlSession.close();
       }
   ```

