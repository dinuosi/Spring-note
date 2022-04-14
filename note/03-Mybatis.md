# Mybatis

## 1. 关于Mybatis

Mybatis的主要作用是快速实现对关系型数据库中的数据进行访问的框架。

## 2. 创建整合了Spring与Mybatis的工程

Mybatis可以不依赖于Spring等框架直接使用的，但是，就需要进行大量的配置，前期配置工作量较大，基于Spring框架目前是业内使用的标准之一，所以，通常会整合Spring与Mybatis，以减少配置。

在创建工程时，创建普通的Maven工程即可（不需要选择特定的骨架）。

在`pom.xml`中，需要添加几个依赖项，分别是：

Mybatis的依赖项：`mybatis`

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.6</version>
</dependency>
```

Mybatis整合Spring的依赖项：`mybatis-spring`

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>2.0.6</version>
</dependency>
```

Spring的依赖项：`spring-context`

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.14</version>
</dependency>
```

Spring JDBC的依赖项：`spring-jdbc`

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.3.14</version>
</dependency>
```

MySQL连接的依赖项：`mysql-connector-java`

```xml
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.21</version>
</dependency>
```

数据库连接池的依赖项：`commons-dbcp2`

```xml
<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-dbcp2 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-dbcp2</artifactId>
    <version>2.8.0</version>
</dependency>
```

JUnit测试的依赖项：`junit-jupiter-api`

```xml
<!-- https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-api -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.7.0</version>
    <scope>test</scope>
</dependency>
```

创建完成后，可以在`src/test/java`下创建测试类，并编写测试方法，例如：

```java
package cn.tedu.mybatis;

import org.junit.jupiter.api.Test;

public class MybatisTests {

    @Test
    public void contextLoads() {
        System.out.println("MybatisTests.contextLoads()");
    }

}
```

由于目前尚未编写实质的代码，以上测试代码也非常简单，应该是可以成功通过测试的，如果不能通过测试，必然是开发工具、开发环境、依赖项、项目创建步骤等问题。

## 3. 配置Mybatis的开发环境

首先，登录MySQL控制台，创建名为`mall_ams`的数据库：

```mysql
CREATE DATABASE mall_ams;
```

然后，在IntelliJ IDEA中配置数据库视图：http://doc.canglaoshi.org/doc/idea_database/index.html

然后，通过数据库视图的Console面板创建数据表：

```mysql
create table ams_admin (
    id bigint unsigned auto_increment,
    username varchar(50) default null unique comment '用户名',
    password char(64) default null comment '密码（密文）',
    nickname varchar(50) default null comment '昵称',
    avatar varchar(255) default null comment '头像URL',
    phone varchar(50) default null unique comment '手机号码',
    email varchar(50) default null unique comment '电子邮箱',
    description varchar(255) default null comment '描述',
    is_enable tinyint unsigned default 0 comment '是否启用，1=启用，0=未启用',
    last_login_ip varchar(50) default null comment '最后登录IP地址（冗余）',
    login_count int unsigned default 0 comment '累计登录次数（冗余）',
    gmt_last_login datetime default null comment '最后登录时间（冗余）',
    gmt_create datetime default null comment '数据创建时间',
    gmt_modified datetime default null comment '数据最后修改时间',
    primary key (id)
) comment '管理员表' charset utf8mb4;
```

至此，本案例所需的数据库与数据表已经准备完毕。

在`src/main/resources`下创建`datasource.properties`配置文件，用于配置连接数据库的参数，例如：

```
datasource.url=jdbc:mysql://localhost:3306/mall_ams?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
datasource.driver=com.mysql.cj.jdbc.Driver
datasource.username=root
datasource.password=root
```

并且，在`cn.tedu.mybatis`包下（不存在，则创建）创建`SpringConfig`类，读取以上配置文件：

```java
@Configuration
@PropertySource("classpath:datasource.properties")
public class SpringConfig {
}
```

完成后，在测试方法中补充测试代码：

```java
@Test
public void contextLoads() {
    System.out.println("MybatisTests.contextLoads()");
    AnnotationConfigApplicationContext ac
            = new AnnotationConfigApplicationContext(SpringConfig.class);

    ConfigurableEnvironment environment = ac.getEnvironment();

    System.out.println(environment.getProperty("datasource.url"));
    System.out.println(environment.getProperty("datasource.driver"));
    System.out.println(environment.getProperty("datasource.username"));
    System.out.println(environment.getProperty("datasource.password"));

    ac.close();
}
```

接下来，在`SpringConfig`中配置一个`DataSource`对象：

```java
package cn.tedu.mybatis;

import org.apache.commons.dbcp.BasicDataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;

import javax.sql.DataSource;

@Configuration
@PropertySource("classpath:datasource.properties")
public class SpringConfig {

    @Bean
    public DataSource dataSource(Environment env) {
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setUrl(env.getProperty("datasource.url"));
        dataSource.setDriverClassName(env.getProperty("datasource.driver"));
        dataSource.setUsername(env.getProperty("datasource.username"));
        dataSource.setPassword(env.getProperty("datasource.password"));
        return dataSource;
    }

}
```

并在测试类中添加新的测试方法，以尝试获取数据库的连接对象，检测是否可以正确连接到数据库：

```java
@Test
public void testConnection() throws Exception {
    AnnotationConfigApplicationContext ac
            = new AnnotationConfigApplicationContext(SpringConfig.class);

    DataSource dataSource = ac.getBean("dataSource", DataSource.class);

    Connection connection = dataSource.getConnection();
    System.out.println(connection);

    ac.close();
}
```

至此，项目的数据库编程的准备完毕。

## 4. Mybatis的基本使用

当使用Mybatis实现数据访问时，主要：

- 编写数据访问的抽象方法
- 配置抽象方法对应的SQL语句

关于抽象方法：

- 必须定义在某个接口中，这样的接口通常使用`Mapper`作为名称的后缀，例如`AdminMapper`
  - Mybatis框架底层将通过接口代理模式来实现
