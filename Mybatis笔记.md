## 1、 简介

### 1.1 Mybatis

- 优秀的**持久层框架**
- 支持定制化SQL、存储过程以及高级映射
- 避免了几乎所有的JDBC代码合手动设置菜蔬以及获取结果集
- 用简单的XML或注解来配置和映射原生类型、接口和Java的POJO（Plain Old Java Objects，普通老式Java对象）为数据库中的记录



- maven仓库：

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```

- 中文官网：https://mybatis.org/mybatis-3/zh/getting-started.html

### 1.2 持久化

数据持久化

- 将程序的数据在持久状态和瞬时状态的转化过程
- 内存： **断电及失** 
- 数据库（jdbc），io文件持久化

### 1.3 持久层

Dao层，Service层，Controller层...

- 完成持久化工作的代码块
- 层界限十分明显

### 1.4 为什么需要Mybatis

- 方便。
- 传统JDBC代码复杂。简化。框架。自动化。
- 将数据存入数据库。
- 优点：
  - 简单易学
  - 灵活
  - 解除sql与程序代码的耦合：通过提供DAO层，将业务逻辑和数据访问逻辑分离，使系统的设计更清晰，更易维护，更易单元测试。sql和代码的分离，提高了可维护性。
  - **提供映射标签，支持对象与数据库的orm字段关系映射**
  - **提供对象关系映射标签，支持对象关系组建维护**
  - **提供xml标签，支持编写动态sql。**



## 2、第一个Mybatis程序

### 2.1 搭建环境

- 搭建数据库

- 新建项目
  - 新建一个普通的maven项目
  - 删除src目录
  - 导入maven依赖

### 2.2 创建一个模块

- 编写mybatis的核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--configuration核心配置文件-->
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://127.0.0.1:3306/mybatis?useSSl=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="a12304560"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

- 编写mybatis的工具类

```java
public class MybatisUtils {
    private static SqlSessionFactory sqlSessionFactory;

    static{
        try {
            // 使用Mybatis第一步： 获取sqlSessionFactory对象
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 既然有了 SqlSessionFactory，顾名思义，我们可以从中获得 SqlSession 的实例。
    // SqlSession 提供了在数据库执行 SQL 命令所需的所有方法
    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();
    }
}
```

### 2.3 编写代码

- pojo实体类

```java
public class User {
    private int id;
    private String name;
    private String pwd;

    public User() {
    }

    public User(int id, String name, String pwd) {
        this.id = id;
        this.name = name;
        this.pwd = pwd;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPwd() {
        return pwd;
    }

    public void setPwd(String pwd) {
        this.pwd = pwd;
    }
}
```

- Dao接口

```java
public interface UserDao {
    List<UserDao> getUserList();
}
```

- 接口实现类由原来的impl转换为一个Mapper配置文件，文件放在resources目录下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--namespace：绑定一个对应的Dao接口/Mappe接口-->
<mapper namespace="com.hhhqzh.dao.UserDao">
    <select id="getUserList" resultType="com.hhhqzh.pojo.User">
        select * from mybatis.user
    </select>
</mapper>
```

### 2.4 测试

注意点：org.apache.ibatis.binding.BindingException: Type interface com.hhhqzh.dao.UserDao is not known to the MapperRegistry.

**MapperRegistry是什么**

配置核心文件中注册 mappers

每个Mapper.xml都需要在Mybatis核心配置文件中注册

- junit测试

```java
public class UserDaoTest {

    @Test
    public void test(){

        // 第一步： 获得SqlSession对象
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        // 第二步：执行SQL

        // 方式一：getMapper
        UserDao userDao = sqlSession.getMapper(UserDao.class);
        List<User> userList = userDao.getUserList();

        for (User user : userList) {
            System.out.println(user);
        }

        // 关闭sqlSession
        sqlSession.close();
    }
}
```



## 3、CRUD

### 2.1、namespace

namespace中的报名要和Dao接口的包名一致

### 2.2、select

- id：namespace对应的包中的方法

- resultType：sql语句执行的返回值

- parameterType：参数类型可以是基本数据类型(int,String,long),可以是Model对象,也可以是Map;

- 如果传多个参数，可以在方法中使用注解@Param

