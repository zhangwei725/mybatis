# 高级映射

## 一、概要

​	前面我们讲的基本都是单表查询,但在我们的实际开发中,我们可能会碰到一些比较复杂的查询

## 二、一对一映射关系

### 1.1、概念

> 所谓的一对一查询，就是说我们在查询一个表的数据的时候，需要关联查询其他表的数据

### 1.2、需求

> 查询所有订单信息，关联查询创建订单的用户信息，因为一个订单只能由一个用户创建，所以这是一对一的查询

### 1.3、实现方式

1. 使用resultType实现
2. 使用resultMap实现

### 1.4、实现思路

1. 确定查询的主表： 订单表orders
2. 确定查询的关联表：用户表user
3. 确定之后,那么，关联表采用内链接查询还是外链接？
4. 由于orders表中有一个外键，通过关联查询用户表t只能查出一条信息，所以可以采用内连接

### 1.5、示例代码(resultType实现)

1. 关系模型

   ![](http://opzv089nq.bkt.clouddn.com/17-11-22/53010635.jpg)

2. 对象模型

   ![](http://opzv089nq.bkt.clouddn.com/17-11-22/6089908.jpg)

3. pojo类

   ```java
   public class Orders {
       /**
        * 主键订单Id
        */
       private Integer ordersId;
       /**
        * 下单用户id
        */
       private Integer userId;
       /**
        * 订单号
        */
       private String number;
       /**
        * 创建订单时间
        */
       private Date createTime;
       /**
        * 备注
        */
       private String note;
   }
   ```

   ```java
   public class OrdersCustom extends Orders {
       private String name;
       private String address;
       private Date birthday;
   }
   ```

   **说明**:因为当前方式是采用resultType的方式查询，那么查询出的字段需要和pojo类中字段名一致才能完成输出映射，我们的订单pojo类只有订单信息，字段并不满足sql语句字段查询出来的全部映射，所以需要新增新的pojo类OrdersCustom，它继承于订单类,这里叫做订单的扩展类

4. mapper接口类

   ```java
   public interface OrdersCustomMapper {
       /**
        * 查出订单的用户信息
        * @return
        * @throws Exception
        */
       List<OrdersCustom> findOrderUser() throws Exception;
   }
   ```

5. mapper.xml映射文件

   ```xml
   <mapper namespace="com.werner.sm.mapper.OrdersCustomMapper">
       <select id="findOrderUser" resultType="OrdersCustom">
           SELECT
               t.USER_NAME AS name,
               t.ADDRESS   AS address,
               t.BIRTHDAY  AS birthday,
               o.*
           FROM ORDERS o, T_USER t
           WHERE o.ORDERS_ID=t.USER_ID
       </select>
   </mapper>
   ```

6. 测试代码

   ```java
   @Test
   public void findOrdersUser() throws Exception {
     List<OrdersCustom> ordersUser = shopService.findOrdersUser();
     for (OrdersCustom ordersCustom : ordersUser) {
       System.out.println(ordersCustom.toString());
     }
   }
   ```

### 1.6、示例代码(resultMap实现)

1. pojo

   ```java
   public class Orders {
       /**
        * 主键订单Id
        */
       private Integer ordersId;
       /**
        * 下单用户id
        */
       private Integer userId;
       /**
        * 订单号
        */
       private String number;
       /**
        * 创建订单时间
        */
       private Date createTime;
       /**
        * 备注
        */
       private String note;
       /**
        * 用户信息
        */
       private User user;
   }
   ```

   ```java
   public class User {
       private Integer userId;
       /**
        * 姓名
        */
       private String username;
       /**
        * 性别 1表示男  2 表示女
        */
       private String sex;
       /**
        * 地址
        */
       private String address;
       /**
        * 生日
        */
       private Timestamp birthday;
       /**
        * 创建日期
        */
       private Timestamp createDate;
   }   
       
   ```

2. OrdersMapper

   ```java
   List<Orders> findOrderUser() throws Exception;
   ```

3. OrdersMapper.xml

   ```xml
       <resultMap id="OrdersUserResultMap" type="com.werner.sm.bean.Orders">
           <!-- 配置映射的订单信息 -->
           <!-- 将查询结果中的订单信息映射到Orders对象中，在Orders中添加User属性，将关联查询出来的用户信息映射到orders对象中的user属性中> 
   <!- - id：指定查询列中的唯一标识，订单信息的中的唯 一标识，如果有多个列组成唯一标识，配置多个id column：订单信息的唯一标识列 property：订单信息的唯一标识列所映射到Orders中哪个属性 -->
           <id column="ORDERS_ID" property="ordersId"/>
           <result column="USER_ID" property="userId"/>
           <result column="NUMBER" property="number"/>
           <result column="CREATE_TIME" property="createTime"/>
           <result column="NOTE" property="note"/>

           <!-- 配置映射的关联的用户信息 -->
           <!-- association：用于映射关联查询单个对象的信息 property：要将关联查询的用户信息映射到Orders中哪个属性 -->
           <association property="user" javaType="com.werner.sm.bean.User">
               <!-- id：关联查询用户的唯一标识 column：指定唯一标识用户信息的列 javaType：映射到user的哪个属性 -->
               <id column="USER_ID" property="userId"/>
               <result column="USER_NAME" property="username"/>
               <result column="SEX" property="sex"/>
               <result column="ADDRESS" property="address"/>
           </association>
       </resultMap>
       <select id="findOrderUser" resultMap="OrdersUserResultMap">
           SELECT
               t.*,
               o.*
           FROM ORDERS o, T_USER t
           WHERE o.ORDERS_ID = t.USER_ID
       </select>
   ```

### 1.7、小结

1. 使用resultType实现较为简单，如果pojo中没有包括查询出来的列名，需要增加列名对应的属性，即可完成映射。
2. resultMap需要单独定义resultMap，实现有点麻烦，如果对查询结果有特殊的要求，使用resultMap可以完成将关联查询映射pojo的属性中。
3. resultMap可以实现延迟加载，resultType无法实现延迟加载

## 三、一对多(多对一)映射关系

### 1.1、概念

> 一对多是指一方持有多方的引用。例如:去京东购物，那么一个京东用户可以对应多个购物订单

### 1.2、实现思路

1. 确定查询的主表： 用户表user

2. 确定查询的关联表：订单表orders

3. 主表与关联表采用外键关联

4. 关系模型

   ​

5. 对象模型

   ​

### 1.3、实现步骤

1. 编写pojo

   ```java
   public class User {  
   	private Integer userId;
       /**
        * 姓名
        */
       private String username;
       /**
        * 性别 1表示男  2 表示女
        */
       private String sex;
       /**
        * 地址
        */
       private String address;
       /**
        * 生日
        */
       private Timestamp birthday;
       /**
        * 创建日期
        */
       private Timestamp createDate;
       /**
        * 用户创建的订单列表
        */
       private List<Orders> ordersList;
     }  
   ```

   ```java
   public class Orders {
       /**
        * 主键订单Id
        */
       private Integer ordersId;
       /**
        * 下单用户id
        */
       private Integer userId;
       /**
        * 订单号
        */
       private String number;
       /**
        * 创建订单时间
        */
       private Date createTime;
       /**
        * 备注
        */
       private String note;
   }    
   ```

2. 编写mapper.java接口(采用代理方式开发)

   ```java
       /**
        * 获得所有用户的所有订单订单信息
        * @return
        */
       List<User> getUsersOrders();
       /**
        * 根据用户id查询所有的订单信息
        * @param userId
        * @return
        */
       List<Orders> getOrdersByUserId(int userId);
   ```

3. 编写mapper.xml

   ```xml
   <resultMap id="usersOrdersResultMap"
              type="User">
     <id property="userId" column="user_id"/>
     <result property="username" column="USER_NAME"/>
     <result property="birthday" column="BIRTHDAY"/>
     <result property="sex" column="SEX"/>
     <result property="address" column="ADDRESS"/>
     <result property="createDate" column="CREATE_DATE"/>
     <collection property="ordersList"
                 column="USER_ID"
                 ofType="Orders"
                 select="getOrdersByUserId"/>
   </resultMap>

   <select id="getOrdersByUserId" resultType="Orders">
     SELECT *
     FROM ORDERS
     WHERE USER_ID = #{userId}
   </select>

   <select id="getUsersOrders"
           resultMap="usersOrdersResultMap">
     SELECT *
     FROM T_USER
   </select>
   ```

### 1.4、多对一参考一对一关系,查询的主体是Orders



## 四、多对多映射关系

### 1.1、概念

> 其中一个多方持有另一个多方的集合对象，要实现多对多关系，必须有一张中间表 用于维护两方之间的关系

### 1.2、概念	

### 1.3、单向多对多

#### 1、说明

> 权限和角色的关系是一种多对多的关系，一个权限可以分配给多个角色，一个角色拥有多个权限。

#### 2、对象模型

> ![](http://opzv089nq.bkt.clouddn.com/17-7-11/92223778.jpg)

#### 3、关系模型

> ![](http://opzv089nq.bkt.clouddn.com/17-7-11/58584552.jpg)

#### 4、配置信息

1. privilege类配置

   ```java
   private Integer privilegeId;
   private String pname;
   ```

2. Role类配置

   ```java
   private Set<Privilege> privileges;
   private int roleId;
   private String roleName;
   ```

3. 创建表语句

   ```mysql
   CREATE TABLE TB_ROLE(                        #角色表  
      ROLE_Id INT PRIMARY KEY AUTO_INCREMENT ,  #角色ID  
      ROLE_NAME VARCHAR(20) NOT NULL            #角色名称  
   ) ;  
     
   CREATE TABLE TB_PRIVILEGE (                  #权限表  
      PRIVILEGE_ID INT PRIMARY KEY ,            #权限ID  
      PRIVILEGE_NAME VARCHAR(45) NOT NULL       #权限表  
   ) ;  
     
   CREATE TABLE TB_ROLE_PRIVILEGE(              #角色_权限中间表  
      ROLE_ID INT ,  
      PRIVILEGE_ID INT ,  
      PRIMARY KEY (ROLE_ID,PRIVILEGE_ID) ,  
      CONSTRAINT fk_privilege_role FOREIGN KEY(ROLE_ID) REFERENCES TB_ROLE(ROLE_ID) ,  
      CONSTRAINT fk_role_privilege FOREIGN KEY(PRIVILEGE_ID) REFERENCES TB_PRIVILEGE(PRIVILEGE_ID)   
   ) ;
   ```

### 1.4、双向多对多

#### 1、对象模型

![](http://opzv089nq.bkt.clouddn.com/17-7-11/86234682.jpg)	

#### 2、关系模型

![](http://opzv089nq.bkt.clouddn.com/17-7-10/88342368.jpg)

#### 



​	

















 