- 方法的返回值类型：如果要执行的数据操作是增、删、改类型的，统一使用`int`作为返回值类型，表示“受影响的行数”，也可以使用`void`，但是不推荐；如果要执行的是查询操作，返回值类型只需要能够装载所需的数据即可
- 方法的名称：自定义，不要重载，建议风格如下：
  - 插入数据使用`insert`作为方法名称中的前缀或关键字
  - 删除数据使用`delete`作为方法名称中的前缀或关键字
  - 更新数据使用`update`作为方法名称中的前缀或关键字
  - 查询数据时：
    - 如果是统计，使用`count`作为方法名称中的前缀或关键字
    - 如果是单个数据，使用`get`或`find`作为方法名称中的前缀或关键字
    - 如果是列表，使用`list`作为方法名称中的前缀或关键字
  - 如果操作数据时有条件，可在以上前缀或关键字右侧添加`by字段名`，例如`deleteById`
- 方法的参数列表：取决于需要执行的SQL语句中有哪些参数，如果有多个参数，可将这些参数封装到同一个类型中，使用封装的类型作为方法的参数类型

假设当需要实现“插入一条管理员数据”，则需要执行的SQL语句大致是：

```mysql
insert into ams_admin (username, password, nickname, avatar, phone, email, description, is_enable, last_login_ip, login_count, gmt_last_login, gmt_create, gmt_modified) values (?,?,? ... ?);
```

由于以上SQL语句中的参数数量较多，则应该将它们封装起来，则在`cn.tedu.mybatis`包下创建`Admin`类，声明一系列的属性，对应以上各参数值：

```java
package cn.tedu.mybatis;

import java.time.LocalDateTime;

public class Admin {
    
    private String username;
    private String password;
    private String nickname;
    private String avatar;
    private String phone;
    private String email;
    private String description;
    private Integer isEnable;
    private String lastLoginIp;
    private Integer loginCount;
    private LocalDateTime gmtLastLogin;
    private LocalDateTime gmtCreate;
    private LocalDateTime gmtModified;
    
 	// Setters & Getters
    // toString()
}
```

接下来，在`cn.tedu.mybatis`包下创建`mapper.AdminMapper`接口，并在接口中添加“插入1条管理员数据”的抽象方法：

```java
package cn.tedu.mybatis.mapper;

import cn.tedu.mybatis.Admin;

public interface AdminMapper {

    int insert(Admin admin);

}
```

所有用于Mybatis处理数据的接口都必须被Mybatis识别，有2种做法：

- 在每个接口上添加`@Mapper`注解
- 推荐：在配置类上添加`@MapperScan`注解，指定接口所在的根包

例如，在`SpringConfig`上添加配置`@MapperScan`：

```java
@Configuration
@PropertySource("classpath:datasource.properties")
@MapperScan("cn.tedu.mybatis.mapper")
public class SpringConfig {

    // ... ...
    
}
```

注意：因为Mybatis会扫描以上配置的包，并自动生成包中各接口中的代理对象，所以，千万不要放其它接口文件！

接下来，需要配置抽象方法对应的SQL语句，这些SQL语句推荐配置在XML文件中，可以从 http://doc.canglaoshi.org/config/Mapper.xml.zip 下载到XML文件。在项目的`src/main/resources`下创建`mapper`文件夹，并将下载得到的XML文件复制到此文件夹中，重命名为`AdminMapper.xml`。

打开XML文件夹，进行配置：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- 根节点必须是mapper -->
<!-- 根节点的namespace属性用于配置此XML对应哪个接口 -->
<mapper namespace="cn.tedu.mybatis.mapper.AdminMapper">

    <!-- 根据需要执行的SQL语句的种类选择需要配置的节点名称 -->
    <!-- 配置SQL的节点的id属性取值为抽象方法名称 -->
    <!-- 在节点内部配置SQL语句 -->
    <!-- SQL语句中的参数值使用 #{} 格式的占位符表示 -->
    <insert id="insert">
        insert into ams_admin (
            username, password, nickname, avatar, 
            phone, email, description, is_enable, 
            last_login_ip, login_count, gmt_last_login, gmt_create, 
            gmt_modified
        ) values (
            #{username}, #{password}, #{nickname}, #{avatar}, 
            #{phone}, #{email}, #{description}, #{isEnable}, 
            #{lastLoginIp}, #{loginCount}, #{gmtLastLogin}, #{gmtCreate}, 
            #{gmtModified}
        )
    </insert>

</mapper>
```

最后，还需要将`DataSource`配置给Mybatis框架，并且，为Mybatis配置这些XML文件的路径，这2项配置都将通过配置`SqlSessionFactoryBean`来完成。

先在`datasource.properties`中补充一条配置：

```
mybatis.mapper-locations=classpath:mapper/AdminMapper.xml
```

然后在配置类中创建`SqlSessionFactoryBean`类型的对象：

```java
@Bean
public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource,
        @Value("${mybatis.mapper-locations}") Resource mapperLocations) {
    SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
    sqlSessionFactoryBean.setDataSource(dataSource);
    sqlSessionFactoryBean.setMapperLocations(mapperLocations);
    return sqlSessionFactoryBean;
}
```

最后，在测试类中补充测试方法，以检验是否可以通过调用`AdminMapper`的`insert()`方法插入数据：

```java
@Test
public void testInsert() {
    AnnotationConfigApplicationContext ac
            = new AnnotationConfigApplicationContext(SpringConfig.class);

    AdminMapper adminMapper = ac.getBean(AdminMapper.class);

    Admin admin = new Admin();
    admin.setUsername("admin001");
    admin.setPassword("12345678");
    adminMapper.insert(admin);

    ac.close();
}
```

## 5. 获取新增的数据的自动编号的id

如果某数据的id是自动编号，当需要获取新增的数据的id时，需要先使得插入的数据类型中有id对应的属性，则在`Admin`类中添加`id`属性：

```java
public class Admin {

    private Long id;
    // 原有其它属性及Setter & Getter
    
    // 补充id的Setter & Getter
    // 重新生成toString()
    
}
```

接下来，在`<insert>`节点配置2个属性，分别是`useGeneratedKeys`和`keyProperty`：

```xml
<insert id="insert" useGeneratedKeys="true"  keyProperty="id">
    原有代码