  ```java
  int addUser(@Parma("id") int id, @Parma("name") String name)
  ```

  

  1. 编写接口

  ```java
  // 根据id查询用户
  User getUserById(int id);
  ```

  2. 编写对应的mapper中的sql语句

  ```xml
  <select id="getUserById" parameterType="int" resultType="com.hhhqzh.pojo.User">
      select * from mybatis.user where id = #{id}
  </select>
  ```

  3. 测试接口

### 2.3、insert

```xml
<!--    对象中的属性可以直接取出来-->
    <insert id="addUser" parameterType="com.hhhqzh.pojo.User">
        insert into mybatis.user (id, name, pwd) value (#{id}, #{name}, #{pwd})
    </insert>
```

### 2.4、update

```xml
<update id="updateUser" parameterType="com.hhhqzh.pojo.User">
    update mybatis.user set name = #{name}, pwd = #{pwd}  where id = #{id};
</update>
```

### 2.5、delete

```xml
<delete id="deleteUser" parameterType="int">
    delete from mybatis.user where id = #{id}
</delete>
```



***注意点***

- 增删改需要提交事务：sqlSession.commit();

### 2.6、分析错误

- 标签不要匹配错
- resource绑定mapper，需要使用路径
- 程序配置文件必须符合规范
- 输出的xml文件中存在中文乱码
- **xml文件要放在resources文件下面**

### 2.7、万能的Map

假设，实体类或者数据库中的表，字段或者参数过多，应当考虑map。



Dao：

```java
// 万能的Map
int addUser2(Map<String, Object> map);
```

Xml：

```xml
<!--传递map的key-->
<update id="updateUser" parameterType="com.hhhqzh.pojo.User">
    update mybatis.user set name = #{name}, pwd = #{pwd}  where id = #{id};
</update>
```

Controller：

```java
Map<String, Object> map = new HashMap<String, Object>();

map.put("userId", 5);
map.put("userName", "hhhqzh");
map.put("password", "12345");

userDao.addUser2(map);
```



Map传参，直接在sql中取出key即可！parameterType=“map"

对象传递餐宿，直接在sql中取对象的属性！parameterType="Object"

只有一个基本类型参数的情况下，可以直接在sql中取到！

多个参数使用Map或**@Param注解**



## 4、配置解析

### 4.1 、核心配置文件

- mybatis-config.xml
- MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。 

``` 
configuration（配置）

    properties（属性）
    settings（设置）
    typeAliases（类型别名）
    typeHandlers（类型处理器）
    objectFactory（对象工厂）
    plugins（插件）
    environments（环境配置）
        environment（环境变量）
            transactionManager（事务管理器）
            dataSource（数据源）
    databaseIdProvider（数据库厂商标识）
    mappers（映射器）

```

### 4.2、环境配置（environments）

MyBatis 可以配置成适应多种环境。

**不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory          实例只能选择一种环境。**

​	所以，如果你想连接两个数据库，就需要创建两个 SqlSessionFactory          实例，每个数据库对应一个。而如果是三个数据库，就需要三个实例，依此类推。

![image-20200726181924747](C:\Users\98538\AppData\Roaming\Typora\typora-user-images\image-20200726181924747.png)

### 4.3、属性（properties）

可以通过properties属性来实现引用配置文件

这些属性可以在外部进行配置，并可以进行动态替换。你既可以在典型的 Java 属性文件【db.properties】中配置这些属性，也可以在 properties 元素的子元素中设置。

- 编写db.properties文件

```properties
driver = com.mysql.cj.jdbc.Driver
url = jdbc:mysql://127.0.0.1:3306/mybatis?useSSl=false&useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
username = root
password = a12304560
```

- 在核心配置文件中引入（可以引入外部文件，也可以在配置属性中增加属性配置）

```xml
<!--引入外部配置文件，且优先使用外部配置文件-->
<properties resource="db.properties">
    <property name="username" value="root"/>
    <property name="password" value="a12304560"/>
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
```

### 4.4、类型别名（typeAliases）

- 实体类较少时

