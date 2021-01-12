---
title: "MySQL & MyBatis"
layout: page
date: 2020-03-08 18:00
---

[TOC]

# MySql & MyBatis

## 什么是sql注入?

SQL注入攻击指的是通过构建特殊的输入作为参数传入Web应用程序，而这些输入大都是SQL语法里的一些组合，通过执行SQL语句进而执行攻击者所要的操作，其主要原因是程序没有细致地过滤用户输入的数据，致使非法数据侵入系统。

一些常见的注入方法

* POST注入：注入字段在 POST 数据中
* Cookie注入：注入字段在 Cookie 数据中
* 延时注入：使用数据库延时特性注入
* 搜索注入：注入处为搜索的地方
* base64注入：注入字符串需要经过 base64 加密

### 如何防御SQL注入？

1. 检查变量数据类型和格式

如果你的SQL语句是类似where id=${id}这种形式，数据库里所有的id都是数字，那么就应该在SQL被执行前，检查确保变量id是int类型；如果是接受邮箱，那就应该检查并严格确保变量一定是邮箱的格式，其他的类型比如日期、时间等也是一个道理。总结起来：只要是有固定格式的变量，在SQL语句执行前，应该严格按照固定格式去检查，确保变量是我们预想的格式，这样很大程度上可以避免SQL注入攻击。
　　
比如，我们前面接受username参数例子中，我们的产品设计应该是在用户注册的一开始，就有一个用户名的规则，比如5-20个字符，只能由大小写字母、数字以及一些安全的符号组成，不包含特殊字符。此时我们应该有一个check_username的函数来进行统一的检查。不过，仍然有很多例外情况并不能应用到这一准则，比如文章发布系统，评论系统等必须要允许用户提交任意字符串的场景，这就需要采用过滤等其他方案了。

2. 过滤特殊符号

对于无法确定固定格式的变量，一定要进行特殊符号过滤或转义处理。

3. 绑定变量，使用预编译语句　　

MySQL的mysql驱动提供了预编译语句的支持，不同的程序语言，都分别有使用预编译语句的方法

实际上，绑定变量使用预编译语句是预防SQL注入的最佳方式，使用预编译的SQL语句语义不会发生改变，在SQL语句中，变量用问号?表示，黑客即使本事再大，也无法改变SQL语句的结构

### 什么是sql预编译?

所谓预编译语句就是将这类语句中的值用占位符替代，可以视为将sql语句模板化或者说参数化，一般称这类语句叫`Prepared Statements`或者Parameterized Statements

预编译语句的优势在于归纳为：一次编译、多次运行，省去了解析优化等过程；此外预编译语句能防止sql注入。

Statement会被sql注入例子：

```java
"select * from tablename where username='"+uesrname+ "'and password='"+password+"'"
```

人工输入加上`or true`

```java
"select * from tablename where username=''or true or'' and password=''"
```

而预编译会变成`select * from tablename where username=? and password=?`，防止了上述问题

## JDBC操作基本流程

1. 加载JDBC驱动程序

`Class.forName("com.mysql.jdbc.Driver"); //反射`

2. 建立连接

`Connection conn = riverManager.getConnection("jdbc:mysql://localhost:3306/数据库名称","用户名称","密码");`

3. 构造sql，往数据库发出请求

```java
String sql = "select id,username,pwd from t_user where id > ?";
PrepareStatement ps = conn.prepareStatement(sql);
ps.setObject(1, 2); //第一个"问号",传入2. -->把id大于2的记录都取出来
rs = ps.executeQuery();
```

4. 返回查询结果

`ResultSet rs=ps.executeQuery();`

5. 关闭`RsultSet`，`PreparedStatement`，`connection`

```java
if(rs!=null){ //RsultSet rs
   rs.close();
}
if(ps!=null){ //PreparedStatement ps
   ps.close();
}
if(conn!=null){ //connection conn
   conn.close();
}
```

## MyBatis `#{}`,`${}`的区别是什么？

* `#{}`是预编译处理(`动态解析 -> 预编译 -> 执行`)，`${}`是字符串替换(`动态解析 -> 编译 -> 执行`)

* MyBatis在处理`#{}`时，会将sql中的`#{}`替换为?号，调用`PreparedStatement`的set方法来赋值；

* MyBatis在处理${}时，就是把${}替换成变量的值。

* 使用`#{}`可以有效的防止SQL注入，提高系统安全性。

eg:

```sql
#{}：select * from t_user where uid=#{uid}

#{}：select * from t_user where uid= ?
```

```sql
${}：select * from t_user where uid= '${uid}'

${}：select * from t_user where uid= '1'
```