</insert>
```

当配置完成后，Mybatis执行此插入数据的操作后，会将自动编号的id赋值到参数`Admin admin`的`id`属性中，以上`keyProperty`指的就是将自动编号的值放回到参数对象的哪个属性中！

## 6. 删除数据

目标：根据id删除某一条数据

要实现此目标，需要执行的SQL语句大致是：

```mysql
delete from ams_admin where id=?
```

然后，在`AdminMapper`接口中添加抽象方法：

```java
int deleteById(Long id);
```

接下来，在`AdminMapper.xml`中配置以上抽象方法映射的SQL语句：

```xml
<delete id="deleteById">
    delete from ams_admin where id=#{id}
</delete>
```

最后，编写并执行测试：

```java
@Test
public void testDeleteById() {
    AnnotationConfigApplicationContext ac
            = new AnnotationConfigApplicationContext(SpringConfig.class);

    AdminMapper adminMapper = ac.getBean(AdminMapper.class);

    Long id = 12L;
    int rows = adminMapper.deleteById(id);
    // System.out.println("删除完成，受影响的行数=" + rows);
    if (rows == 1) {
        System.out.println("删除成功");
    } else {
        System.out.println("删除失败，尝试删除的数据（id=" + id + "）不存在！");
    }

    ac.close();
}
```

## 7. 修改数据

目标：根据id修改某一条数据的密码

要实现此目标，需要执行的SQL语句大致是：

```mysql
update ams_admin set password=? where id=?
```

然后，在`AdminMapper`接口中添加抽象方法：

```java
int updatePasswordById(@Param("id") Long id, 
            @Param("password") String password);
```

接下来，在`AdminMapper.xml`中配置以上抽象方法映射的SQL语句：

```xml
<update id="updatePasswordById">
    update ams_admin set password=#{password} where id=#{id}
</update>
```

最后，编写并执行测试：

```java
@Test
public void testUpdatePasswordById() {
    AnnotationConfigApplicationContext ac
            = new AnnotationConfigApplicationContext(SpringConfig.class);

    AdminMapper adminMapper = ac.getBean(AdminMapper.class);

    Long id = 12L;
    String password = "000000";
    int rows = adminMapper.updatePasswordById(id, password);
    if (rows == 1) {
        System.out.println("修改密码成功");
    } else {
        System.out.println("修改密码失败，尝试访问的数据（id=" + id + "）不存在！");
    }

    ac.close();
}
```

## 8. 查询数据-1

目标：统计当前表中有多少条数据

要实现此目标，需要执行的SQL语句大致是：

```mysql
select count(*) from ams_admin
```

然后，在`AdminMapper`接口中添加抽象方法：

```java
int count();
```

接下来，在`AdminMapper.xml`中配置以上抽象方法映射的SQL语句：

```xml
<!-- int count(); -->
<!-- 所有select节点必须配置resultType或resultMap这2个属性中的其中1个 -->
<select id="count" resultType="int">
    select count(*) from ams_admin
</select>
```

最后，编写并执行测试：

```java
@Test
public void testCount() {
    AnnotationConfigApplicationContext ac
            = new AnnotationConfigApplicationContext(SpringConfig.class);

    AdminMapper adminMapper = ac.getBean(AdminMapper.class);

    int count = adminMapper.count();
    System.out.println("当前表中有" + count + "条记录");

    ac.close();
}
```

## 9. 查询数据-2

目标：根据id查询管理员信息

要实现此目标，需要执行的SQL语句大致是：

```mysql
select * from ams_admin where id=?
```

然后，在`AdminMapper`接口中添加抽象方法：

```java
Admin getById(Long id);
```

接下来，在`AdminMapper.xml`中配置以上抽象方法映射的SQL语句：

```xml
<!-- Admin getById(Long id); -->
<select id="getById" resultType="cn.tedu.mybatis.Admin">
    select * from ams_admin where id=#{id}
</select>
```

最后，编写并执行测试：

```java
@Test
public void testGetById() {
    AnnotationConfigApplicationContext ac
            = new AnnotationConfigApplicationContext(SpringConfig.class);

    AdminMapper adminMapper = ac.getBean(AdminMapper.class);

    Long id = 3L;
    Admin admin = adminMapper.getById(id);
    System.out.println("查询结果：" + admin);

    ac.close();
}
```

通过测试可以发现：当存在匹配的数据时，将可以查询到数据，当不存在匹配的数据时，将返回`null`。

需要注意，如果查询结果集中的列名与类的属性名不匹配时，默认将放弃处理这些结果数据，则返回的对象中对应的属性值为`null`，为了解决此问题，可以在查询时使用自定义的别名，使得名称保持一致，不过，更推荐配置`<resultMap>`以指导Mybatis封装查询结果，例如：

```xml
<!-- Admin getById(Long id); -->
<select id="getById" resultMap="BaseResultMap">
    select * from ams_admin where id=#{id}
</select>

<!-- resultMap节点的作用是：指导Mybatis如何将结果集中的数据封装到返回的对象中 -->
<!-- id属性：自定义名称 -->
<!-- type属性：将结果集封装到哪种类型的对象中 -->
<resultMap id="BaseResultMap" type="cn.tedu.mybatis.Admin">
    <!-- 使用若干个result节点配置名称不统一的对应关系 -->
    <!-- 在不是“一对多”的查询时，名称本来就一致的是不需要配置的 -->
    <!-- column属性：列名 -->
    <!-- property属性：属性名 -->
    <result column="is_enable" property="isEnable" />
    <result column="last_login_ip" property="lastLoginIp" />
    <result column="login_count" property="loginCount" />
    <result column="gmt_last_login" property="gmtLastLogin" />
    <result column="gmt_create" property="gmtCreate" />
    <result column="gmt_modified" property="gmtModified" />
</resultMap>
```

注意：在后续的应用中，凡是可以通过`resultMap`处理结果的，都不要使用`resultType`。

## 10. 查询数据-3

目标：查询所有管理员的信息

要实现此目标，需要执行的SQL语句大致是：

```mysql
select * from ams_admin order by id
```

**注意：(1) 查询时，结果集中可能超过1条数据时，必须显式的使用`ORDER BY`子句对结果集进行排序；(2) 查询时，结果集中可能超过1条数据时，应该考虑是否需要分页。**

然后，在`AdminMapper`接口中添加抽象方法：

```java
List<Admin> list();
```

接下来，在`AdminMapper.xml`中配置以上抽象方法映射的SQL语句：

```xml
<!-- List<Admin> list(); -->
<select id="list" resultMap="BaseResultMap">
    select * from ams_admin order by id