  类型别名可为 Java 类型设置一个缩写名字。它仅用于 XML 配置，意在降低冗余的全限定类名书写，当这样配置时，`User`用 `com.hhhqzh.pojo.User` 的地方。例如：

``` xml
<!--给实体类起别名-->
    <typeAliases>
        <typeAlias alias="User" type="com.hhhqzh.pojo.User"/>
    </typeAliases>
```

- 实体类较多时

  也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，扫描实体类的包，它的默认别名就为这个类的类名，首字母小写（大写也行）。比如：

``` xml
<!--给实体类起别名-->
    <typeAliases>
        <package name="com.hhhqzh.pojo"/>
    </typeAliases>
```

​		在没有注解的情况下 `com.hhhqzh.pojo.User` 的别名为`user`；**若有注解，则别名为其注解值**，比如：

``` java
@Alias("hahha")
public class User {
    ...
}
```

### 4.5、设置（Setting）

![image-20200726184752268](C:\Users\98538\AppData\Roaming\Typora\typora-user-images\image-20200726184752268.png)



![image-20200726184803341](C:\Users\98538\AppData\Roaming\Typora\typora-user-images\image-20200726184803341.png)

### 4.6、其他配置

- typeHandlers（类型处理器）

- objectFactory（对象工厂）

- plugins（插件）
  - mybatis-generator-core
  - mybatis-plus
  - 通用mapper

### 4.7、映射器（mappers）

MapperRegistry：注册绑定我们的Mapper文件。

- 方式一：

``` xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
</mappers>
```

​	Mapper配置文件所存放的目录无限制。

- 方式二：使用class文件绑定注册

``` xml
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
</mappers>
```

- 方式三：使用扫描包的方式注入绑定

```xml
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

​	方式二和三的注意点：

​			1、接口和它的Mapper配置文件必须同名。

​			2、接口和它的Mapper配置文件必须在同一个包下，也			就是mapper放在resources目录下会报错。

### 4.8、生命周期和作用域

![image-20200727095214890](C:\Users\98538\AppData\Roaming\Typora\typora-user-images\image-20200727095214890.png)

不同作用域和生命周期类别是至关重要的，因为错误的使用会导致非常严重的**并发问题**。

**SqlSessionFactoryBuilder：**

- 这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。
- 最佳作用域是方法作用域（也就是**局部方法变量**）。



**SqlSessionFactory：**

- 说白了就是：数据库连接池。
- SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，**没有任何理由丢弃它或重新创建另一个实例**。
- 最佳作用域是应用作用域（**单例模式或者静态单例模式,只有一个变量**）。



**SqlSession：**

- 连接到连接池的一个请求！
- SqlSession的实例不是线程安全的，因此是不能被共享的。
- 最佳的作用域是**请求或方法作用域**。
- 用完之后必须关闭，否则资源被占用！
- 绝对不能将 SqlSession 实例的引用放在一个类的静态域。

![image-20200727124709361](C:\Users\98538\AppData\Roaming\Typora\typora-user-images\image-20200727124709361.png)

​	这里面sqlSession获取的每一个Mapper，就代表一个业务

## 5、解决属性名与字段名不一致的问题、

### 5.1、问题

数据库中的字段：

![image-20200727143552793](C:\Users\98538\AppData\Roaming\Typora\typora-user-images\image-20200727143552793.png)

项目中实体类的字段：

![image-20200727143637427](C:\Users\98538\AppData\Roaming\Typora\typora-user-images\image-20200727143637427.png)

getUserById测试出现问题，未能映射到password

![image-20200727144032364](C:\Users\98538\AppData\Roaming\Typora\typora-user-images\image-20200727144032364.png)

``` mysql
select * from mybatis.user where id = #{id}
```



解决方法：

- 给返回的字段起别名，与实体类的字段对应

```mysql
select id, name, pwd as password from mybatis.user where id = #{id}
```



### 5.2、 resultMap

结果集映射

```xml
<!--结果集映射-->
<resultMap id="UserMap" type="User">
    <!--column数据库中的字段，property实体类中的属性-->
    <result column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="pwd" property="password"/>
</resultMap>

<select id="getUserById" resultMap="UserMap">
    select * from mybatis.user where id = #{id}
</select>
```

- `resultMap` 元素是 MyBatis 中最重要最强大的元素

- `resultMap` 的设计思想是，对于简单的语句根本不需要配置显示的结果映射，二对于复杂一点的语句只需要描述它们的关系就行了。

## 6、日志

### 6.1、日志工厂

如果数据库操作出现异常，需要排错，使用日志！！



曾经：sout、debug

现在：日志工厂！！



核心配置文件mybatis-config.xml中配置属性（setting）里的设置：具体使用哪一个，在设置中设定

![image-20200726184803341](C:\Users\98538\AppData\Roaming\Typora\typora-user-images\image-20200726184803341.png)

- SLF4J 
- **LOG4J**  【掌握】
- LOG4J2
-  JDK_LOGGING 
- COMMONS_LOGGING 
- **STDOUT_LOGGING **【掌握】
- NO_LOGGING              



**STDOUT_LOGGING 标准日志输出**

```xml
<!--mybatis-config配置文件中配置-->
<settings>
    <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```



![image-20200727151940166](C:\Users\98538\AppData\Roaming\Typora\typora-user-images\image-20200727151940166.png)

### 6.2、Log4j

什么是Log4j

- Log4j是[Apache](https://baike.baidu.com/item/Apache/8512995)的一个开源项目，通过使用Log4j，我们可以控制日志信息输送的目的地是[控制台](https://baike.baidu.com/item/控制台/2438626)、文件、[GUI](https://baike.baidu.com/item/GUI)组件。
- 我们控制每一条日志的输出格式。
- 通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。
- 通过一个[配置文件](https://baike.baidu.com/item/配置文件/286550)来灵活地进行配置，而不需要修改应用的代码。

​	1、先导入Log4j的包

``` xml
<!-- https://mvnrepository.com/artifact/log4j/log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

​	2、log4j.properties

``` properties
log4j.rootLogger=DEBUG,console,file

#控制台输出的相关设置
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%c]-%m%n

#文件输出的相关设置
log4j.appender.file = org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./log/hhhqzh.log
log4j.appender.file.MaxFileSize=10mb
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n

#日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```

​	3、 在mybatis-config核心配置文件中配置log4j为日志的实现

``` xml
<!--mybatis-config配置文件中配置-->
<settings>
    <setting name="logImpl" value="LOG4J"/>
</settings>
```

​	4、Log4j的使用，直接测试运行

![image-20200727155346855](C:\Users\98538\AppData\Roaming\Typora\typora-user-images\image-20200727155346855.png)

**简单使用**

​	1、在要使用Log4j的类中导入包 import org.apache.log4j.Logger;

​	2、日志对象，加载参数为当前类的class

```java
    static Logger logger = Logger.getLogger(UserDaoTest.class);
```

​	3、 日志级别

```java
    logger.info("info:进入了testLog4j");
    logger.debug("debug:进入了testLog4j");
    logger.error("error:进入了testLog4j");
```

## 7、分页

### 7.1、使用limit分页

```mysql
select * from user limit startIndex, pageSize;
```



使用Mybatis实现分页，核心Sql

​	1、接口

```java
// 分页查询
List<User> getUserByLimit(Map<String, Integer> map);
```

​	2、Mapper.xml

```xml
<select id="getUserByLimit" parameterType="map" resultMap="UserMap">
    select * from mybatis.user limit #{startIndex}, #{pageSize};
</select>
```

​	3、测试

```java
@Test
public void getUserByLimit(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserDao userDao = sqlSession.getMapper(UserDao.class);

    Map<String, Integer> map = new HashMap<String, Integer>();
    map.put("startIndex", 0);
    map.put("pageSize", 2);

    List<User> userList = userDao.getUserByLimit(map);
    for (User user : userList) {
        System.out.println(user);
    }
    sqlSession.close();
}
```

### 7.2、使使用RowBounds分页

不在使用SQL实现分页

​	1、接口

```java
List<User> getUserByRowBounds();
```

​	2、Mapper.xml

```xml
<!--选择全部-->
<select id="getUserByRowBounds" resultMap="UserMap">
    select * from mybatis.user
</select>
```

3、测试

```java
@Test
public void getUserByRowBounds(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();

    //通过RowBounds实现
    RowBounds rowBounds = new RowBounds(1, 2);

    List<User> userList = sqlSession.selectList("com.hhhqzh.dao.UserDao.getUserByRowBounds", null, rowBounds);
    for (User user : userList) {
        System.out.println(user);
    }
    sqlSession.close();
}
```

### 7.3 、分页插件

pagehelper

## 8、注解

​	@Select

​	@Insert

​	@Update

​	@Delete

## 9、多对一处理

多对一的理解：

​	多个学生对应一个老师

​	如果对于学生这边，就是一个多对一的现象，即从学生这边关联一个老师！

- **数据库搭建**

```sql
CREATE TABLE teacher (
  id INT(10) NOT NULL,
  name VARCHAR(30) DEFAULT NULL,
  PRIMARY KEY (id)
) ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO teacher(id, name) VALUES (1, '秦老师'); 

CREATE TABLE student (
  id INT(10) NOT NULL,
  name VARCHAR(30) DEFAULT NULL,
  tid INT(10) DEFAULT NULL,
  PRIMARY KEY (id),
  KEY fktid (tid),
  CONSTRAINT fktid FOREIGN KEY (tid) REFERENCES teacher (id)
) ENGINE=INNODB DEFAULT CHARSET=utf8


INSERT INTO student (id, name, tid) VALUES ('1', '小明', '1'); 
INSERT INTO student (id, name, tid) VALUES ('2', '小红', '1'); 
INSERT INTO student (id, name, tid) VALUES ('3', '小张', '1'); 
INSERT INTO student (id, name, tid) VALUES ('4', '小李', '1'); 
INSERT INTO student (id, name, tid) VALUES ('5', '小王', '1'); 
```



- **pojo**

```java
//GET,SET,ToString，有参，无参构造
public class Teacher {
    private int id;
    private String name;
}
```

```java
//Get,Set,ToString, 有参，无参构造
public class Student {
    private int id;
    private String name;
    //多个学生可以是同一个老师，即多对一
    private Teacher teacher;
}
```



* **接口编写方法：**

```java
//获取所有学生及对应老师的信息
public List<Student> getStudents();
```

### 9.1、按查询嵌套查询

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ttt.mapper.StudentMapper">

    <select id="getStudents" resultMap="StudentTeacher">
      select * from student
    </select>
    <resultMap id="StudentTeacher" type="Student">
        <!--association关联属性  property属性名 javaType属性类型 column在多的一方的表中的列名-->
        <association property="teacher"  column="tid" javaType="Teacher" select="getTeacher"/>
    </resultMap>
    <!--
    这里传递过来的id，只有一个属性的时候，下面可以写任何值
    association中column多参数配置：
        column="{key=value,key=value}"
        其实就是键值对的形式，key是传给下个sql的取值名称，value是片段一中sql查询的字段名。
    -->
    <select id="getTeacher" resultType="teacher">
        select * from teacher where id = #{id}
    </select>

</mapper>
```

### 9.2、按结果嵌套查询

```
<select id="getStudents2" resultMap="StudentTeacher2" >
    select s.id sid, s.name sname , t.name tname
    from student s,teacher t
    where s.tid = t.id
</select>

<resultMap id="StudentTeacher2" type="Student">
    <id property="id" column="sid"/>
    <result property="name" column="sname"/>
    <!--关联对象property 关联对象在Student实体类中的属性-->
    <association property="teacher" javaType="Teacher">
        <result property="name" column="tname"/>
    </association>
</resultMap>
```

**在核心配置文件注册Mapper.xml。**

 ## 10、一对多处理

一对多的理解：

- 一个老师拥有多个学生
- 如果对于老师这边，就是一个一对多的现象。



- pojo

  ```java
  //GET,SET,ToString，有参，无参构造
  public class Teacher {
      private int id;
      private String name;
      //一个老师可以对多个学生，一对多
      private List<Student> students;
  }
  ```

  ```java
  //Get,Set,ToString, 有参，无参构造
  public class Student {
      private int id;
      private String name;
      private int tid;
  }
  ```

  

- **接口编写方法：**

```
//获取指定老师，及老师下的所有学生
public Teacher getTeacher(int id);
```



### 10.1、按查询嵌套查询

```xml
<select id="getTeacher2" resultMap="TeacherStudent2">
  select * from teacher where id = #{id}
</select>
<resultMap id="TeacherStudent2" type="Teacher">
    <!--可以省略-->
    <result  property="id" column="tid"/>
    <result  property="name" column="tname"/>
    <!--column是一对多的外键 , 写的是一的主键的列名-->
    <collection property="students" javaType="ArrayList" ofType="Student" column="id" select="getStudentByTeacherId"/>
</resultMap>
<select id="getStudentByTeacherId" resultType="Student">
    select * from student where tid = #{id}
</select>
```



### 10.2、按结果嵌套查询

```xml
<mapper namespace="com.ttt.mapper.TeacherMapper">
    <select id="getTeacher" resultMap="TeacherStudent">
        select s.id sid, s.name sname , t.name tname, t.id tid
        from student s,teacher t
        where s.tid = t.id and t.id=#{id}
    </select>

