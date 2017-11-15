# Mapper映射文件(三)

## 一、生成主键/获取插入数据主键(selectKey)

### 1、MyBatis的主要作用

1. 对于不支持自动生成类型的数据库或可能不支持自动生成主键 JDBC 驱动来说，可以以使用selectKey生成主键
2. 获取最近插入数据的主键值

### 2、完整配置说明

1. xml配置(模板)

> ```xml
> <selectKey
>   keyProperty=""
>   resultType=""
>   order=""
>   statementType="">
> </selectKey>
> ```

1. 说明

   | 属性            | 描述                                       |
   | ------------- | ---------------------------------------- |
   | keyProperty   | selectKey 语句结果应该被设置的目标属性。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
   | keyColumn     | 匹配属性的返回结果集中的列名称。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
   | resultType    | 结果的类型。MyBatis 通常可以推算出来，但是为了更加确定写上也不会有什么问题。MyBatis 允许任何简单类型用作主键的类型，包括字符串。如果希望作用于多个生成的列，则可以使用一个包含期望属性的 Object 或一个 Map。 |
   | order         | 这可以被设置为 BEFORE 或 AFTER。如果设置为 BEFORE，那么它会首先选择主键，设置 keyProperty 然后执行插入语句。如果设置为 AFTER，那么先执行插入语句，然后是 selectKey 元素 - 这和像 Oracle 的数据库相似，在插入语句内部可能有嵌入索引调用。 |
   | statementType | 与前面相同，MyBatis 支持 STATEMENT，PREPARED 和 CALLABLE 语句的映射类型，分别代表 PreparedStatement 和 CallableStatement 类型。 |

### 3、主键生成

#### 3.1、概念

​	通常主键的生成都是让数据库自动生成(Oracle SqlServer)，主流的数据库一般都支持，当然也有其他不支持的(如oracle)

#### 3.2、不支持自动生成主键

1. oracle创建序列

   ```
   CREATE SEQUENCE SEQ_USER_ID
   INCREMENT BY 1 -- 每次递增1
   START WITH 1 -- 从1开始
   MINVALUE 1 -- 最小值=1
   NOCYCLE; -- 不循环
   ```

2. mapper.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
   <mapper namespace="com.werner.mb.mapper.UserMapper">
     <insert id="update" parameterType="com.werner.mb.entity.User">
       <selectKey keyProperty="userId" resultType="int" order="BEFORE">
         select SEQ_USER_ID.nextval FROM dual
       </selectKey>
       INSERT INTO TB_USER (USER_ID,NAME)
       VALUES (#{userId}, #{name})
     </insert>
   </mapper>
   ```

3. 测试代码

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

### 4、主键获取

#### 4.1、支持主键自动增长

1. mapper.xml

   ```xml
   <!--设置 useGeneratedKeys=”true”，然后再把 keyProperty 设置到目标属性上 -->
   <insert id="saveUser" useGeneratedKeys="true"  keyProperty="userId"  >
     insert into...
   </insert>
   或者
   <insert id="saveUser"  parameterType="com.werner.mb.entity.User" order="AFTER">
     INSERT INTO TB_USER (name) values (#{name})
     <selectKey keyProperty="userId" resultType="java.lang.Integer">
       SELECT LAST_INSERT_ID() as USER_ID
     </selectKey>
   </insert>
   ```

2. java代码

   ```java
   public interface UserMapper {
      void saveUser(User user);
   }
   ```

   ```java
   @Test
   public void saveUser() throws Exception {
     SqlSession sqlSession = ssf.openSession();
     UserMapper mapper = sqlSession.getMapper(UserMapper.class);
     User user = new User("测试用户");
     mapper.saveUser(user);
     System.out.println(user.getUserId());
     sqlSession.commit();
     sqlSession.close();
   }
   ```

### 5、批量处理返回主键

#### 5.1、概要

1. Mybatis 全局设置批处理
2. Mybatis 局部设置批处理
3. Mybatis foreach批量插入

#### 5.2、批量更新返回错误的解决方案

1. 升级Mybatis版本到3.3.1。官方在这个版本中加入了批量新增返回主键id的功能
2. 在Dao中不能使用@param注解。
3. Mapper.xml中使用list变量（parameterType="java.util.List"）接受Dao中的参数集合。

#### 5.3、案例分析

##### 1、mysql

1. mapper.xml

   ```xml
   <!-- 批量新增 -->  
   <insert id="batchInsert" parameterType="java.util.List" useGeneratedKeys="true" keyProperty="userId" >  
     INSERT INTO  
     TB_USER (NAME) 
     VALUES  
     <!--
       1.如果传入的是单参数且参数类型是一个List的时候，collection属性值为list
       2.如果传入的是单参数且参数类型是一个array数组的时候，collection的属性值为array
       3.如果传入的参数是多个的时候，我们就需要把它们封装成一个Map了，当然单参数也可以封装成map
   --> 
     <foreach collection="list"  item="user" separator=",">  
       (#{user.name})  
     </foreach>  
   </insert>  
   ```

2. dao

   ```java
   public interface UserMapper {
     int  batchInsert(List<User> list);
   }
   ```

3. 测试代码

   ```java
   @Test
   public void batchInsert() throws Exception {
     SqlSession sqlSession = ssf.openSession();
     UserMapper mapper = sqlSession.getMapper(UserMapper.class);
     List<User> users = new ArrayList<>();
     for (int i = 0; i < 10; i++) {
       users.add(new User("测试批量"));
     }
     int i = mapper.batchInsert(users);
     System.out.println("行数:" + i);
     for (User user : users) {
       System.out.println(user.getUserId());
     }
     sqlSession.commit();
     sqlSession.close();
   }
   ```

##### 2、oracle

1. mapper.xml

   ```xml
    <!-- insert into table(...) (select ... from dual) union all (select ... from dual) -->
    <!-- useGeneratedKeys="false" 指定为false -->
   <insert id="addList" parameterType="java.util.List" useGeneratedKeys="false">
           INSERT INTO T_APPLAUD
           (USER_ID,NAME)   
           <foreach item="user" index="index" collection="list" separator="union all">
           (
               SELECT  #{user.userId }   FROM DUAL
           )
           </foreach>
       </insert>
   ```

2. dao

   ```
   public interface UserMapper {
     int  batchInsert(List<User> list);
   }
   ```

3. 测试代码

   ```
   @Test
   public void batchInsert() throws Exception {
     SqlSession sqlSession = ssf.openSession();
     UserMapper mapper = sqlSession.getMapper(UserMapper.class);
     List<User> users = new ArrayList<>();
     for (int i = 0; i < 10; i++) {
       users.add(new User("测试批量"));
     }
     int i = mapper.batchInsert(users);
     System.out.println("行数:" + i);
     for (User user : users) {
       System.out.println(user.getUserId());
     }
     sqlSession.commit();
     sqlSession.close();
   }
   ```

### 5、小结

1. 当数据插入操作不关心插入后数据的主键（唯一标识），那么建议使用 *不返回自增主键值* 的方式来配置插入语句，这样可以**避免额外的SQL开销**.
2. 当执行插入操作后需要立即获取插入的自增主键值，比如一次操作中保存一对多这种关系的数据，那么就要使用 *插入后获取自增主键值* 的方式配置.

###  