</select>
```

最后，编写并执行测试：

```java
@Test
public void testList() {
    AnnotationConfigApplicationContext ac
            = new AnnotationConfigApplicationContext(SpringConfig.class);

    AdminMapper adminMapper = ac.getBean(AdminMapper.class);

    List<Admin> list = adminMapper.list();
    for (Admin admin : list) {
        System.out.println(admin);
    }

    ac.close();
}
```

## 11. 动态SQL -- foreach

Mybatis中的动态SQL表现为：根据参数不同，生成不同的SQL语句

目标：根据若干个id一次性删除若干条管理数据

要实现此目标，需要执行的SQL语句大致是：

```mysql
delete from ams_admin where id in (?,?)
```

以上SQL语句中，id值的数量（以上`?`的数量）对于开发人员而言是未知的！

然后，在`AdminMapper`接口中添加抽象方法：

```java
int deleteByIds(Long... ids);
```

或

```java
int deleteByIds(Long[] ids);
```

或

```java
int deleteByIds(List<Long> ids);
```

接下来，在`AdminMapper.xml`中配置以上抽象方法映射的SQL语句：

```xml
<!-- int deleteByIds(List<Long> ids); -->
<delete id="deleteByIds">
    delete from ams_admin where id in (
    	<foreach collection="list" item="id" separator=",">
          #{id}
    	</foreach>
    )
</delete>
```

以上代码中：

- `<foreach>`标签：用于遍历集合或数组类型的参数对象
- `collection`属性：被遍历的参数对象，当抽象方法的参数只有1个且没有添加`@Param`注解时，如果参数是`List`类型则此属性值为`list`，如果参数是数组类型（包括可变参数）则此属性值为`array`；当抽象方法的参数有多个或添加了`@Param`注解时，则此属性值为`@Param`注解中配置的值
- `item`属性：自定义的名称，表示遍历过程中每个元素的变量名，可在`<foreach>`子级使用`#{变量名}`表示数据
- `separator`属性：分隔符号，会自动添加在遍历到的各元素之间

最后，编写并执行测试：

```java
@Test
public void testDeleteByIds() {
    AnnotationConfigApplicationContext ac
            = new AnnotationConfigApplicationContext(SpringConfig.class);

    AdminMapper adminMapper = ac.getBean(AdminMapper.class);

    List<Long> ids = new ArrayList<>();
    ids.add(16L);
    ids.add(18L);
    ids.add(19L);
    int rows = adminMapper.deleteByIds(ids);
    System.out.println("受影响的行数为：" + rows);

    ac.close();
}
```

## 12. 动态SQL -- 其它

在Mybatis中动态SQL还有其它节点，例如：`<if>` / `<choose>` + `<when>` + `<otherwise>`等，将在项目中补充。

## 13. 关于查询时的字段列表

在阿里巴巴的《Java开发手册》中指出：

> **【强制】在表查询中，一律不要使用 * 作为查询的字段列表，需要哪些字段必须明确写明。**

通常，建议将字段列表使用`<sql>`节点进行封装，例如：

```xml
<sql id="BaseQueryFields">
    <if test="true">
        id, username, password, nickname, avatar, phone, email, description, is_enable, last_login_ip, login_count, gmt_last_login, gmt_create, gmt_modified
    </if>
</sql>
```

提示：为避免IntelliJ IDEA误以为以上代码片段是错误的而提示红色的波浪线，所以使用`<if test="true">`框住了字段列表的代码片段，但这个`<if>`并不是必须的，即使提示了红色的波浪线，也不影响运行。

注意：以上`<Sql>`节点可以用于封装任何SQL语句的任何片段，不仅仅只是字段列表。

封装后，当需要引用以上代码片段时，可以使用`<include>`节点进行引用，例如：

```xml
<!-- Admin getById(Long id); -->
<select id="getById" resultMap="BaseResultMap">
    select
        <include refid="BaseQueryFields" />
    from ams_admin where id=#{id}
</select>

<!-- List<Admin> list(); -->
<select id="list" resultMap="BaseResultMap">
    select
        <include refid="BaseQueryFields" />
    from ams_admin order by id
</select>
```

## 14. 基于Spring的测试

在`pom.xml`中添加`spring-test`依赖项：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.3.14</version>
</dependency>
```

**注意：与其它的`spring-????`使用完全相同的版本！**

接下来，在编写测试时，就可以在测试类上添加`@SpringJUnitConfig`注解，并在注解中配置Spring的配置类作为参数，则执行此类的任何测试方法之前，都会加载这些Spring配置类，并且，在编写测试时，只要是在Spring容器中存在的对象，都可以自动装配，例如：

```java
@SpringJUnitConfig(SpringConfig.class)
public class MybatisSpringTests {

    @Autowired
    Environment env;

    @Test
    public void contextLoads() {
        System.out.println(env.getProperty("datasource.url"));
        System.out.println(env.getProperty("datasource.driver"));
        System.out.println(env.getProperty("datasource.username"));
        System.out.println(env.getProperty("datasource.password"));
    }
    
}
```

## 15. 关于`@Sql`注解

当添加了`spring-test`依赖后，可以在测试时使用`@Sql`注解，以加载某些`.sql`脚本，使得测试之前或之后将执行这些脚本！

使用此注解主要是为了保障可以反复测试，并且得到预期的结果！例如执行删除的测试时，假设数据是存在的，第1次删除可以成功，但是在这之后的测试将不会成功，因为数据在第1次测试时就已经被删除！则可以编写一个`.sql`脚本，通过脚本向数据表中插入数据，并在每次测试之前执行此脚本，即可保证每次测试都是成功的！

此注解可以添加在测试类上，则对当前测试类的每个测试方法都是有效的。

此注解也可以添加在测试方法上，则只对当前测试方法是有效的。

如果测试类和测试方法上都添加了此注解，则仅测试方法上的注解会生效。

此注解除了配置需要执行的`.sql`脚本以外，还可以通过`executionPhase`属性配置其执行阶段，例如取值为`Sql.ExecutionPhase.AFTER_TEST_METHOD`时将使得`.sql`脚本会在测试方法之后被执行。

每个测试方法可以添加多个`@Sql`注解。

例如：

```java
@Test
@Sql(scripts = {"classpath:truncate.sql", "classpath:insert_data.sql"})
@Sql(scripts = {"classpath:truncate.sql"}, 
     executionPhase = Sql.ExecutionPhase.AFTER_TEST_METHOD)
