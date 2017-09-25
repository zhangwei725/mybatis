# Mybatis配置详解

mybatis主要配置一下几个功能

1. 配置连接数据库的四个元素
2. 注册实体类的权限定性类名的别名
3. 配置MyBatis运行环境，即数据源与事务管理器
4. 注册映射文件

## 一、properties

### 1、概要

> 主要是用来定义配置外在化，比如数据库的连接属性等。这些属性都是可外部配置且可动态替换的，既可以在典型的Java属性文件中配置，亦可以通过properties元素的子元素来传递
>
> 将数据库连接参数单独配置在db.properties中，放在类路径下。这样只需要在config.xml中加载db.properties的属性值。这样在config.xml中就不需要对数据库连接参数硬编码。
> 将数据库连接参数只配置在db.properties中，方便对参数进行统一管理，其它xml可以引用该db.properties

### 2、示例代码

1. 定义外部的db.properties

   ```xml
   driver=com.mysql.jdbc.Driver
   url=jdbc:mysql://localhost:3306/mybatis
   username=root
   password=root

   //然后在mybatis-config.xml引用
   <properties resource="config/db.properties"/>  
   ```

2. 也可直接在properties元素中配置(不推荐)

   ```
   <properties">
      <property name="driver" value="com.mysql.jdbc.Driver"/>  
      <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>  
      <property name="username" value="root"/>  
      <property name="password" value="root"/>  
   </properties>  
   ```

## 二、配置环境（environments）

### 1、概要

> 配置环境主要指的是配置SqlSessionFactory的初始化参数,比如:数据库的事物管理机制还有连接数据的一些必要参数
>
> MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中， 现实情况下有多种理由需要这么做。例如，开发、测试和生产环境需要有不同的配置；或者共享相同 Schema 的多个生产数据库， 想使用相同的 SQL 映射。许多类似的用例。
>
> **注:尽管可以配置多个环境，每个 SqlSessionFactory 实例只能选择其一**

### 2、environment

#### 2.1、说明

> 如果你想连接两个数据库，就需要创建两个 SqlSessionFactory 实例，每个数据库对应一个。而如果是三个数据库，就需要三个实例，依此类推，
>
> **每个数据库环境对应一个 SqlSessionFactory 实例**

#### 2.2、示例代码

1. java代码

   ```java
   //指定配置文件
   SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment);
   SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment,properties);

   //使用默认的配置文件加载
   SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader);
   SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader,properties);
   ```

2. xml配置

   ```
   <environments default="dev">
     <environment id="dev">
     </environment>
     <environment id="test">
     </environment>
     <environment id="product">
     </environment>
   </environments>
   ```

3. 关键点

   > 默认的环境 ID（environments元素的属性 default=”dev”）
   >
   > 每个 environment 元素定义的环境 ID（environment元素的属性id=”dev”, id="product"）

### 3、事务管理器（transactionManager）

#### 3.1、概要

在 MyBatis 中有两种类型的事务管理器 主要是通过type属性将配置

1. type="JDBC" 

   这个配置就是直接使用了 JDBC 的提交和回滚设置，它依赖于从数据源得到的连接来管理事务作用域

2. type="MANAGED"

   这个配置几乎没做什么。它从来不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。 默认情况下它会关闭连接，然而一些容器并不希望这样，因此需要将 closeConnection 属性设置为 false 来阻止它默认的关闭行为

#### 3.2、示例代码

1. 示例一(JDBC)

   ```
   <environments default="dev">
     <environment id="dev">
     <transactionManager type="JDBC">
         <property name="..." value="..."/>
       </transactionManager>
     </environment>
   </environments>
   ```

2. 示例二(MANAGED)

   ```
   <transactionManager type="MANAGED">
     <property name="closeConnection" value="false"/>
   </transactionManager>
   ```

### 4、数据源（dataSource）

#### 4.1、概要

> 配置MyBatis使用的数据源类型与数据库连接基本属性
>
> 1. **UNPOOLED**
> 2. **POOLED**
> 3. **JNDI**

#### 4.2、UNPOOLED

1. 说明

   > UNPOOLED：不使用连接池。即每次请求，都会为其创建一个DB连接，使用完毕后，会马上将此连接关闭

2. 属性介绍

   | 属性                               | 说明                                       |
   | -------------------------------- | ---------------------------------------- |
   | driver                           | 这是 JDBC 驱动的 Java 类的完全限定名（并不是JDBC驱动中可能包含的数据源类 |
   | url                              | 这是数据库的 JDBC URL 地址。                      |
   | username                         | 登录数据库的用户名。                               |
   | password                         | 登录数据库的密码                                 |
   | defaultTransactionIsolationLevel | 默认的连接事务隔离级别                              |