    <resultMap id="TeacherStudent" type="Teacher">
        <result  property="id" column="tid"/>
        <result  property="name" column="tname"/>
        <collection property="students" ofType="Student">
            <result property="id" column="sid" />
            <result property="name" column="sname" />
            <result property="tid" column="tid" />
        </collection>
    </resultMap>
</mapper>
```



### ***10.3、小结***

- JavaType是用来指定pojo中属性的类型。

- ofType指定的是映射到**list**集合属性中pojo的类型。
- 关联 - association
- 集合 - collection

![image-20200730100628932](C:\Users\98538\AppData\Roaming\Typora\typora-user-images\image-20200730100628932.png)

## 11、动态SQL

### 11.1、什么是动态SQL

​	根据不同的条件生成不同的SQL语句

````xml
    if
    choose (when, otherwise)
    trim (where, set)
    foreach
````

### 11.2、搭建环境

- 建表

```sql
CREATE TABLE `blog` (
  `id` int(11) NOT NULL,
  `title` varchar(255) NOT NULL,
  `author` varchar(255) NOT NULL,
  `create_time` datetime NOT NULL,
  `views` int(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

- 创建一个基础工程

  1、导包

  2、编写核心配置文件

  3、编写实体类pojo

  ```Java
  @Data
  public class Blog {
      private int id;
      private String title;
      private String author;
      private Date createTime;
      private int views;
  }
  ```

  4、编写实体类对应的Dao接口和Mapper.xml文件

### 11.3、 if

```xml
<select id="queryBlogIF" parameterType="map" resultType="blog">
    select * from mybatis.blog
    <where>
        <if test="title != null">
        	title = #{title}
    	</if>
    	<if test="author != null">
        	and author = #{author}
    	</if>
    </where>
    
</select>
```

***where* 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，*where* 元素也会将它们去除。**

### 11.4、choose（when、otherwise）

​	类似于switch语句

```xml
<select id="findActiveBlogLike"
     resultType="blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

### 11.5、trim（where、set）

- where

```xml
<select id="queryBlogIF" parameterType="map" resultType="blog">
    select * from mybatis.blog
    <where>
        <if test="title != null">
        	title = #{title}
    	</if>
    	<if test="author != null">
        	and author = #{author}
    	</if>
    </where>
 </select>
```



- 用于动态更新语句的类似解决方案叫做 *set*

```xml
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>
```

​	*set* 元素会动态地在行首插入 SET 关键字，并会删掉额外的逗号（这些逗号是在使用条件语句给列赋值时引入的）

### 11.6、SQL片段

​	对公共的SQL语句进行复用

​	1、使用SQL标签抽取公共部分

```xml
<sql id='if-title-author'>
    <if test="title !=null">
        	title = #{title}
    	</if>
    	<if test="author != null">
        	and author = #{author}
    	</if>
</sql>
```

​	2、使用include标签引用

```xml
<select id="queryBlogIF" parameterType="map" resultType="blog">
    select * from mybatis.blog
    <where>
        <include refid="if-title-author"></include></include>
    </where>
 </select>
```

- 注意

  最好基于单表来定义SQL片段！

  不要存在where标签！

### 11.7、foreach

允许你指定一个集合，声明可以在元素体内使用的集合项（item）和索引（index）变量。它也允许你指定开头**(open)**与结尾**(close)**的字符串以及集合项迭代之间的分隔符**(separator)**。

```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