public void testDeleteByIdSuccessfully() {
    Long id = 1L;
    int rows = adminMapper.deleteById(id);
    Assertions.assertEquals(1, rows);
}
```

`insert_data.sql`脚本示例：

```mysql
insert into ams_admin (username, password) values ('admin001', '123456');
insert into ams_admin (username, password) values ('admin002', '123456');
insert into ams_admin (username, password) values ('admin003', '123456');
insert into ams_admin (username, password) values ('admin004', '123456');
insert into ams_admin (username, password) values ('admin005', '123456');
```

`truncate.sql`脚本示例：

```mysql
truncate ams_admin;
```

## 16. 关于测试中的断言

在执行测试时，应该使用**断言**对测试结果进行预判，而不是使用输出语句结合肉眼观察结果，这样才更符合自动化测试的标准（在自动化测试中，可以一键执行项目中的所有测试方法，并将测试结果汇总到专门的测试报告文件中）。

通过调用`Assertions`类中的静态方法可以对结果进行断言，常用方法有：

- `assertEquals()`：断言匹配（相等）
- `assertNotEquals()`：断言不匹配（不相等）
- `assertTrue()`：断言为“真”
- `assertFalse()`：断言为“假”
- `assertNull()`：断言为`null`
- `assertNotNull()`：断言不为`null`
- `assertThrows()`：断言将抛出异常
- `assertDoesNotThrow()`：断言不会抛出异常
- 其它

## 17. 关于`#{}`和`${}`格式的占位符

在Mybatis中，配置SQL语句时，参数可以使用`#{}`或`${}`格式的占位符。

例如存在需求：分页查询表中的所有数据。

需要执行的SQL语句大致是：

```mysql
select * from ams_admin order by id limit ?, ?
```

则此功能的抽象方法应该是：

```java
List<Admin> listPage(@Param("offset") Integer offset, @Param("size") Integer size);
```

配置SQL语句：

```xml
<!-- List<Admin> listPage(@Param("offset") Integer offset, 
								@Param("size") Integer size); -->
<select id="listPage" resultMap="BaseResultMap">
    select 
    	<include refid="BaseQueryFields" />
    from ams_admin 
    order by id 
    limit #{offset}, #{size}
</select>
```

最后，执行测试：

```java
@Test
@Sql(scripts = {"classpath:truncate.sql", "classpath:insert_data.sql"})
@Sql(scripts = {"classpath:truncate.sql"}, 
     executionPhase = Sql.ExecutionPhase.AFTER_TEST_METHOD)
public void testListPage() {
    // 准备测试数据
    Integer offset = 0;
    Integer size = 3;
    // 断言不会抛出异常
    Assertions.assertDoesNotThrow(() -> {
        // 执行测试
    	List<Admin> adminList = adminMapper.listPage(offset, size);
        // 观察结果（通过输出语句）
        System.out.println("查询到的记录数：" + adminList.size());
        for (Admin admin : adminList) {
            System.out.println(admin);
        }
    });
}
```

以上代码可以正常通过测试，并且观察结果也都是符合预期的，即使把SQL语句中的`#{}`换成`${}`格式，也是完全没有问题的！

例如还存在需求：根据用户名查询此用户的详情。

在“根据用户名查询用户详情”时，如果将`username=#{username}`换成`username=${username}`会出现错误！

**其实，使用`#{}`格式的占位符时，Mybatis在处理时会使用预编译的做法，所以，在编写SQL语句时不必关心数据类型的问题（例如字符串值不需要添加单引号），也不存在SQL注入的风险！这种占位符只能用于表示某个值，而不能表示SQL语句片段！**

**当使用`${}`格式的占位符时，Mybatis在处理时会先将参数值代入到SQL语句中，然后再执行编译相关过程，所以需要关心某些值的数据类型问题（例如涉及字符串值时，需要在编写SQL语句时添加一对单引号框住字符串），并且，存在SQL注入的风险！其优点是可以表示SQL语句中的任何片段！**

**在一般情况下，应该尽可能的使用`#{}`格式的占位符，并不推荐使用`${}`格式的占位符，即使它可以实现“泛用”的效果！在一些特殊的情况下，如果一定要使用`${}`格式的占位符，必须考虑SQL注入的风险，应该使用正则表达式或其它做法避免出现SQL注入问题！**

## 18. 关于RBAC

**RBAC** = **R**ole **B**ased **A**ccess **C**ontrol（基于角色的访问控制）

RBAC是经典的用户权限管理的设计思路。在这样的设计中，会存在3种类型：用户、角色、权限，权限将分配到各种角色上，用户可以关联某种角色，进而实现用户与权限相关。使用这样的设计，更加利于统一管理若干个用户的权限。

在RBAC的设计思路中，用户与角色一般是多对多的关系，而在数据库中，仅仅只是使用“用户”和“角色”这2张表是不利于维护多对多关系的，通常会增加一张中间表，专门记录对应关系，同理，角色和权限也是多对多的关系，也需要使用中间表来记录对应关系！

关于这些表的设计参考如下：

**ams_admin：管理员表**

```mysql
-- 管理员表：创建数据表
drop table if exists ams_admin;
create table ams_admin (
    id bigint unsigned auto_increment,
    username varchar(50) default null unique comment '用户名',
    password char(64) default null comment '密码（密文）',
    nickname varchar(50) default null comment '昵称',
    avatar varchar(255) default null comment '头像URL',
    phone varchar(50) default null unique comment '手机号码',
    email varchar(50) default null unique comment '电子邮箱',
    description varchar(255) default null comment '描述',
    is_enable tinyint unsigned default 0 comment '是否启用，1=启用，0=未启用',
    last_login_ip varchar(50) default null comment '最后登录IP地址（冗余）',
    login_count int unsigned default 0 comment '累计登录次数（冗余）',
    gmt_last_login datetime default null comment '最后登录时间（冗余）',
    gmt_create datetime default null comment '数据创建时间',
    gmt_modified datetime default null comment '数据最后修改时间',
    primary key (id)
) comment '管理员表' charset utf8mb4;

-- 管理员表：插入测试数据
insert into ams_admin (username, password, nickname, email, description, is_enable) values
    ('root', '1234', 'root', 'root@tedu.cn', '最高管理员', 1),
    ('super_admin', '1234', 'administrator', 'admin@tedu.cn', '超级管理员', 1),
    ('nobody', '1234', '无名', 'liucs@tedu.cn', null, 0);
```

