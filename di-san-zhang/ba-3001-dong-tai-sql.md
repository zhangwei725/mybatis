# 动态SQL

## 一、概要

> 传统的使用JDBC的方法，相信大家在组合复杂的的SQL语句的时候，需要去拼接，稍不注意哪怕少了个空格，都会导致错误。Mybatis的动态SQL功能正是为了解决这种问题， 其通过Mybatis提供的动态SQL标签，可组合成非常灵活的SQL语句，从而提高开发人员的效率，
>
> 总体来说动态SQL标签分为以下几类
>
> 1. if语句(条件判断)
> 2. choose(when,otherwize) 类似java的switch
> 3. trim (对包含的内容加上前缀,或者后缀)
> 4. where(主要是用来简化SQL语句中的where条件,能智能处理AND OR,不用担心多余的导致语句出错)
> 5. set(主要用户更新时设置)
> 6. foreach(循环语句,遍历集合数组)
>
> 常用逻辑判断符：
>
> 1，"!=" : 表示不等于
>
> 2，"="：表示等于。注意是一个等号。
>
> 3，"and" : 逻辑与(小写)
>
> 4，"or" ： 逻辑或(小写)

## 二、if

### 1、说明

> if元素是简单的条件判断逻辑，满足指定条件时追加if元素内的SQL

### 2、语法结构

> ```xml
> <select ... >
>   SQL语句
>   <if test="条件表达式">
>      SQL语句2
>   </if>
> </select>
> ```

### 3、示例代码



## 二、choose(when，otherwise)

### 1、说明

> choose元素的作用相当于JAVA中的Switch语句，基本用法和JSTL中使用一样，和otherwist搭配使用。

### 2、语法结构

> ```xml
> <select..>
> SQL语句1
>  <choose>
>   <when test="条件表达式">
>        SQL语句2
>    </when>
>   <otherwise>
>        SQL语句3
>    </otherwise>
>  </choose>
> </select>
>
> ```

### 3、示例代码

1.  
2.  
3.  
4. ​



## 三、where

### 1、说明

> where元素主要是用于简化查询语句中where部门的条件判断。where元素可以在<where>元素所在的位置输出一个where关键字，而且还可以将后面的条件多余的and或or关键字去掉。

### 2、语法结构

> ```xml
> <select..>
>    select 字段 from 表
>    <where>
>       动态追加条件
>    </where>
> </select>
> ```

### 3、示例代码

1.  
2.  
3.  
4. ​

## 四、set(更新专用)

### 1、说明

> set元素主要是用在更新操作的时候，它的主要功能和where元素相似，主要是在set元素所在位置输出一个set关键字，而且还可以去除内容结尾中无关的逗号。

### 2、语法结构

> ```xml
>     
> <update...>
>     update 表
>     <set>
>        动态追加更新字段
>        <if test="条件判断"></if>
>        <if test="条件判断"></if>
>        <if test="条件判断"></if>
>        ...
>     </set>
> 	...
> </update>
> ```

### 3、示例代码

1.  
2.  
3.  
4. ​

## 五、trim(更新专用)

### 1、说明

> 可以在自己包含的内容前加上某前缀，也可以在其后加上某些后缀，预制对应的属性是prefix和suffix;
> 可以把包含内容的首部某些内容过滤，即忽略，也可以把尾部的某些内容过滤，对应的属性是prxfixOverrides和suffixOverridex;
> 正因为trim有上述功能，所有我们也可以非常简单的利用trim里代替where和set元素的功能

### 2、语法结构

> ```xml
> <trim prefix="WHERE" prefixOverrides="AND|OR">
>  ...
> </trim>
> <!-等价于set元素>
> <trim prefix="SET" prefixOverrides=",">
>  ...
> </trim>
> ```

### 3、示例代码

1.  
2.  
3.  
4. ​

## 六、foreach

### 1、说明

> foreach元素实现了逻辑循环，可以进行一个集合的迭代，主要用在构建in条件中。

### 2、语法结构

> ```xml
> <select ...>
> 	select * from 表 where 字段 in**
> 	<foreach collection="集合" index="索引" item="迭代变量" open="(" separator=","close=")">
> 	</foreach>
> </select>
> ```

### 3、示例代码

1.  
2.  
3.  
4. ​













# 