3. 示例代码

   ```
   <environments default="dev">
       <environment id="dev">
           <transactionManager type="JDBC"/>
           <dataSource type="UNPOOLED">
               <property name="driver" value="${driver}"/>
               <property name="url" value="${url}"/>
               <property name="username" value="${username}"/>
               <property name="password" value="${password}"/>
               <defaultTransactionIsolationLevel name>
           </dataSource>
       </environment>
   </environments>
   ```

#### 4.3、POOLED

1. 说明

   > 使用数据库连接池来维护连接

2. 属性介绍

   | 属性                                     | 说明                                       |
   | -------------------------------------- | ---------------------------------------- |
   | poolMaximumActiveConnections           | 在任意时间可以存在的活动（也就是正在使用）连接数量，默认值：10         |
   | poolMaximumIdleConnections             | 任意时间可能存在的空闲连接数                           |
   | poolMaximumCheckoutTime                | 在被强制返回之前，池中连接被检出（checked out）时间，默认值：20000 毫秒（即 20 秒） |
   | poolTimeToWait                         | 这是一个底层设置，如果获取连接花费的相当长的时间，它会给连接池打印状态日志并重新尝试获取一个连接（避免在误配置的情况下一直安静的失败），默认值：20000 毫秒（即 20 秒） |
   | poolMaximumLocalBadConnectionTolerance | 这是一个关于坏连接容忍度的底层设置， 作用于每一个尝试从缓存池获取连接的线程. 如果这个线程获取到的是一个坏的连接，那么这个数据源允许这 个线程尝试重新获取一个新的连接，但是这个重新尝试的次数不应该超过 `poolMaximumIdleConnections` 与 `poolMaximumLocalBadConnectionTolerance` 之和。 默认值：3 |
   | poolPingQuery                          | 发送到数据库的侦测查询，用来检验连接是否处在正常工作秩序中并准备接受请求。默认是“NO PING QUERY SET”，这会导致多数数据库驱动失败时带有一个恰当的错误消息 |
   | poolPingEnabled                        | 是否启用侦测查询。若开启，也必须使用一个可执行的 SQL 语句设置 poolPingQuery 属性（最好是一个非常快的 SQL），默认值：false |
   | poolPingConnectionsNotUsedFor          | 配置 poolPingQuery 的使用频度。这可以被设置成匹配具体的数据库连接超时时间，来避免不必要的侦测，默认值：0（即所有连接每一时刻都被侦测 — 当然仅当 poolPingEnabled 为 true 时适用 |

3. 示例代码

   ```
   <dataSource type="POOLED">
       <property name="driver" value="com.mysql.jdbc.Driver"/>  
       <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>  
       <property name="username" value="root"/>  
       <property name="password" value="root"/>  
   </dataSource>
   ```

   ```
   //通过properties配置
   <dataSource type="POOLED">
       <property name="driver" value="${dirver}"/>  
       <property name="url" value="${url}"/>  
       <property name="username" value="${user}"/>  
       <property name="password" value="${password}"/>  
   </dataSource>
   ```

#### 4.4、JDNI

1. 说明

   > 为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用

2. 属性介绍

   | 属性              | 说明                                       |
   | --------------- | ---------------------------------------- |
   | initial_context | 这个属性用来在 InitialContext 中寻找上下文（即，initialContext.lookup(initial_context)）。这是个可选属性，如果忽略，那么 data_source 属性将会直接从 InitialContext 中寻找。 |
   | data_source     | 这是引用数据源实例位置的上下文的路径。提供了 initial_context 配置时会在其返回的上下文中进行查找，没有提供时则直接在 InitialContext 中查找。 |

3. 示例代码

   1、环境配置

   ```
   <environment id="dev">
   <dataSource type="JNDI">
      <property name="data_source" value="java:comp/env/jndi/mybatis"/>
      </dataSource>
   </environment>
   ```

   2、tomcatconf目录下配置context.xml

   ```
       <Resource name="jndi/mybatis"   <!--这里就是上文自己定义的目录-->
                   auth="Container"   
                   type="javax.sql.DataSource"   
                   driverClassName="com.mysql.jdbc.Driver"   
                   url="jdbc:mysql://localhost:3306/mybatis"   
                   username="root"   
                   password="root"   
                   poolMaximumActiveConnections="20"   
                   poolMaximumIdleConnections="10"   
                   poolMaximumCheckoutTime="10000"/> 
   ```