**ams_role：角色表**

```mysql
-- 角色表：创建数据表
drop table if exists ams_role;
create table ams_role (
    id bigint unsigned auto_increment,
    name varchar(50) default null comment '名称',
    description varchar(255) default null comment '描述',
    sort tinyint unsigned default 0 comment '自定义排序序号',
    gmt_create datetime default null comment '数据创建时间',
    gmt_modified datetime default null comment '数据最后修改时间',
    primary key (id)
) comment '角色表' charset utf8mb4;

-- 角色表：插入测试数据
insert into ams_role (name) values
    ('超级管理员'), ('系统管理员'), ('商品管理员'), ('订单管理员');
```

**ams_admin_role：管理员与角色的关联表**

```mysql
-- 管理员角色关联表：创建数据表
drop table if exists ams_admin_role;
create table ams_admin_role (
    id bigint unsigned auto_increment,
    admin_id bigint unsigned default null comment '管理员id',
    role_id bigint unsigned default null comment '角色id',
    gmt_create datetime default null comment '数据创建时间',
    gmt_modified datetime default null comment '数据最后修改时间',
    primary key (id)
) comment '管理员角色关联表' charset utf8mb4;

-- 管理员角色关联表：插入测试数据
insert into ams_admin_role (admin_id, role_id) values
    (1, 1), (1, 2), (1, 3), (2, 2), (2, 3), (2, 4), (3, 3);
```

**ams_permission：权限表**

```mysql
-- 权限表：创建数据表
drop table if exists ams_permission;
create table ams_permission (
    id bigint unsigned auto_increment,
    name varchar(50) default null comment '名称',
    value varchar(255) default null comment '值',
    description varchar(255) default null comment '描述',
    sort tinyint unsigned default 0 comment '自定义排序序号',
    gmt_create datetime default null comment '数据创建时间',
    gmt_modified datetime default null comment '数据最后修改时间',
    primary key (id)
) comment '权限' charset utf8mb4;

-- 权限表：插入测试数据
insert into ams_permission (name, value, description) values
('商品-商品管理-读取', '/pms/product/read', '读取商品数据，含列表、详情、查询等'),
('商品-商品管理-编辑', '/pms/product/update', '修改商品数据'),
('商品-商品管理-删除', '/pms/product/delete', '删除商品数据'),
('后台管理-管理员-读取', '/ams/admin/read', '读取管理员数据，含列表、详情、查询等'),
('后台管理-管理员-编辑', '/ams/admin/update', '编辑管理员数据'),
('后台管理-管理员-删除', '/ams/admin/delete', '删除管理员数据');
```

**ams_role_permission：角色与权限的关联表**

```mysql
-- 角色权限关联表：创建数据表
drop table if exists ams_role_permission;
create table ams_role_permission (
    id bigint unsigned auto_increment,
    role_id bigint unsigned default null comment '角色id',
    permission_id bigint unsigned default null comment '权限id',
    gmt_create datetime default null comment '数据创建时间',
    gmt_modified datetime default null comment '数据最后修改时间',
    primary key (id)
) comment '角色权限关联表' charset utf8mb4;

-- 角色权限关联表：插入测试数据
insert into ams_role_permission (role_id, permission_id) values
    (1, 1), (1, 2), (1, 3), (1, 4), (1, 5), (1, 6),
    (2, 1), (2, 2), (2, 3), (2, 4), (2, 5), (2, 6),
    (3, 1), (3, 2), (3, 3);
```

在`mall_ams`数据库中创建以上数据表，并插入以上测试数据。

练习（单表数据访问）：

- 向权限表中插入新的数据
- 根据id删除某个权限
- 查询权限表中的所有权限

提示：

- 需要新的数据类型，例如`Permission`类
- 需要新的接口文件，用于定义以上2个数据访问功能的抽象方法
- 需要新的XML文件，用于配置抽象方法对应的SQL语句
- 需要修改配置信息，将此前指定的XML文件由`AdminMapper.xml`改为`*.xml`，并把`SpringConfig`类中`sqlSessionFactoryBean()`方法的第2个参数由`Resource`类型改为`Resource...`类型
- 当需要测试时，使用新的测试类

## 19. 关于关联查询

首先，请准备一些测试数据，使得：存在若干条用户数据，存在若干条角色数据，某个用户存在与角色的关联，最好有些用户有多个关联，又有些用户只有1个关联，还有些用户没有关联。

假设存在需求：根据id查询某用户信息时，也查出该用户归属于哪几种角色。

**测试数据参考：**

```mysql
truncate ams_admin;
truncate ams_admin_role;
truncate ams_role;
truncate ams_permission;

insert into ams_admin (username, password) values ('admin001', '123456');
insert into ams_admin (username, password) values ('admin002', '123456');
insert into ams_admin (username, password) values ('admin003', '123456');
insert into ams_admin (username, password) values ('admin004', '123456');
insert into ams_admin (username, password) values ('admin005', '123456');
insert into ams_admin (username, password) values ('admin006', '123456');
insert into ams_admin (username, password) values ('admin007', '123456');
insert into ams_admin (username, password) values ('admin008', '123456');
insert into ams_admin (username, password) values ('admin009', '123456');
insert into ams_admin (username, password) values ('admin010', '123456');
insert into ams_admin (username, password) values ('admin011', '123456');
insert into ams_admin (username, password) values ('admin012', '123456');
insert into ams_admin (username, password) values ('admin013', '123456');
insert into ams_admin (username, password) values ('admin014', '123456');
insert into ams_admin (username, password) values ('admin015', '123456');
insert into ams_admin (username, password) values ('admin016', '123456');
insert into ams_admin (username, password) values ('admin017', '123456');
insert into ams_admin (username, password) values ('admin018', '123456');
insert into ams_admin (username, password) values ('admin019', '123456');
insert into ams_admin (username, password) values ('admin020', '123456');

insert into ams_permission (name, value, description) values
('商品-商品管理-读取', '/pms/product/read', '读取商品数据，含列表、详情、查询等'),
('商品-商品管理-编辑', '/pms/product/update', '修改商品数据'),
('商品-商品管理-删除', '/pms/product/delete', '删除商品数据'),
('后台管理-管理员-读取', '/ams/admin/read', '读取管理员数据，含列表、详情、查询等'),
('后台管理-管理员-编辑', '/ams/admin/update', '编辑管理员数据'),
('后台管理-管理员-删除', '/ams/admin/delete', '删除管理员数据');

insert into ams_role (name) values
('超级管理员'), ('系统管理员'), ('商品管理员'), ('订单管理员');

insert into ams_admin_role (admin_id, role_id) values
(1, 1), (1, 2), (1, 3), (1, 4),
(2, 1), (2, 2), (2, 3),
(3, 1), (3, 2),
(4, 1);
```

本次查询需要执行的SQL语句大致是：

```mysql
select *
from ams_admin
left join ams_admin_role on ams_admin.id=ams_admin_role.admin_id
left join ams_role on ams_admin_role.role_id=ams_role.id
where ams_admin.id=?
```

通过测试运行，可以发现（必须基于以上测试数据）：

- 当使用的id值为1时，共查询到4条记录，并且用户的基本信息是相同的，只是与角色关联的数据不同
- 当使用的id值为2时，共查询到3条记录
- 当使用的id值为3时，共查询到2条记录
- 当使用其它有效用户的id时，共查询到1条记录

其实，这种查询期望的结果应该是：

```java
public class xxx {
    // 用户基本信息的若干个属性，例如用户名、密码等
    // 此用户的若干个角色数据，可以使用 List<?>
}
```

则可以先创建“角色”对应的数据类型：

```java
public class Role {
    private Long id;
    private String name;
    private String description;
    private Integer sort;
    private LocalDateTime gmtCreate;
    private LocalDateTime gmtModified;
    // Setters & Getterss
    // toString()
}
```

再创建用于封装此次查询结果的类型：

```java
public class AdminDetailsVO {
    private Long id;
    private String username;
    private String password;
    private String nickname;
    private String avatar;
    private String phone;
    private String email;
    private String description;
    private Integer isEnable;
    private String lastLoginIp;
    private Integer loginCount;
    private LocalDateTime gmtLastLogin;
    private LocalDateTime gmtCreate;
    private LocalDateTime gmtModified;
    private List<Role> roles;
    // Setters & Getterss
    // toString()
}
```

接下来，可以在`AdminMapper`接口中添加抽象方法：

```java
AdminDetailsVO getDetailsById(Long id);
```

需要注意，由于此次关联了3张表一起查询，结果集中必然出现某些列的名称是完全相同的，所以，在查询时，不可以使用星号表示字段列表（因为这样的结果集中的列名就是字段名，会出现相同的列名），而是应该至少为其中的一部分相同名称的列定义别名，例如：

```mysql
select
    ams_admin.id,
       ams_admin.username,
       ams_admin.password,
       ams_admin.nickname,
       ams_admin.avatar,
       ams_admin.phone,
       ams_admin.email,
       ams_admin.description,
       ams_admin.is_enable,
       ams_admin.last_login_ip,
       ams_admin.login_count,
       ams_admin.gmt_last_login,
       ams_admin.gmt_create,
       ams_admin.gmt_modified,

       ams_role.id AS role_id,
       ams_role.name AS role_name,
       ams_role.description AS role_description,
       ams_role.sort AS role_sort,
       ams_role.gmt_create AS role_gmt_create,
       ams_role.gmt_modified AS role_gmt_modified
from ams_admin
left join ams_admin_role on ams_admin.id=ams_admin_role.admin_id
left join ams_role on ams_admin_role.role_id=ams_role.id
where ams_admin.id=1;
```

在Mybatis处理中此查询时，并不会那么智能的完成结果集的封装，所以，必须自行配置`<resultMap>`用于指导Mybatis完成封装！

```xml
<resultMap id="DetailsResultMap" type="xx.xx.xx.xx.AdminDetailsVO">
    <!-- 在1对多、多对多的查询中，即使名称匹配的结果，也必须显式的配置 -->
    <!-- 主键字段的结果必须使用id节点进行配置，配置方式与result节点完全相同 -->
    <id column="id" property="id" />
	<result column="gmt_create" property="gmtCreate" />
    <!-- 需要使用collection节点配置1对多中“多”的数据 -->
    <collection property="roles" ofType="xx.xx.xx.Role">
        <id column="role_id" property="id" />
        <result column="gmt_create" property="gmtCreate" />
    </collection>
</resultMap>
```

最后，使用以上的查询SQL语句，并使用以上的`<resultMap>`封装结果即可！

```xml
<sql id="DetailsQueryFields">
    <if test="true">
       ams_admin.id,
       ams_admin.username,
       ams_admin.password,
       ams_admin.nickname,
       ams_admin.avatar,
       ams_admin.phone,
       ams_admin.email,
       ams_admin.description,
       ams_admin.is_enable,
       ams_admin.last_login_ip,
       ams_admin.login_count,
       ams_admin.gmt_last_login,
       ams_admin.gmt_create,
       ams_admin.gmt_modified,

       ams_role.id AS role_id,
       ams_role.name AS role_name,
       ams_role.description AS role_description,
       ams_role.sort AS role_sort,
       ams_role.gmt_create AS role_gmt_create,
       ams_role.gmt_modified AS role_gmt_modified
    </if>
</sql>

<resultMap id="DetailsResultMap" type="cn.tedu.mybatis.AdminDetailsVO">
    <!-- 在1对多、多对多的查询中，即使名称匹配的结果，也必须显式的配置 -->
    <!-- 主键字段的结果必须使用id节点进行配置，配置方式与result节点完全相同 -->
    <id column="id" property="id" />
    <result column="username" property="username" />
    <result column="password" property="password" />
    <result column="nickname" property="nickname" />
    <result column="avatar" property="avatar" />
    <result column="phone" property="phone" />
    <result column="email" property="email" />
    <result column="description" property="description" />
    <result column="is_enable" property="isEnable" />
    <result column="last_login_ip" property="lastLoginIp" />
    <result column="login_count" property="loginCount" />
    <result column="gmt_last_login" property="gmtLastLogin" />
    <result column="gmt_create" property="gmtCreate" />
    <result column="gmt_modified" property="gmtModified" />
    <!-- 需要使用collection节点配置1对多中“多”的数据 -->
    <collection property="roles" ofType="cn.tedu.mybatis.Role">
        <id column="role_id" property="id" />
        <result column="role_name" property="name" />
        <result column="role_description" property="description" />
        <result column="role_sort" property="sort" />
        <result column="role_gmt_create" property="gmtCreate" />
        <result column="role_gmt_modified" property="gmtModified" />
    </collection>
</resultMap>

<select id="getDetailsById" resultMap="DetailsResultMap">
    select <include refid="DetailsQueryFields" />
    from ams_admin
    left join ams_admin_role on ams_admin.id=ams_admin_role.admin_id
    left join ams_role on ams_admin_role.role_id=ams_role.id
    where ams_admin.id=#{id}
</select>
```

