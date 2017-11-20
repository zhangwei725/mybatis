# Mapper映射文件(三)

### 概要

​	MyBatis中在查询进行select映射的时候，返回类型可以用resultType，也可以用resultMap，resultType是直接表示返回类型的，而resultMap则是对外部ResultMap的引用，但是resultType跟resultMap不能同时存在。
在MyBatis进行查询映射时，其实查询出来的每一个属性都是放在一个对应的Map里面的，其中键是属性名，值则是其对应的值。

1. 当提供的返回类型属性是resultType时，MyBatis会将Map里面的键值对取出赋给resultType所指定的对象对应的属性。所以其实MyBatis的每一个查询映射的返回类型都是ResultMap，只是当提供的返回类型属性是resultType的时候，MyBatis对自动的给把对应的值赋给resultType所指定对象的属性。
2. 当提供的返回类型是resultMap时，因为Map不能很好表示领域模型，就需要自己再进一步的把它转化为对应的对象，这常常在复杂查询中很有作用

## 一、resultType标签元素

### 1、概要

​	使用resultType进行输出映射，只有查询出来的列名和pojo中的属性名一致，该列才可以映射成功。如果查询出来的列名和pojo中的属性名全部不一致，没有创建pojo对象。只要查询出来的列名和pojo中的属性有一个一致，就会创建pojo对象

### 2、配置说明

1. mapper.xml(模板)

   ```xml
   <!--resultType="int" 可以省略不写 框架会自行判断 -->
   <select id="countUser" resultType="int">
     SELECT COUNT(*)
     FROM T_USER
   </select>
   ```

### 3、示例代码

#### 3.1、返回基本类型

1. 映射文件

   ```xml
   <!--resultType="int" 可以省略不写 框架会自行判断 -->
   <select id="countUser" resultType="int">
     SELECT COUNT(*)
     FROM T_USER
   </select>
   ```

2. UserMapper.java

   ```java
   int countUser();
   ```

3. 测试代码

   ```java
   @Test
   public void countUser() throws Exception {
     int count = userMapper.countUser();
     System.out.println(count);
   }
   ```

3.2、返回集合套pojo

1. 映射文件

   ```xml
   <!--根据根据注册时间查询用户信息-->
   <select id="findUserByDate" resultType="User">
     SELECT *
     FROM T_USER t
     <where>
       <if test="startDate!=null and startDate!=''">
         <![CDATA[
                   t.CREATE_DATE>=#{startDate}
                 ]]>
       </if>
       <if test="endDate!=null and endDate!=''">
         <![CDATA[
                  AND t.CREATE_DATE<=#{endDate}
               ]]>
       </if>
     </where>
   </select>
   ```

2. UserMapper

   ```java
   List<User> findUserByDate(@Param("startDate") String startDate, @Param("endDate") String endDate);
   ```

3. 测试代码

   ```java
   @Test
   public void findUserByDate() {
     List<User> users = userMapper.findUserByDate("2017-11-20", "2017-11-21");
     System.out.println(users.size());
   }
   ```

#### 3.3、当列名跟pojo属性不一致的情况

1. 关系模型

   ![](http://opzv089nq.bkt.clouddn.com/17-11-20/93941924.jpg)

2. 对象模型

   ![](http://opzv089nq.bkt.clouddn.com/17-11-20/86924989.jpg)

3. 映射文件

   ```xml
   <select id="findUserByName" resultType="User">
     SELECT *
     FROM T_USER t
     WHERE t.USER_NAME = #{name}
   </select>
   ```

4. DAO

   ```java
   @Test
   public void findUserByName() throws Exception {
     User user = userMapper.findUserByName("小红");
     System.out.println(user.toString());
   }
   ```

5. 测试代码

   ```java
   @Test
   public void findUserByName() throws Exception {
     User user = userMapper.findUserByName("小红");
     System.out.println(user.toString());
   }
   /**
    * id的值为null
    * 输出:User{id=null, username='小红', sex='1', address='深圳市宝安区', birthday=2017-11-
    * 01 00:00:00.0, createDate=2017-11-21 16:46:38.0}
    */
   ```

### 4、总结

1. 查询出来的结果集只有一行且一列，可以使用简单类型进行输出映射。（输出简单类型的要求）
2. 查询pojo对象和pojo集合
   不管是输出的pojo单个对象还是一个集合（list中包括pojo），在mapper.xml中resultType指定的类型是一样的。在mapper.java指定的方法返回值类型不一样：
   - 输出单个pojo对象，方法返回值是单个对象类型
   - 输出pojo对象list，方法返回值是List
3. 输出pojo对象可以改用hashmap输出类型，将输出的字段名称作为map的key，value为字段值。如果是集合，那就是list里面套了HashMap
4. 使用resultType进行输出映射，只有查询出来的列名和pojo中的属性名一致，该列才可以映射成功

## 二、ResultMap标签元素

### 1、概要

​	 resultMap 是MyBatis 中最重要最强大的元素了。你可以让你比使用JDBC 调用结果集省掉90%的代码，也可以让你做许多JDBC 不支持的事	

### 2、什么情况下使用

1. 如果查询出来的列名和pojo的属性名不一致，通过定义一个resultMap对列名和pojo属性名之间作一个映射关系
2. 关系映射,比如一对一,一对多(多对一),多对多

### 2、配置说明

1. 完整配置(模板)

   ```
   <!--
         1. type 对应的返回类型，可以是javabean, 也可以是其它
         2. id 必须唯一， 用于标示这个resultMap的唯一性，在使用resultMap的时候，就是通过id引用
         3. extends 继承其他resultMap标签
        -->
   <resultMap id="唯一的标识" type="映射的pojo对象">
     <!--
               1. id 主键，配合select标签的 resultMap属性使用
               2. property 属性对应javabean的属性名
               3. column 对应数据库表的列名
      -->
     <id column="表的主键字段，或者可以为查询语句中的别名字段"
         jdbcType="字段类型"
         property="映射pojo对象的主键属性"/>

     <!--
           result 与id相比，对应普通属性
     -->
     <result column="表的一个字段（可以为任意表的一个字段）"
             jdbcType="字段类型"
             property="映射到pojo对象的一个属性（须为type定义的pojo对象中的一个属性）"/>
     <!--
        constructor 对应javabean中的构造方法
      -->
     <constructor>
       <!-- idArg 对应构造方法中的id参数 -->
       <idArg column="构造方法id"/>
       <!-- arg 对应构造方法中的普通参数 -->
       <arg column="构造方法普通属性"/>
     </constructor>
     <!--
               association 为关联关系，是实现一对一的关键
               1. property 为javabean中容器对应字段名
               2. javaType 指定关联的类型，当使用select属性时，无需指定关联的类型
               3. select 使用另一个select查询封装的结果
               4. column 为数据库中的列名，与select配合使用
           -->
     <association property="pojo的一个对象属性" javaType="pojo关联的pojo对象">
       <!--
                 使用select属性时，无需下面的配置
       -->
       <id column="关联pojo对象对应表的主键字段"
           jdbcType="字段类型"
           property="关联pojo对象的主席属性"/>
       <result column="任意表的字段"
               jdbcType="字段类型"
               property="关联pojo对象的属性"/>
     </association>
     <!--
       有时一个单独的数据库查询也许返回很多不同 (但是希望有些关联) 数据类型的结果集。
       鉴别器元素就是被设计来处理这个情况的, 还有包括类的继承层次结构。
       鉴别器非常容易理 解,因为它的表现很像 Java 语言中的 switch 语句。
       定义鉴别器指定了 column 和 javaType 属性。列是 MyBatis 查找比较值的地方。
       JavaType 是需要被用来保证等价测试的合适类型(尽管字符串在很多情形下都会有用)
     -->
     <discriminator javaType="value的类型">
       <case value="1" resultMap="引用外部resultMap"/>
       <case value="2" resultMap="引用外部resultType"/>
     </discriminator>
     <!--
       collection 为关联关系，是实现一对多的关键
       1. property 为javabean中容器对应字段名
       2. ofType 指定集合中元素的对象类型
       3. select 使用另一个查询封装的结果
       4. column 为数据库中的列名，与select配合使用
      -->
     <!-- 集合中的property须为ofType定义的pojo对象的属性-->
     <collection property="pojo的集合属性"
                 ofType="集合中的pojo对象">
       <!--
                 使用select属性时，无需下面的配置
       -->
       <id column="集合中pojo对象对应的表的主键字段"
           jdbcType="字段类型"
           property="集合中pojo对象的主键属性"/>
       <result column="可以为任意表的字段"
               jdbcType="字段类型"
               property="集合中的pojo对象的属性"/>
     </collection>
   </resultMap>
   ```

### 3、示例代码

#### 前期准备工作

1. 数据库

   ```mysql
   /*
    Navicat Premium Data Transfer

    Source Server         : work
    Source Server Type    : MySQL
    Source Server Version : 50720
    Source Host           : localhost:3306
    Source Schema         : mybatis

    Target Server Type    : MySQL
    Target Server Version : 50720
    File Encoding         : 65001

    Date: 18/11/2017 19:22:07
   */

   SET NAMES utf8mb4;
   SET FOREIGN_KEY_CHECKS = 0;

   -- ----------------------------
   -- Table structure for ITEMS
   -- ----------------------------
   DROP TABLE IF EXISTS `ITEMS`;
   CREATE TABLE `ITEMS` (
     `ITEM_ID` int(11) NOT NULL AUTO_INCREMENT,
     `ITEM_NAME` varchar(32) NOT NULL COMMENT '商品名称',
     `PRICE` decimal(10,2) NOT NULL COMMENT '商品定价',
     `DETAIL` text COMMENT '商品描述',
     `PIC` varchar(64) DEFAULT NULL COMMENT '商品图片',
     `CREATE_TIME` datetime NOT NULL COMMENT '生产日期',
     PRIMARY KEY (`ITEM_ID`)
   ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

   -- ----------------------------
   -- Records of ITEMS
   -- ----------------------------
   BEGIN;
   INSERT INTO `ITEMS` VALUES (1, 'Apple', 10000.00, 'Apple iPhone X (A1865) 64GB 银色 移动联通电信4G手机', NULL, '2017-11-07 13:28:53');
   INSERT INTO `ITEMS` VALUES (2, 'HUAWEI', 6000.00, '华为 HUAWEI Mate 10 Pro 全网通 6GB+64GB 银钻灰 移动联通电信4G手机 双卡双待', NULL, '2017-11-07 13:28:53');
   INSERT INTO `ITEMS` VALUES (3, 'HUAWEI', 3899.00, '华为 HUAWEI Mate10 4GB+64GB 摩卡金 移动联通电信4G手机 双卡双待', NULL, '2017-11-07 13:28:53');
   COMMIT;

   -- ----------------------------
   -- Table structure for ORDERS
   -- ----------------------------
   DROP TABLE IF EXISTS `ORDERS`;
   CREATE TABLE `ORDERS` (
     `ORDERS_ID` int(11) NOT NULL AUTO_INCREMENT,
     `USER_ID` int(11) NOT NULL COMMENT '下单用户id',
     `NUMBER` varchar(30) NOT NULL COMMENT '订单号',
     `CREATETIME` datetime NOT NULL COMMENT '创建订单时间',
     `NOTE` varchar(100) DEFAULT NULL COMMENT '备注',
     PRIMARY KEY (`ORDERS_ID`),
     KEY `FK_orders_1` (`USER_ID`),
     CONSTRAINT `FK_orders_id` FOREIGN KEY (`USER_ID`) REFERENCES `t_user` (`USER_ID`) ON DELETE NO ACTION ON UPDATE NO ACTION
   ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

   -- ----------------------------
   -- Records of ORDERS
   -- ----------------------------
   BEGIN;
   INSERT INTO `ORDERS` VALUES (1, 1, '1000010', '2015-06-04 13:22:35', NULL);
   INSERT INTO `ORDERS` VALUES (2, 1, '1000011', '2015-07-08 13:22:41', NULL);
   INSERT INTO `ORDERS` VALUES (3, 2, '1000012', '2015-07-17 14:13:23', NULL);
   INSERT INTO `ORDERS` VALUES (4, 3, '1000012', '2015-07-16 18:13:23', NULL);
   INSERT INTO `ORDERS` VALUES (5, 4, '1000012', '2015-07-15 19:13:23', NULL);
   INSERT INTO `ORDERS` VALUES (6, 5, '1000012', '2015-07-14 17:13:23', NULL);
   INSERT INTO `ORDERS` VALUES (7, 6, '1000012', '2015-07-13 16:13:23', NULL);
   COMMIT;

   -- ----------------------------
   -- Table structure for ORDER_DETAIL
   -- ----------------------------
   DROP TABLE IF EXISTS `ORDER_DETAIL`;
   CREATE TABLE `ORDER_DETAIL` (
     `ID` int(11) NOT NULL AUTO_INCREMENT,
     `ORDERS_ID` int(11) NOT NULL COMMENT '订单id',
     `ITEM_ID` int(11) NOT NULL COMMENT '商品id',
     `ITEMS_NUM` int(11) DEFAULT NULL COMMENT '商品购买数量',
     PRIMARY KEY (`ID`),
     KEY `FK_ORDERDETAIL_1` (`ORDERS_ID`),
     KEY `FK_ORDERDETAIL_2` (`ITEM_ID`),
     CONSTRAINT `FK_ORDERDETAIL_1` FOREIGN KEY (`ORDERS_ID`) REFERENCES `orders` (`ORDERS_ID`) ON DELETE NO ACTION,
     CONSTRAINT `FK_ORDERDETAIL_2` FOREIGN KEY (`ITEM_ID`) REFERENCES `items` (`ITEM_ID`) ON DELETE NO ACTION
   ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

   -- ----------------------------
   -- Records of ORDER_DETAIL
   -- ----------------------------
   BEGIN;
   INSERT INTO `ORDER_DETAIL` VALUES (10, 1, 1, 1);
   INSERT INTO `ORDER_DETAIL` VALUES (11, 1, 2, 3);
   INSERT INTO `ORDER_DETAIL` VALUES (12, 2, 3, 4);
   INSERT INTO `ORDER_DETAIL` VALUES (13, 3, 2, 3);
   COMMIT;

   -- ----------------------------
   -- Records of T_USER
   -- ----------------------------
   DROP TABLE IF EXISTS `T_USER`;
   CREATE TABLE `T_USER` (
     `USER_ID` int(11) NOT NULL AUTO_INCREMENT,
     `USER_NAME` varchar(32) NOT NULL COMMENT '用户名称',
     `BIRTHDAY` date DEFAULT NULL COMMENT '生日',
     `SEX` char(1) DEFAULT NULL COMMENT '性别',
     `ADDRESS` varchar(256) DEFAULT NULL COMMENT '地址',
     PRIMARY KEY (`USER_ID`)
   ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

   -- ----------------------------
   -- Records of T_USER
   -- ----------------------------
   BEGIN;
   INSERT INTO `T_USER` VALUES (1, '李四', '2014-07-31', '1', '北京市');
   INSERT INTO `T_USER` VALUES (2, '张小明', '2017-11-18', '1', '深圳市宝安区');
   INSERT INTO `T_USER` VALUES (3, '陈小明', '2017-11-18', '1', '深圳市宝安区');
   INSERT INTO `T_USER` VALUES (4, '小青', '2017-11-24', '1', '深圳市宝安区');
   INSERT INTO `T_USER` VALUES (5, '小红', '2017-11-01', '1', '深圳市宝安区');
   INSERT INTO `T_USER` VALUES (6, '老王', '2015-06-27', '2', '北京');
   COMMIT;
   SET FOREIGN_KEY_CHECKS = 1;
   ```

2. Entity对象

   1、**商品表items**：记录了商品信息

   ```java
   public class Items {
     /**
              * 商品表主键Id
              */
     private Integer itemId;
     /**
              * 商品名称
              */
     private String itemsName;
     /**
              * 商品定价
              */
     private BigDecimal price;
     /**
              * 商品描述
              */
     private String detail;
     /**
              * 商品图片
              */
     private String picture;
     /**
              * 生产日期
              */
     private Timestamp createTime;
   }    
   ```

   2、**订单表orders**：记录了用户所创建的订单（购买商品的订单）

   ```java
   public class Orders {
     /**
              * 主键订单Id
              */
     private Integer ordersId;
     /**
     * 下单用户id
     */
     private Integer userid;
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
     // 订单明细
     private List<OrderDetail> orderdetails;
   }
   ```

   3、**订单明细表orderdetail**：记录了订单的详细信息即购买商品的信息

   ```java
   public class OrderDetail {
     /**
      * 主键，订单明细表Id
      */
     private Integer id;
     /**
      * 订单Id
      */
     private Integer ordersId;
     /**
      * 商品id
      */
     private Integer itemId;
     /**
      * 商品购买数量
      */
     private Integer itemsNum;
     /**
      * 明细对应的商品信息
      */
     private Items items;
   }  
   ```

   4、**用户表use**r：记录了购买商品的用户信息

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
        * 用户创建的订单列表
        */
       private List<Orders> ordersList;
   }  
   ```

3. 关系模型

#### 3.1、简单的使用

1. DAO

   ```java
   List<Items> findAllItems(@Param("page") int page, @Param("pageSize") int pageSize);
   ```

2. 映射文件

   ```xml
   <resultMap id="items" type="Items">
     <id property="itemId" column="ITEM_ID"/>
     <result property="createDate" column="CREATE_DATE"  jdbcType="TIMESTAMP"/>
     <result property="price" column="PRICE" jdbcType="NUMERIC"/>
   </resultMap>
   <select id="findAllItems" resultMap="items">
     SELECT *
     FROM ITEMS
     LIMIT #{page}, #{pageSize}
   </select>
   ```

3. 测试代码

   ```java
   @RunWith(SpringJUnit4ClassRunner.class)
   @ContextConfiguration("classpath:spring-context.xml")
   public class ShopMapperTest {
       @Autowired
       ShopService shopService;
       @Test
       public void findAllItems() throws Exception {
         //核心代码
           List<Items> all = shopMapper.findAllItems();
           for (Items items : all) {
               System.out.println(items.toString());
           }
       }
   }
   ```

#### 3.2、通过构造方法

## 二、高级

 