4.5、第三方数据源

1. 概要

   > Mybaits也可以通过需要实现接口 org.apache.ibatis.datasource.DataSourceFactory来配置第三方的数据源

2. 示例代码

   1、java代码

   ```java
   public class C3P0DataSourceFactory extends UnpooledDataSourceFactory {
     public C3P0DataSourceFactory() {
       this.dataSource = new ComboPooledDataSource();
     }
   }
   ```

   2、环境配置

   ```
   <dataSource type="com.werner.config.C3P0DataSourceFactory">
     <property name="driver" value="com.mysql.jdbc.Driver"/>
     <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
     <property name="username" value="root"/>
     <property name="password" value="root"/>
   </dataSource>
   ```

## 三、映射器(Mapper)

### 1、概要

> 定义SQL映射语句。首先需要告诉MyBatis到哪里去找到这些语句。Java在自动查找这方面没有提供一个很好的方法，所以最佳的方式是告诉MyBatis到哪里去找映射文件。可以使用相对于类路径的资源引用、或完全限定资源定位符（包括file:///的URL），或类名和包名等等

### 2、配置方式

#### 1、'<mapper resource="">'

1. 说明

   resource指向的是相对于类路径下的目录

2. 示例代码

   ```
   <mapper resource="mapper/User.xml" />
   ```

3. 结构图

   ![](http://opzv089nq.bkt.clouddn.com/17-9-25/31157835.jpg)

#### 2、'<mapper class=" " />'

1. 说明

   使用mapper接口类路径

2. 示例代码

   ```
   <mapper class="com.werner.mybatis.mapper.UserMapper"/>
   ```

3. 工程结构

   ![](http://opzv089nq.bkt.clouddn.com/17-9-25/84214133.jpg)


1. 注意
   - 映射文件名要与dao接口名相同
   - 映射文件要与接口在同一包中
   - 映射文件中<mapper/>的namespace属性值为dao接口的全类名

#### 3、'<package name="">'

1. 说明

   注册指定包下的所有mapper接口

2. 示例代码

   ```
   <package name="com.werner.mybatis.mapper"/>
   ```

3. 注意

   - dao使用mapper动态代理实现
   - 映射文件名要与dao接口名相同
   - 映射文件要与接口在同一包中
   - 映射文件中\<mapper/>的namespace属性值为dao接口的全类名

#### 4、'<mapper url="" />'

1. 说明

   使用完全限定路径，可以将映射文件放在本地或网络的任意位置，通过url地址即可直接访问。当通常映射文件是存放在当前应用中的，所以该方式不常用

2. 示例代码

   ```
   <mapper url="file:///D:\workspace\mybatis\config\mapper\User.xml" />
   ```

# 四、TypeAliase（类型别名）

### 1、概要

> 指定实体类权限定性类名的别名,通过这种方式设置别名在以后的操作是如果用到了某个javabean的完全限定名的时候我们就可以使用alias设置的值来代替，从而简化了编程

### 2、配置方式

#### 2.1、xml配置方式

##### 1、\<package/>方式(开发常用)

1. 说明

   一般使用\<package/>方式，这样做的好处是会将该包中年所有实体类的简单类名指定为别名

2. 示例代码

   ```
   <!--配置别名-->
   <typeAliases>
       <package name="com.werner.mybatis.entity"/>
   </typeAliases>
   ```

#### 2.2、通过\<typealias/>指定

1. 说明

   该方式的好处是，可以指定别名为简单类名以外的其他名称。当然，弊端是，必须逐个指定，比较繁琐。

2. 属性

   - type：权限定性类名
   - alias：别名

3. 示例代码

   ```
   <!-- 注册类的别名 -->
   <typeAliases>
       <typeAlias type="com.werner.mybatis.entity.User" alias="User"/>
   </typeAliases>
   ```

### 2.3、注解配置方式

1. 示例代码

   ```
   import org.apache.ibatis.type.Alias;

   @Alias(value = "User")
   public class User {
       private int uid;
       private String name;
   }	
   ```

### 3、内置的类型别名

| 别名         | 映射的类型      |
| ---------- | ---------- |
| _byte      | byte       |
| _long      | long       |
| _short     | short      |
| _int       | int        |
| _integer   | int        |
| _double    | double     |
| _float     | float      |
| _boolean   | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |

