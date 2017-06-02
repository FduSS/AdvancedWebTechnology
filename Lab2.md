# 高级Web技术 Lab 2：Spring Boot 与 MyBatis

2017.6

[TOC]

# Part 1: Spring Boot

前后端分离的 Web 架构中，后端一般是一个提供 Restful 接口的服务器，为前端提供所需数据。

目前主流的 Restful 后端的语言与框架有：

- JavaScript，运行于 Node.js 环境，有 Express，Restify 等框架
- Java，有 Spring Boot 等框架
- Python，有 web.py 等框架
- Ruby，有 Sinatra 等框架
- PHP，有 Laravel 等框架

**课程 PJ 不限制后端的实现语言与框架**，同学们可以自由选择。

本次 Lab 贴合课程内容，以 Spring Boot 为例，开发一个简单的 Restful 服务。

## 环境依赖

### JDK 与 JRE

没有 Java 环境的电脑需要同学们去安装 JDK 与 JRE，推荐 1.8 版本。

### Maven

Maven 是 Java 的库管理工具，其功能与 npm 比较类似。

如果不安装 Maven，我们也可以通过手动将 Jar 包放到程序的 classpath 中来添加依赖文件，就像前端也可以手动下载 js 库并添加至页面中一样。但是这样的方式不利于管理依赖文件，因此推荐使用 Maven。

下载 Maven: http://maven.apache.org/download.cgi 或解压 Lab 目录中的 `apache-maven-3.5.0-bin.zip`

安装 Maven: http://maven.apache.org/install.html

## 新建 Spring Boot 工程

同学们可以使用 Lab 目录中的 lab2-seed 工程，或者自己新建 Spring Boot 工程。注意新建工程时修改 `pom.xml` 中的 `groupId` 和 `artifactId`。

新建 Spring Boot 工程参考官方教程：https://spring.io/guides/gs/spring-boot/。按此教程新建完工程后，在 IntelliJ 中打开要选择 `Import Project` 或者菜单栏中的 `File - Project from Existing Source`， 并在其中选择 Maven 类型。

使用 IntelliJ IDEA Ultimate 版本的同学新建工程时也可以参考：https://www.jetbrains.com/help/idea/creating-spring-boot-projects.html

搭建完 Spring Boot 工程后，首先运行 `maven install` 安装所需依赖。

## 运行 Spring Boot

以 Lab 目录中的 lab2-seed 工程为例，`Application.java` 为工程的入口，运行此类便可运行我们的 Spring Boot 工程。

代码如下：

```java
package adweb.lab2;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

`SpringApplication.run(Application.class, args);` 一行运行了我们的 Spring 工程。Spring Boot 会搜索工程中所有的 Controller，并注册对应的接口。然后 Spring Boot 会运行内置的 Tomcat 服务器来提供服务。

在 lab2 工程中已经包含一个接口，查看 `controller/HelloController.java` 文件：

```java
package adweb.lab2.controller;

import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;

@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String index() {
        return "Hello, World!";
    }

}
```

首先，`@RestController` 这个 Annotation 标记了这个类为 Restful Service 的一个 Controller，这是每个 Controller 在类名前必须加的 Annotation。

`@RequestMapping("/hello")` 表示 `index` 这个方法注册了 `/hello` 这个 path，可以通过在浏览器中打开 `http://localhost:8080/hello` 来访问这个接口。接口的 HTTP method 默认为 `GET` ，其他 method 会在后面讲述。

> 注意，这个接口返回的只是一个 String，不是 JSON，所以这个接口并不是一个符合 Restful 标准的接口。在写代码时要避免出现这种不标准的情况。

## 创建第一个 Restful 接口

接下来我们创建一个简单的 Restful 接口。

接口 path 为 `/greeting`，接收一个 url 参数 `name`，方法为 `GET`。

样例 URL：`http://localhost:8080/greeting?name=Peter`。

样例返回：

```json
{
	"id":1,
	"name":"Hello, Peter!"
}
```

实现方式如下：

首先为返回的 JSON 对象创建对应的 Bean 类，新建 `response` 包并新建 `GreetingResponse`，代码如下：

```java
package adweb.lab2.response;

public class GreetingResponse {
    private final long id;
    private final String name;

    public GreetingResponse(long id, String name) {
        this.id = id;
        this.name = name;
    }

    public long getId() {
        return id;
    }

    public String getName() {
        return name;
    }
}
```

注意这个类当中，最主要的是 `getId` 和 `getName` 这两个 getter 方法，这两个方法决定了返回的 JSON 的样式。

>  Java Bean 和 JSON 直接的对应关系是由 `com.fasterxml.jackson.core` 这个包提供的，这个包被包含在了我们的 `spring-boot-starter-web` 中，如果是使用的 `spring-boot` 包，需要手动添加 `jackson` 包才能支持 Java Bean 和 JSON 之间的 Mapping。
>
>  具体包依赖关系在 pom.xml 中的 dependencies 中配置。

接下去让我们创建接口函数。在 `HelloController` 中添加如下代码：

```java
private final AtomicLong counter = new AtomicLong();

@RequestMapping("/greeting")
public @ResponseBody GreetingResponse greeting(@RequestParam(value = "name", defaultValue = "World") String name){
  return new GreetingResponse(counter.incrementAndGet(), "Hello, " + name + "!");
}
```

`counter` 是用来从 1 开始计数的，每次接口被调用时将 `counter` 加一。

`@RequestMapping("/greeting")` 将本接口注册到 `/greeting` 上。

`@ResponseBody GreetingResponse` 指定了接口返回的数据类型。

方法参数`@RequestParam(value = "name", defaultValue = "World") String name` 中，首先参数是 `String name`，`@RequestParam` 指定了这个参数是在 URL 的 path 上的一个参数，而非在 header 中或者 body 中。 `value = "name"` 指定了这个变量对应 path 上的 `name` 参数。`defaultValue = "World"` 表示如果没有 `name` 参数，则将变量的值赋值为 `World`。

函数内部可以自由编写，最后返回一个 GreetingResponse 对象即可。

## 支持跨域访问

目前我们的后端不能被前端直接调用，这是因为我们还没有设置跨域访问的权限。

比如当我们在浏览器新的 Tab 页面的 console 中使用 ajax 调用后端服务，会出现如下报错：

> XMLHttpRequest cannot load http://localhost:8080/hello. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin '...' is therefore not allowed access.

为了让前端能够调用后端服务，我们可以将前端域名添加到头部 `Access-Control-Allow-Origin` 属性中。

在接口函数前添加 `@CrossOrigin(origins = "<前端 hostname，如 http://localhost:9000>")`，或者方便起见允许来自所有源的请求：

```java
    @CrossOrigin(origins = "*")
    @RequestMapping("/hello")
    public String index() {
        return "Hello, World!";
    }
```

## 接口方法定义

之前我们的接口没有指定接口方法，这些接口会默认方法为 `GET` 。

修改 `@RequestMapping` 的参数可以定义其他方法，如 

```java
@RequestMapping(value = "/register", method = RequestMethod.POST)
```

是注册在 /register 的 POST 方法上的。

对于同一个 path，可以注册多个方法不一样的接口。

## 输入参数

### @RequestParam

`@RequestParam` 用来接收 path 中的变量，一个接口可以定义多个`@RequestParam` 变量，如：

```java
public @ResponseBody GreetingResponse greeting(@RequestParam(value = "name") String name, @RequestParam(value = "message") String message)
```

### @RequestBody

`@RequestBody` 用来接收 body 中的变量，该变量一般是 JSON 对象，我们可以像定义 `GreetingResponse` 一样定义 body 的输入。

比如当 body 的数据是用户注册数据时，body 数据如下：

```json
{
	"username": "new-user",
	"password": "123456",
	"email": "a@b.com",
	"phone": "12345678900"
}
```

我们对应新建一个 `UserRegisterRequest.java` ：

```java
package adweb.lab2.request;

public class UserRegisterRequest {
    private String username;
    private String password;
    private String email;
    private String phone;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

	// 省略剩下的 getter 与 setter。
}
```

> 注意：对应 RequestBody 的类不能有 Constructor，这是 jackson 库的一个限制。

### @RequestHeader

与上述两个类似， `@RequestHeader` 可以获取 Header 中的数据。一般不常用

除了 `@RequestBody` 只能出现一次外，`@RequestHeader` 和 `@RequestParam` 都可以出现多次。

## 返回数据

标准的 Restful 服务返回的数据必须是 JSON，不能是一个字符串或者一个数字。

有时同一个接口可能返回不同结构的 JSON 数据，比如用户注册可能有注册成功和注册失败两种情况，会返回不同的数据。一个解决方法是把返回的数据类型改为 `Object` ，代码如下：

```java
@RestController
@RequestMapping("/user")
public class UserController {

    @RequestMapping(value = "/register", method = RequestMethod.POST)
    public @ResponseBody Object register(@RequestBody UserRegisterRequest request){
      if (/* success */ true) {
        return new RegisterResponse(/* ... */);
      } else {
        return new ErrorResponse(/* ... */);
      }
    }

}
```

> 在类之前也可以加上 @RequestMapping。这样做的话，所有函数的 @RequestMapping 的 path 之前都会加上这个 path。比如上面代码中，注册的 path 是 /user/register。

## 测试接口

对于 GET 方法，我们可以通过浏览器很方便地测试，但是对于其他方法，就无法这样测试了。因此推荐使用 Postman 进行接口测试。

Postman 可以从 Google 网上应用店中下载：https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=zh-CN

Postman 使用方法简单说明，以 /user/register 为例：

首先指定 HTTP method 为 POST，然后输入 url 为 localhost:8080/user/register。

接下去填写 body 数据。选择 Body，选择 raw，在右侧类型选择 `JSON(application/json)`，然后输入 body 数据。

最后点击 `Send` 发送请求，在下方可以查看到运行结果

示意图： ![Screen Shot 2017-06-03 at 1.00.09 AM](https://cloud.githubusercontent.com/assets/6532225/26740140/f9a506be-4806-11e7-88b2-d9e748263fbb.png)

Postman 可以团队协作、分享，使用样例可以让前端开发者快速了解掌握后端 API 的使用方式。

## 用户身份认证机制

### token 验证

Restful 服务的用户身份认证推荐使用 token 验证的方式。

> Reference：
>
> https://ninghao.net/blog/2834
>
> http://www.cnblogs.com/xiekeli/p/5607107.html

简单来说，token 验证是在用户登录成功后将用户名、token 发行时间和过期时间等信息进行加密，并将生成的密文发回给客户端作为身份认证的凭证。这个 token 密文只有服务端能解密，当用户进行权限相关的操作时，需要将 token 发回给服务端，服务端解密验证通过后进行相关操作。在这个过程中，服务端不需要储存任何用户登录状态，相比传统的 session 的方式，实现了服务端的无状态化。

### session 验证

传统的 session 验证仍旧是一个优秀的身份认证机制。同学们可以使用 Spring Session，或者自己手动编写自定义的 session 管理代码来进行用户身份认证。

> Spring Session: http://projects.spring.io/spring-session/

## 继续学习 Spring Boot

Get started: https://spring.io/guides/gs/spring-boot/

Full reference: http://docs.spring.io/spring-boot/docs/2.0.0.BUILD-SNAPSHOT/reference/htmlsingle/#getting-started-installing-the-cli

Learn more about spring: https://spring.io/

# Part 2: MyBatis

MyBatis 是一个后端数据库层面的框架，类似的框架还有 Hibernate。同学们可以在 PJ 中自由选择后端数据库层面的实现方式，但不太推荐

## 概述

MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以对配置和原生Map使用简单的 XML 或注解，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

## 环境依赖

### 数据库

MyBatis 支持各种数据库，本次 Lab 中以 MySQL 为例。

### Maven

将下列两个依赖加入到 pom.xml 的 dependencies 中：

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>3.4.4</version>
</dependency>

<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>6.0.6</version>
</dependency>
```

然后运行 `maven install` 安装这两个包。

## 官方文档

MyBatis 的官方文档将 MyBatis 的各方面阐述得很清楚，本 Lab 中不重复其中的内容。

**请同学们首先阅读 MyBatis 官方文档来学习 MyBatis**。Lab 将展示一个结合 Spring Boot 的样例程序。

中文：http://www.mybatis.org/mybatis-3/zh/index.html

英文：http://www.mybatis.org/mybatis-3/index.html

## 样例代码

> 加入了 MyBatis 的样例代码在 lab2-half 工程中。同学们可以直接打开查看。

我们接着上文中 Spring Boot 创建的工程，将 MyBatis 加入进来。

最终的工程目录结构如图所示：

 ![Screen Shot 2017-06-03 at 2.09.42 AM](https://cloud.githubusercontent.com/assets/6532225/26740141/f9a59296-4806-11e7-9c40-666c02a1acb0.png)

1. 创建 MyBatis 核心配置文件 `SqlMapConfig.xml`：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <properties resource="db.properties">

  </properties>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="mapper/UserMapper.xml"/>
  </mappers>
</configuration>
```

2. properties 在 `db.properties` 文件中定义：

```
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/adweb_lab2?characterEncoding=utf-8
username=root
password=123456
```

3. 对应地，在 MySQL 中创建新的 Schema `adweb-lab2`。
4. 使用 `lab2.sql` 创建表：

```sql
DROP TABLE IF EXISTS Article;
DROP TABLE IF EXISTS User;

CREATE TABLE IF NOT EXISTS User (
  userID INT(11) NOT NULL AUTO_INCREMENT,
  username VARCHAR(50) NOT NULL,
  password VARCHAR(50) NOT NULL,
  email VARCHAR(100) DEFAULT NULL,
  phone VARCHAR(20) DEFAULT NULL,
  PRIMARY KEY (UserID)
);

INSERT INTO User (userID, username, password, email, phone) VALUES
  (1, 'kaiyudai', '12345678', 'kydai@fudan.edu.cn', '13666666666'),
  (2, 'fengshuangli', '12345678', '13302010002@fudan.edu.cn', '13888888888'),
  (3, 'zhongyitong', '12345678', NULL, NULL);

CREATE TABLE IF NOT EXISTS Article (
  articleID INT(11) NOT NULL AUTO_INCREMENT,
  userID INT(11) NOT NULL,
  title VARCHAR(100) NOT NULL,
  content VARCHAR(5000) NOT NULL,
  PRIMARY KEY (articleID),
  FOREIGN KEY (userID) REFERENCES User(userID)
);
```

5. 新建 `SqlSessionLoader.java` 来载入 MyBatis：

```java
package adweb.lab2.mybatis;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

public class SqlSessionLoader {

    private static SqlSessionFactory sqlSessionFactory;

    public static SqlSessionFactory getSqlSessionFactory() throws IOException {
        if (sqlSessionFactory == null) {
            InputStream inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        }
        return sqlSessionFactory;
    }

    public static SqlSession getSqlSession() throws IOException {
        return getSqlSessionFactory().openSession();
    }

}
```

6. 为了使程序能够找到 `SqlMapConfig.xml` 文件，在 `pom.xml` 的 `build` 标签中添加如下代码：

```xml
<resources>
  <resource>
    <directory>src/main/java/adweb/lab2/mybatis/config</directory>
    <includes>
      <include>**/*.xml</include>
      <include>**/*.properties</include>
    </includes>
    <filtering>true</filtering>
  </resource>
</resources>
```

7. 创建 `UserMapper.xml` 来定义对 User 表的操作映射：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="adweb.lab2.UserMapper">

  <select id="findUserById" parameterType="int" resultType="adweb.lab2.mybatis.po.User">
    select * from User where userID = #{userID}
  </select>

  <select id="findUserByUsername" parameterType="java.lang.String" resultType="adweb.lab2.mybatis.po.User">
    select * from User where username = #{username}
  </select>

  <insert id="addUser" parameterType="adweb.lab2.mybatis.po.User" useGeneratedKeys="true" keyProperty="userID">
    insert into User (username, password, email, phone)
    values (#{username}, #{password}, #{email}, #{phone})
  </insert>

</mapper>
```

Mapper 是 MyBatis 核心功能，注意仔细阅读相关文档，理解 Mapper 的工作原理。

8. 创建 Plain Object `User.java`：

```java
package adweb.lab2.mybatis.po;

public class User {
    private int userID;
    private String username;
    private String password;
    private String email;
    private String phone;

    public User(int userID, String username, String password, String email, String phone) {
        this.userID = userID;
        this.username = username;
        this.password = password;
        this.email = email;
        this.phone = phone;
    }

    public User(String username, String password, String email, String phone) {
        this.username = username;
        this.password = password;
        this.email = email;
        this.phone = phone;
    }

    public int getUserID() {
        return userID;
    }

    public void setUserID(int userID) {
        this.userID = userID;
    }

	/* 省略其他 setter 与 getter */
}

```

9. 修改 UserController.java 中注册用户的对应代码：

```java
    @RequestMapping(value = "/register", method = RequestMethod.POST)
    public @ResponseBody Object register(@RequestBody UserRegisterRequest request) throws IOException {
        SqlSession sqlSession = SqlSessionLoader.getSqlSession();
        User user = sqlSession.selectOne("adweb.lab2.UserMapper.findUserByUsername", request.getUsername());
        if (user != null) {
            sqlSession.close();
            return new ErrorResponse("The username is already used");
        } else {
            sqlSession.insert("adweb.lab2.UserMapper.addUser", new User(request.getUsername(), request.getPassword(), request.getEmail(), request.getPhone()));
            sqlSession.commit();
            sqlSession.close();
            return new UserResponse("abc"); // use your generated token here.
        }
    }
```

通过以上 9 个步骤，我们将 MyBatis 加入到了 Spring Boot 工程中，并进行了简单的运用。

以上代码有些地方写的比较粗糙，在结构上有改进空间。

# Part 3. 练习

目前，Lab 工程已经完成了用户注册的流程，请同学们设计 `用户登录` 和 `列举所有用户` 这两个接口，结合Spring Boot 和 MyBatis 编写对应的代码。

仍有剩余时间的同学可以尝试添加用户身份认证相关代码，继续学习 Spring Boot 和 MyBatis 或将 Lab 所学内容运用于 PJ 中。

本练习不需要提交。