## 20. Mybatis的缓存机制

缓存：通常是一个临时存储的数据，在未来的某个时间点可能会被删除，通常，存储缓存数据的位置通常是读写效率较高的，相比其它“非缓存”的数据有更高的处理效率。由于缓存的数据通常并不是必须的，则需要额外消耗一定的存储空间，同时由于从缓存获取数据的效率更高，所以是一种牺牲空间、换取时间的做法！另外，你必须知道，从数据库读取数据的效率是非常低下的！

Mybatis有2种缓存机制，分别称之一级缓存和二级缓存，其中，一级缓存是基于`SqlSession`的缓存，也称之为“会话缓存”，仅当是同一个会话、同一个Mapper、同一个抽象方法（同一个SQL语句）、同样的参数值时有效，一级缓存在集成框架的应用中默认是开启的，且整个过程不由人为控制（如果是自行得到`SqlSession`后的操作，可自行清理一级缓存），另外，二级缓存默认是全局开启的，它是基于namespace的，所以也称之为“namespace缓存”，需要在配置SQL语句的XML中添加`<cache />`节点，以表示当前XML中的所有查询都允许开通二级缓存，并且，在`<select>`节点上配置`useCache="true"`，则对应的`<select>`节点的查询结果将被二级缓存处理，并且，此查询返回的结果的类型必须是实现了`Serializable`接口的，如果使用了`<resultMap>`配置如何封装查询结果，则必须使用`<id>`节点来封装主键的映射，满足以上条件后，二级缓存将可用，只要是当前namespace中查询出来的结果，都会根据所执行的SQL语句及参数进行结果的缓存。无论是一级缓存还是二级缓存，只要数据发生了写操作（增、删、改），缓存数据都将被自动清理。

由于Mybatis的缓存清理机制过于死板，所以，一般在开发实践中并不怎么使用！更多的是使用其它的缓存工具并自行制定缓存策略。

## 21. 关于Mybatis小结

关于Mybatis框架，你应该：

- 了解如何创建一个整合了Spring框架的Mybatis工程
- 了解整合了Spring框架的Mybatis工程的配置
- 掌握基于`spring-test`的测试
  - 在测试类上使用`@SpringJUnitConfig`注解加载Spring的配置文件，则在当前测试中可以自动装配Spring容器中的任何对象
- 掌握`@Sql`注解的使用，掌握断言的使用
- 掌握声明抽象方法的原则：
  - 返回值类型：增删改类型的操作均返回`int`，表示“受影响的行数”，查询类型操作，根据操作得到的结果集来决定，只要能够放入所有所需的数据即可，通常，统计查询返回`int`，查询最多1个结果时返回自定义的某个数据类型，查询多个结果时返回`List`集合类型
  - 方法名称：自定义，不要使用重载，其它命名建议参考此前的笔记
  - 参数列表：根据需要执行的SQL语句中的参数来设计，并且，当需要执行的是插入数据操作时，必须将这些参数进行封装，并在封装的类中添加主键属性，以便于Mybatis能获取自动编号的值回填到此主键属性中，当需要执行的是其它类型的操作时，如果参数数量较多，可以封装，如果只有1个，则直接声明为方法参数，如果超过1个且数量不多，则每个参数之前添加`@Param`注解
- 了解使用注解配置SQL语句
- 掌握使用XML配置SQL语句
  - 这类XML文件需要顶部特殊的声明，所以，通常是从网上下载或通过复制粘贴得到此类文件
  - 根节点必须是`<mapper>`，且必须配置`namespace`，取值为对应的Java接口的全限定名
  - 应该根据要执行的SQL语句不同来选择`<insert>`、`<delete>`、`<update>`、`<select>`节点，这些节点都必须配置`id`属性，取值为对应的抽象方法的名称
    - 其实，`<delete>`节点和`<update>`节点可以随意替换使用，但不推荐
    - 在不考虑“获取自动生成的主键值”的情况下，`<insert>`和`<delete>`、`<update>`也可以混为一谈，但不推荐
    - 当插入数据时，当需要获取自动生成的主键值时，需要在`<insert>`节点上配置`useGeneratedKeys`和`keyProperty`属性
    - 在`<select>`节点上，必须配置`resultMap`或`resultType`属性中的其中1个
- 掌握使用`<sql>`封装SQL语句片段，并使用`<include>`进行引用，以实现SQL语句的复用
- 掌握`<resultMap>`的配置方式
  - 主键列与属性的映射必须使用`<id>`节点配置
  - 在1对多、多对多的查询中，集合类型的属性的映射必须使用`<collection>`子节点配置
  - 其它列与属性的映射使用`<result>`节点配置
  - 在单表查询中，列与属性名一致时，可以不必显式的配置，但是，在关联查询中，即使列与属性名称一致，也必须显式的配置出来
- 理解`resultType`与`resultMap`使用原则
  - 尽可能的全部使用`resultMap`，如果查询结果是单一某个数据类型（例如基本数据类型或字符串或某个时间等），则使用`resultType`
- 掌握动态SQL中的`<foreach>`的使用
- 大概了解动态SQL中的其它标签
  - 在后续项目中补充
- 理解`#{}`和`${}`的区别
- 了解Mybatis中的一级缓存和二级缓存
  - 有专门的视频讲解















 



