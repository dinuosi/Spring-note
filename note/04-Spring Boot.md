# Spring Boot

## 1. 关于Spring Boot

Spring Boot框架主要解决了创建工程后需要进行繁琐的配置的问题，是一个“开箱即用”的框架，其核心思想是“约定大于配置”。

## 2. 创建Spring Boot工程

使用IntelliJ IDEA的创建向导中的`Spring Initializer`即可创建Spring Boot工程。

在创建时，如果 https://start.spring.io 无响应，可尝试替换为 https://start.springboot.io。

在创建过程中，需要填写并关注的几项有：

- Group Id：组Id，通常是公司的域名倒序排列的结果，例如`cn.tedu`
- Artifact Id：坐标Id，应该是此工程的名称，如果名称中有多个单词，应该使用减号分隔，例如`boot-demo`
- Java Version：使用到的Java版本，目前推荐选择`8`
- Package：项目的根包，默认是由以上填写的`Group Id`和`Artifact Id`组成

**注意：如果`Artifact Id`中使用减号分隔了多个单词，在`Package`中默认并没有分开，通常建议手动添加小数点（`.`）进行分隔**

**注意：此处`Package`决定了默认的组件扫描，所以，在后续开发代码时，所有的组件类都必须放在此包或其子孙包下，在开发实践中，其实会把所有创建的类、接口都放在此包或其子孙包下，不是组件的类不添加组件即可**

**注意：当工程已经创建出来后，不要修改包的名称，除非你已经掌握了解决方案！**

在添加依赖项时，首先需要注意的就是Spring Boot的版本号，通常非常不建议使用较新的版本号，建议使用的是半年或1年之内的版本即可！如果在创建向导的界面没有需要的版本号，可以随便选一下，当项目创建成功后，打开`pom.xml`，修改`<parent>`子级的`<version>`节点的值即可。

当项目创建成功后，在`src/main/java`下默认就存在一个包，是由创建项目时填写的`Package`决定的，就是当前项目组件扫描的包，相当于默认就有了`@ComponentScan("cn.tedu.boot.demo")`。

项目中默认就存在`BootDemoApplication`类，此类的名称是由创建项目时填写的`Artifact Id`加上`Application`单词组成的，这个类名称是可以改的，这个类中有`main()`方法，执行此方法就会启动整个项目，将加载项目中所有依赖所需的环境。

在`src/main/resources`下默认存在`application.properties`配置文件，它是项目默认会加载的配置文件。另外，Spring Boot的自动配置机制要求此处的许多配置是使用固定的属性名的！

## 3. 当前案例目标

客户端发出请求，最终增加管理员信息。

## 4. 开发数据访问层

### 4.1. 添加Mybatis相关依赖项

在`pom.xml`中添加必要的依赖项：

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.2</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 4.2. 配置连接数据库的信息

当添加以上依赖项，如果启动项目（执行`BootDemoApplication`类中的`main()`方法）会报告错误，因为Spring Boot允许自动配置，当添加以上依赖项后，就会自动读取连接数据库的相关信息，并自动配置数据源，甚至Mybatis所需要其它基础配置，而目前并没有配置连接数据库的相关信息，所以出现错误！

则在`application.properties`中添加配置：

```
spring.datasource.url=jdbc:mysql://localhost:3306/mall_ams?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=root
```

完成后，在`src/test/java`下找到默认即存在的测试类，在此测试类中尝试获取数据库连接对象：

```java
@SpringBootTest
class BootDemoApplicationTests {

    @Autowired
    DataSource dataSource;

    @Test
    void contextLoads() throws Exception {
        System.out.println(dataSource.getConnection());
    }

}
```

如果能顺利执行此测试，则表示以上配置是正确的！

### 4.3. 创建与数据表对应的实体类

为了简化编写POJO类，通常会在项目中添加`Lombok`依赖：

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

提示：当使用了`Lombok`后，应该在开发工具中安装Lombok插件，否则，在编写代码时，所有相关的Setters & Getters都没有自动提示，也会报告语法错误，但是不影响运行。

在插入数据时，需要使用实体类封装即将插入到表中的多个数据，则在`cn.tedu.boot.demo`包下创建`entity`子包，并在其下创建`Admin`类：

```java
@Data
public class Admin implements Serializable {
    
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
    
}
```

### 4.4. 插入管理员数据

要执行的SQL语句大致是：

```mysql
insert into ams_admin (除了id以外的字段列表……) values (值列表)
```

则在`cn.tedu.boot.demo`包下创建`mapper`子包，并在其下创建`AdminMapper`接口，在接口中添加抽象方法：

```java
package cn.tedu.boot.demo.mapper;

import cn.tedu.boot.demo.entity.Admin;
import org.springframework.stereotype.Repository;

@Repository
public interface AdminMapper {

    int insert(Admin admin);

}
```

还需要进行配置，使得Mybatis知道这些接口文件在哪里！则在`cn.tedu.boot.demo`下创建`config`包，并在此包下创建`MybatisConfiguration`类，通过`@MapperScan`配置接口文件所在的包：

```java
package cn.tedu.boot.demo.config;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@MapperScan("cn.tedu.boot.demo.mapper")
public class MybatisConfiguration {
}
```

提示：关于`@MapperScan`注解，还可以配置在项目的启动类上（`BootDemoApplication`），因为启动类上有`@SpringBootApplication`注解，其元注解中有`@SpringBootConfiguration`，其元注解中有`@Configuration`，所以，启动类本身也是配置类！但是，如果项目中的配置较多，不建议全部写在启动类中，所以，可以分为多个配置类，独立配置。

接下来，在`src/main/resources`下创建`mapper`文件夹，并从前序项目中复制粘贴得到`AdminMapper.xml`文件（删除原文件中已经配置的SQL等代码），然后，在此文件中配置抽象方法映射的SQL：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="cn.tedu.boot.demo.mapper.AdminMapper">

    <!-- int insert(Admin admin); -->
    <insert id="insert" useGeneratedKeys="true" keyProperty="id">
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

完成后，还是应该配置这些XML文件的位置，需要在`application.properties`中添加配置：

```
mybatis.mapper-locations=classpath:mapper/*.xml
```

接下来，应该通过测试检验以上代码是否可以正确运行，为了保证测试时可以正确的断言，应该在`src/test`下创建`resources`文件夹，并从前序项目中复制脚本文件，至少包含清空并还原数据表、插入测试数据这2个脚本文件。

然后，在`src/test/java`下的`cn.tedu.boot.demo`包下创建`mapper`子包，并在其下创建`AdminMapperTests`测试类，在类上添加`@SpringBootTest`注解，在类中自动装配`AdminMapper`类型的对象，并编写、执行测试：

```java
package cn.tedu.boot.demo.mapper;

import cn.tedu.boot.demo.entity.Admin;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.jdbc.Sql;

@SpringBootTest
public class AdminMapperTests {

    @Autowired
    AdminMapper mapper;

    // 测试插入数据是成功的
    @Test
    @Sql(scripts = {"classpath:truncate.sql"})
    @Sql(scripts = {"classpath:truncate.sql"}, executionPhase = Sql.ExecutionPhase.AFTER_TEST_METHOD)
    public void testInsertSuccessfully() {
        // 准备测试数据
        String username = "admin001";
        String password = "000000";
        Admin admin = new Admin();
        admin.setUsername(username);
        admin.setPassword(password);
        // 断言测试过程中不会抛出异常
        Assertions.assertDoesNotThrow(() -> {
            // 执行测试
            int rows = mapper.insert(admin);
            // 断言结果
            Assertions.assertEquals(1, rows);
            Assertions.assertEquals(1L, admin.getId());
        });
    }

}
```

### 4.5. 根据用户名查询用户数据

基于当前数据表的设计，每个管理员的“用户名”必须是唯一的，在提交增加管理员（或注册）时，必须先检查用户名是否被占用，如果被占用，将不允许增加（或注册）。判断“是否被占用”可以通过“根据用户名查询用户数据”来分析！

此部分练习请自行完成！

## 5. 业务逻辑层

### 5.1. 关于业务逻辑层

业务逻辑层是制定数据访问规则的层，此前的数据访问层只有功能，没有规则，例如执行“插入管理员数据”，则对应的方法就一定会执行，并不考虑是否合理，有关“合理”、“是否允许”等这样规则都是通过业务逻辑层来实现的。

业务逻辑层是数据访问层的调用者，通过调用相关的功能来保证规则的合理性、完整性、有效性！例如，在业务逻辑层中，可以先调用“根据用户名查询管理员信息”，再根据调用的返回值来决定是否执行“插入管理员数据”，就可以保证“每个管理员的用户名都是唯一的”这样的规则。

另外，业务逻辑层还需要考虑数据的完整性，因为在执行数据访问时，并不是所有必须的数据都会由客户端提交过来，在这样的过程中，业务逻辑层就需要补全一些数据，例如在“增加管理员”时，“是否启用”可能不会设计为客户端提交的数据，则业务逻辑层就可以补全此属性再调用数据访问进行插入数据操作。

对于一些特殊的数据，可能还需要在业务逻辑层中进行特殊的处理，以保证数据的合理性或有效性，典型的例如各用户的密码，由客户端提交过来的密码通常是明文，在业务逻辑层就应该对密码进行加密处理，并得到密文，然后再向数据库中写入。

在实际编写代码时，业务逻辑层的关键字是`Service`，通常业务逻辑层的类或接口名中都有此关键字。

业务逻辑层通常有2个部分，一个是接口，另一个是此接口的实现类。

**注意：强烈建议在业务逻辑层先定义接口，再编写实现类！这样做是一种基于接口编程的做法，是提倡的，并且，在后续使用基于Spring JDBC的事务管理中，也要求业务逻辑层必须有接口！**

在编写业务逻辑层，所有视为“失败”的情况都应该将异常抛出，而不要处理！

### 5.2.  自定义异常

为了更好的在业务逻辑层表现“错误”（操作失败，例如增加管理员时，用户名已存在，即视为错误），应该自定义一些异常类型，并在处理业务逻辑的过程中，当出现错误时抛出异常！

则在`cn.tedu.boot.demo`下创建`ex`子包，并在其下创建`ServiceException`异常类，继承自`RuntimeException`，并且，至少添加带`String`参数的构造方法，便于抛出异常时可以快捷封装错误的描述文本。

```java
package cn.tedu.boot.demo.ex;

public class ServiceException extends RuntimeException {
    
    public ServiceException() {
    }

    public ServiceException(String message) {
        super(message);
    }
    
}
```

提示：自定义的业务异常应该继承自`RuntimeException`，因为当抛出`RuntimeException`对象时，不需要在方法的声明上使用`throws`声明抛出，并且，此方法的调用者还必须通过`try...catch`或`throws`解决语法问题，同时，由于业务逻辑层不适合处理异常，应该始终抛出，并且，业务逻辑层的调用者是控制器层，在Spring MVC中有统一处理异常的机制，所以在控制器中也应该是始终抛出即可，那么，对于异常的语法使用是固定的，而使用`RuntimeException`就可以避免受到语法的约束！另外，在后续基于Spring JDBC的事务管理中，默认也是根据`RuntimeException`进行失败的处理的！

### 5.3. 业务接口与抽象方法

需要自定义类型将“增加管理员”的各数据封装起来，则在`cn.tedu.boot.demo`下创建`dto`子包，并在其下创建`AdminAddNewDTO`类，并在这个类中声明各属性：

```java
@Data
public class AdminAddNewDTO implements Serializable {
    private String username;
    private String password;
    private String nickname;
}
```

在`cn.tedu.boot.demo`包下创建`service`子包，并在其下创建`IAdminService`接口，并在接口中声明“增加管理员”的抽象方法：

```java
public interface IAdminService {
    
    void addNew(AdminAddNewDTO adminAddNewDTO);
    
}
```

提示：在业务逻辑层的抽象方法中，设计返回值时，仅以操作成功为前提来设计即可，因为所有的失败都会通过抛出异常的方式来表现。

提示：关于抽象方法的参数，如果参数的数量较少，直接声明即可，如果参数数量较多，则应该封装，在封装时，应该注意“将客户端会提交的数据封装在一起，如果某些数据不是客户端提交过来的，则不要封装在一起”。

### 5.4. 关于SLF4j

SLF4j是一款主流的日志框架，用于在代码中添加一些输出日志的语句，最终这些日志可以输出到控制台，或文件，甚至数据库中。

在SLF4j日志框架中，会将日志的重要程度分为几个级别，常用级别中，从不重要到非常重要，依次是：

- trace：跟踪
- debug：调试
- info：一般信息（默认）
- warn：警告
- error：错误

在使用时，可以控制日志的显示级别，较低级别的将不会被显示，例如：

- 当显示级别为`info`时，只会显示`info`、`warn`、`error`
- 当显示级别为`debug`时，只会显示`debug`、`info`、`warn`、`error`
- 当显示级别为`trace`时，会显示所有级别的日志

在Spring Boot项目中，在`spring-boot-starter`中已经集成了日志的依赖项，是可以直接使用的！在`application.properties`中添加配置，可以控制日志的显示级别，例如：

```
logging.level.cn.tedu.boot.demo.service.impl=info
```

在以上属性名中，配置的包是“根包”，例如配置为`cn.tedu`时，其子孙包中都会应用此配置。

当项目中已经添加了Lombok依赖后，可以在需要输出日志的类上添加`@Slf4j`注解，然后，在类中就可以使用名为`log`的变量来输出日志！

输出日志的示例代码：

```java
log.trace("输出trace级别的日志");
log.debug("输出debug级别的日志");
log.info("输出info级别的日志");
log.warn("输出warn级别的日志");
log.error("输出error级别的日志");
```

在开发实践中，应该根据要输出的内容的敏感程度、重要性来选择调用某个方法，以输出对应级别的日志，例如涉及关键数据的应该使用`trace`或`debug`级别，这样的话，当交付项目时，将设置日志显示级别的配置删除，或显式的配置为`info`级别，则`trace`、`debug`级别的日志将不会被输出。

另外，`warn`和`error`级别的日志不受显示级别的限制。

关于输出日志的方法，都是被重载了多次的！如果输出的内容只是1个字符串，应该使用例如：

```java
public void debug(String msg);
```

如果这个字符串中需要拼接多个变量的值，则应该使用：

```java
public void debug(String format, Object... arguments);
```

使用示例如下：

```java
log.debug("已经对密码进行加密处理，原文={}，密文={}", rawPassword, encodedPassword);
```

以上这种做法会缓存、预编译字符串，再将值代入去执行，所以执行效率还远高于`System.out.println()`的输出语句！

另外，需要注意的是，SLF4j只是一个日志框架，它提供了使用日志的标准，并没有实现输出日志的具体功能，在现行版本的Spring Boot中，还依赖了SLF4j的具体实现，默认是`logback`框架。

### 5.5. 业务实现

通常，会在`service`包下再创建`impl`子包，用于存放业务接口的实现类，并且，实现类的名称通常是“接口名（不包含首字母`I`） + `Impl`”。

业务实现类应该实现业务接口，并且，还应该添加`@Service`注解。

所以，在`cn.tedu.boot.demo.service.impl`中创建`AdminServiceImpl`类，实现`IAdminService`接口，在类上添加`@Service`注解，并重写接口中的抽象方法：

```java
package cn.tedu.boot.demo.service.impl;

import cn.tedu.boot.demo.dto.AdminAddNewDTO;
import cn.tedu.boot.demo.service.IAdminService;
import org.springframework.stereotype.Service;

@Service
public class AdminServiceImpl implements IAdminService {

    @Override
    public void addNew(AdminAddNewDTO adminAddNewDTO) {

    }
    
}
```

接下来，在编写业务方法（实现接口中的抽象方法）之前，应该整理此业务的编写思路：

```java
package cn.tedu.boot.demo.service.impl;

import cn.tedu.boot.demo.dto.AdminAddNewDTO;
import cn.tedu.boot.demo.entity.Admin;
import cn.tedu.boot.demo.ex.ServiceException;
import cn.tedu.boot.demo.mapper.AdminMapper;
import cn.tedu.boot.demo.service.IAdminService;
import cn.tedu.boot.demo.util.GlobalPasswordEncoder;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;

@Slf4j
@Service
public class AdminServiceImpl implements IAdminService {

    // 自动装配AdminMapper
    @Autowired
    private AdminMapper adminMapper;

    @Override
    public void addNew(AdminAddNewDTO adminAddNewDTO) {
        // 通过参数adminAddNewDTO中的username，调用AdminMapper的Admin getByUsername(String username)执行查询，并获取查询结果
        log.debug("即将增加管理员：{}", adminAddNewDTO);
        String username = adminAddNewDTO.getUsername();
        Admin queryResult = adminMapper.getByUsername(username);
        // 判断查询结果是否【不为null】
        if (queryResult != null) {
            // 是：表示用户名已经被占用，抛出ServiceException：增加管理员失败，用户名已经被占用
            log.warn("增加管理员失败，用户名（{}）已经被占用！", username);
            throw new ServiceException("增加管理员失败，用户名已经被占用！");
        }

        // 以参数adminAddNewDTO中的password作为明文，执行加密，得到密文密码
        String rawPassword = adminAddNewDTO.getPassword();
        String encodedPassword = GlobalPasswordEncoder.encode(rawPassword);
        log.debug("已经对密码进行加密处理，原文={}，密文={}", rawPassword, encodedPassword);

        // 创建新的Admin对象
        Admin admin = new Admin();
        // 为Admin对象的属性赋值：username,nickname来自参数adminAddNewDTO
        admin.setUsername(username);
        admin.setNickname(adminAddNewDTO.getNickname());
        // 为Admin对象的属性赋值：password > 密文密码
        admin.setPassword(encodedPassword);
        // 为Admin对象的属性赋值：avatar, phone, email, description保持为null
        // 为Admin对象的属性赋值：isEnable > 1
        admin.setIsEnable(1);
        // 为Admin对象的属性赋值：lastLoginIp > null
        // 为Admin对象的属性赋值：loginCount > 0
        admin.setLoginCount(0);
        // 为Admin对象的属性赋值：gmtLastLogin > null
        // 为Admin对象的属性赋值：gmtCreate, gmtModified > LocalDateTime.now()
        LocalDateTime now = LocalDateTime.now();
        admin.setGmtCreate(now);
        admin.setGmtModified(now);
        // 调用AdminMapper对象的int insert(Admin admin)方法插入管理员数据，并获取返回值
        log.debug("即将执行插入管理员数据：{}", admin);
        int rows = adminMapper.insert(admin);
        // 判断返回值是否不为1
        if (rows != 1) {
            // 抛出ServiceException：服务器忙，请稍后再次尝试
            log.warn("服务器忙，请稍后再次尝试！");
            throw new ServiceException("服务器忙，请稍后再次尝试！");
        }
    }

}
```

完成后，在`src/test/java`下的`cn.tedu.boot.demo`下创建`service`子包，并在其下创建`AdminServiceTests`测试类，编写并执行测试：

```java
package cn.tedu.boot.demo.service;

import cn.tedu.boot.demo.dto.AdminAddNewDTO;
import cn.tedu.boot.demo.ex.ServiceException;
import cn.tedu.boot.demo.service.impl.AdminServiceImpl;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.jdbc.Sql;

@SpringBootTest
public class AdminServiceTests {

    @Autowired
    IAdminService service;

    @Test
    @Sql(scripts = {"classpath:truncate.sql"})
    @Sql(scripts = {"classpath:truncate.sql"}, executionPhase = Sql.ExecutionPhase.AFTER_TEST_METHOD)
    public void testAddNewSuccessfully() {
        // 测试数据
        String username = "admin001";
        String password = "123456";
        String nickname = "管理员";
        AdminAddNewDTO adminAddNewDTO = new AdminAddNewDTO()
                .setUsername(username)
                .setPassword(password)
                .setNickname(nickname);
        // 断言不会抛出异常
        Assertions.assertDoesNotThrow(() -> {
            // 执行测试
            service.addNew(adminAddNewDTO);
        });
    }

    @Test
    @Sql(scripts = {"classpath:truncate.sql", "classpath:insert_data.sql"})
    @Sql(scripts = {"classpath:truncate.sql"}, executionPhase = Sql.ExecutionPhase.AFTER_TEST_METHOD)
    public void testAddNewFailBecauseUsernameConflict() {
        // 测试数据
        String username = "admin001";
        String password = "123456";
        String nickname = "管理员";
        AdminAddNewDTO adminAddNewDTO = new AdminAddNewDTO()
                .setUsername(username)
                .setPassword(password)
                .setNickname(nickname);
        // 断言不会抛出异常
        Assertions.assertThrows(ServiceException.class, () -> {
            // 执行测试
            service.addNew(adminAddNewDTO);
        });
    }

}
```

## 6. 控制器

### 6.1. 处理依赖项

当需要开发控制器时，需要在项目中存在`spring-boot-starter-web`的依赖项，此依赖项将包含此前学习时涉及的`spring-webmvc`、`jackson-databind`等依赖项。

在具体操作方面，并不需要追加添加这个依赖项，只需要将`spring-boot-starter`改为`spring-boot-starter-web`即可，并且，在`spring-boot-starter-web`中也包含了`spring-boot-starter`，所以，对此项目原本的依赖也不产生影响。

### 6.2. 简单开发

在`cn.tedu.boot.demo`下创建`controller`子包，并在其下创建`AdminController`类，作为处理“管理员”数据相关请求的控制器类，并在这个类中处理“增加管理员”的请求：

```java
package cn.tedu.boot.demo.controller;

import cn.tedu.boot.demo.dto.AdminAddNewDTO;
import cn.tedu.boot.demo.service.IAdminService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Slf4j
@RestController
@RequestMapping("/admin")
public class AdminController {

    @Autowired
    private IAdminService adminService;
    
    // http://localhost:8080/admin/add-new?username=admin001&password=1234&nickname=a001
    @RequestMapping("/add-new")
    public String addNew(AdminAddNewDTO adminAddNewDTO) {
        adminService.addNew(adminAddNewDTO);
        return "OK";
    }
    
}
```

因为`spring-boot-starter-web`中依赖了Tomcat，相当于每个Spring Boot工程都有一个内置的Tomcat，并且将Context Path配置为空字符串，所以在URL上并不需要添加其它路径，最后，启动项目时，就会自动打包部署此项目到内置的Tomcat上。

所以，执行`BootDemoApplication`，打开浏览器，通过 http://localhost:8080/admin/add-new?username=admin001&password=1234&nickname=a001 即可增加管理员。

以上只是简单的实现了数据访问，还需要解决的问题有：

- 响应的结果不是JSON格式
- 没有处理异常
- 需要提供在线API文档
- 没有对参数的基本格式进行检查

### 6.3. 响应JSON格式的数据

将此前学习Spring MVC时设计的`JsonResult`复制到此项目的`cn.tedu.boot.demo.web`包中，并且将处理请求的方法的返回值类型改为`JsonResult`类型：

```java
// http://localhost:8080/admin/add-new?username=admin001&password=1234&nickname=a001
@RequestMapping("/add-new")
public JsonResult<Void> addNew(AdminAddNewDTO adminAddNewDTO) {
    adminService.addNew(adminAddNewDTO);
	return JsonResult.ok();
}
```

完成后，重启项目，通过正确的参数即可成功增加管理员，并且可以看到响应的结果是JSON格式的数据，例如：

```json
{"state":20000,"message":null,"data":null}
```

以上数据中，`message`和`data`都没有数据，是多余的！可以在`application.properties`中添加配置，以去除JSON数据中为`null`的部分：

```
spring.jackson.default-property-inclusion=non_null
```

重启服务后，响应的JSON数据中将不再包含为`null`的部分！

### 6.4. 处理异常

目前，在业务逻辑层抛出了2种不同原因导致的异常，异常的类型是完全相同的，会导致处理异常时，无法判断是哪种情况导致的异常，所以，应该先改造异常类，在类中添加`State`属性，并要求通过构造方法传入，则每个异常对象中都会包含异常的状态码和错误时的文本描述：

```java
package cn.tedu.boot.demo.ex;

import cn.tedu.boot.demo.web.JsonResult;

public class ServiceException extends RuntimeException {

    private JsonResult.State state;

    public ServiceException() {
    }

    public ServiceException(JsonResult.State state, String message) {
        super(message);
        this.state = state;
    }

    public JsonResult.State getState() {
        return state;
    }

}
```

由于抛出异常时既包含了状态码，又包含了错误的描述文本，在`JsonResult`中还可以添加一个更加便捷的静态方法：

```java
public static JsonResult<Void> fail(ServiceException e) {
    return fail(e.getState(), e.getMessage());
}
```

为了保证能够对当前已分析的2种错误进行区分，应该在`State`枚举中添加对应的状态码：

```java
public enum State {
   OK(20000),
   ERR_CONFLICT(40900),
   ERR_INTERNAL_ERROR(50000);

   Integer value;

   State(Integer value) {
       this.value = value;
   }

   public Integer getValue() {
       return value;
   }
}
```

经过以上调整，原本的业务逻辑层的实现类将会报告错误，需要在创建并抛出异常时，除了传入错误的描述文本，还需要传入状态码

```java
@Override
public void addNew(AdminAddNewDTO adminAddNewDTO) {
    // 忽略此次不需要调整的代码... ...
    // 判断查询结果是否【不为null】
    if (queryResult != null) {
        // 是：表示用户名已经被占用，抛出ServiceException：增加管理员失败，用户名已经被占用
        log.warn("增加管理员失败，用户名（{}）已经被占用！", username);
        throw new ServiceException(JsonResult.State.ERR_CONFLICT, "增加管理员失败，用户名已经被占用！");
    }

    // 忽略此次不需要调整的代码... ...
    // 判断返回值是否不为1
    if (rows != 1) {
        // 抛出ServiceException：服务器忙，请稍后再次尝试
        log.warn("服务器忙，请稍后再次尝试！");
        throw new ServiceException(JsonResult.State.ERR_INTERNAL_ERROR, "服务器忙，请稍后再次尝试！");
    }
}
```

在`cn.tedu.boot.demo.controller`包下创建`handler`子包，并在其下创建`GlobalExceptionHandler`统一处理异常的类，在类上添加`@RestControllerAdvice`注解，并在类中处理异常。

```java
@RestControllerAdvicd
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ServiceException.class)
    public JsonResult<Void> handleServiceException(ServiceException e) {
        return JsonResult.fail(state, e.getMessage());
    }
    
}
```

### 6.5. Knife4j -- 在线API文档框架

Knife4j是国人开发一个基于Swagger2的在线API文档的框架，它可以扫描控制器所在的包，并解析每一个控制器及其内部的处理请求的方法，生成在线API文档，为前后端的开发人员的沟通提供便利。

在`pom.xml`中添加依赖：

```xml
<!-- https://mvnrepository.com/artifact/com.github.xiaoymin/knife4j-spring-boot-starter -->
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <version>2.0.9</version>
</dependency>
```

然后，需要在`application.properties`中添加配置：

```
knife4j.enable=true
```

并且，需要在`cn.tedu.boot.demo.config`下创建`Knife4jConfiguration`配置类：

```java
/**
 * Knife4j（Swagger2）的配置
 */
@Configuration
@EnableSwagger2WebMvc
public class Knife4jConfiguration {

    /**
     * 【重要】指定Controller包路径
     */
    private String basePackage = "cn.tedu.boot.demo.controller";
    /**
     * 分组名称
     */
    private String groupName = "xxx";
    /**
     * 主机名
     */
    private String host = "xxx";
    /**
     * 标题
     */
    private String title = "xxx";
    /**
     * 简介
     */
    private String description = "xxx";
    /**
     * 服务条款URL
     */
    private String termsOfServiceUrl = "http://www.apache.org/licenses/LICENSE-2.0";
    /**
     * 联系人
     */
    private String contactName = "xxx";
    /**
     * 联系网址
     */
    private String contactUrl = "xxx";
    /**
     * 联系邮箱
     */
    private String contactEmail = "xxx";
    /**
     * 版本号
     */
    private String version = "1.0.0";

    @Autowired
    private OpenApiExtensionResolver openApiExtensionResolver;

    @Bean
    public Docket docket() {
        String groupName = "1.0.0";
        Docket docket = new Docket(DocumentationType.SWAGGER_2)
                .host(host)
                .apiInfo(apiInfo())
                .groupName(groupName)
                .select()
                .apis(RequestHandlerSelectors.basePackage(basePackage))
                .paths(PathSelectors.any())
                .build()
                .extensions(openApiExtensionResolver.buildExtensions(groupName));
        return docket;
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title(title)
                .description(description)
                .termsOfServiceUrl(termsOfServiceUrl)
                .contact(new Contact(contactName, contactUrl, contactEmail))
                .version(version)
                .build();
    }

}
```

完成后，启动项目，通过 http://localhost:8080/doc.html 即可访问在线API文档。

在开发实践中，每个处理请求的方法应该限定为某1种请求方式，如果允许多种请求方式，则在API文档的菜单中会有多项。

在API文档中，菜单中的各名称默认是根据控制器类名、方法名转换得到的，通常，应该通过配置改为更加易于阅读理解的名称：

- `@Api`：是添加在控制器类上的注解，通过此注解的`tags`属性可以修改原本显示控制器类名称的位置的文本，通常，建议在配置的`tags`值上添加序号，例如：`"1. 管理员模块"`、`"2. 商品模块"`，则框架会根据值进行排序
- `@ApiOperation`：是添加在控制器类中处理请求的方法上的注解，用于配置此方法处理的请求在API文档中显示的文本
- `@ApiOperationSupport`：是添加在控制器类中处理请求的方法上的注解，通过配置其`order`属性可以指定各方法在API文档中的显示顺序
- `@ApiModelProperty`：是添加在POJO类的属性上的注解，用于对请求参数或响应结果中的某个属性进行说明，主要通过其`value`属性配置描述文本，并可通过`example`属性配置示例值，还可在响应结果时通过`position`属性指定顺序
- `@ApiImplicitParam`：是添加在控制器类中处理请求的方法上的注解，也可以作为`@ApiImplicitParams`注解的参数值，主要用于配置非封装的参数，主要配置`name`、`value`、`example`、`required`、`dataType`属性
- `@ApiImplicitParams`：是添加在控制器类中处理请求的方法上的注解，当方法有多个非封装的参数时，在方法上添加此注解，并在注解内部通过`@ApiImplicitParam`数组配置多个参数

提示：以上`@ApiImplicitParams`、`@ApiImplicitParam`和`@ApiModelProperty`可以组合使用。

配置示例--控制器类：

```java
@Api(tags = "1. 管理员模块")
@Slf4j
@RestController
@RequestMapping("/admin")
public class AdminController {

    @ApiOperationSupport(order = 10)
    @ApiOperation("增加管理员")
    @PostMapping("/add-new")
    public JsonResult<Void> addNew(AdminAddNewDTO adminAddNewDTO) {
        throw new RuntimeException("此功能尚未实现");
    }

    @ApiOperationSupport(order = 40)
    @ApiOperation("根据id查询管理员详情")
    @ApiImplicitParam(name = "id", value = "管理员id", example = "1",
            required = true, dataType = "long")
    @GetMapping("/{id:[0-9]+}")
    public JsonResult<Admin> getById(@PathVariable Long id) {
        throw new RuntimeException("此功能尚未实现");
    }

    @ApiOperationSupport(order = 41)
    @ApiOperation("根据角色类型查询管理员列表")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "roleId", value = "角色id", example = "1",
                required = true, dataType = "long"),
            @ApiImplicitParam(name = "page", value = "页码", example = "1",
                dataType = "int")
    })
    @GetMapping("/list-by-role")
    public JsonResult<Admin> listByRole(Long roleId, Integer page) {
        throw new RuntimeException("此功能尚未实现");
    }

}
```

配置示例--封装的POJO类：

```java
@Data
@Accessors(chain = true)
public class AdminAddNewDTO implements Serializable {

    @ApiModelProperty(value = "用户名", example = "admin001")
    private String username;

    @ApiModelProperty(value = "密码", example = "123456")
    private String password;

    @ApiModelProperty(value = "昵称", example = "管理员1号")
    private String nickname;

}
```

### 6.6. 检查参数的基本格式

使用`spring-boot-starter-validation`即可检查请求参数的有效性，需要在`pom.xml`中添加此依赖项：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

然后，在封装的POJO类的属性上配置检查相关注解，例如：

```java
@Data
@Accessors(chain = true)
public class AdminAddNewDTO implements Serializable {

    @NotNull(message = "增加管理员失败，必须提交用户名！")
    private String username;

    @NotNull(message = "增加管理员失败，必须提交密码！")
    private String password;

    @NotNull(message = "增加管理员失败，必须提交昵称！")
    private String nickname;

}
```

然后，在处理请求的方法中，对需要检查的参数添加`@Valid`或`@Validated`注解，例如：

```java
@PostMapping("/add-new")
public JsonResult<Void> addNew(@Valid AdminAddNewDTO adminAddNewDTO) {
    adminService.addNew(adminAddNewDTO);
    return JsonResult.ok();
}
```

至此，Validation框架已经生效，可以对以上请求的参数进行检查！

启动项目后，如果故意没有提交以上必要的参数，则会出现400错误，并且在IntelliJ IDEA控制台可以看到以下信息：

```
Resolved [org.springframework.validation.BindException: org.springframework.validation.BeanPropertyBindingResult: 3 errors<LF>Field error in object 'adminAddNewDTO' on field 'password': rejected value [null]; codes [NotNull.adminAddNewDTO.password,NotNull.password,NotNull.java.lang.String,NotNull]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [adminAddNewDTO.password,password]; arguments []; default message [password]]; default message [增加管理员失败，必须提交密码！]<LF>Field error in object 'adminAddNewDTO' on field 'username': rejected value [null]; codes [NotNull.adminAddNewDTO.username,NotNull.username,NotNull.java.lang.String,NotNull]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [adminAddNewDTO.username,username]; arguments []; default message [username]]; default message [增加管理员失败，必须提交用户名！]<LF>Field error in object 'adminAddNewDTO' on field 'nickname': rejected value [null]; codes [NotNull.adminAddNewDTO.nickname,NotNull.nickname,NotNull.java.lang.String,NotNull]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [adminAddNewDTO.nickname,nickname]; arguments []; default message [nickname]]; default message [增加管理员失败，必须提交昵称！]]
```

由于Validation框架在验证不通过时会抛出`BindException`，则可以使用Spring MVC统一处理异常的机制进行处理！

首先，在`State`枚举中添加新的枚举值：

```java
public enum State {
	OK(20000),
	ERR_BAD_REQUEST(40000),
	ERR_CONFLICT(40900),
	ERR_INTERNAL_ERROR(50000);

	// 原有其它代码……
}
```

然后在`GlobalExceptionHandler`中添加新的处理异常的方法：

```java
@ExceptionHandler(BindException.class)
public JsonResult<Void> handleBindException(BindException e) {
    String message = e.getBindingResult().getFieldError().getDefaultMessage();
    return JsonResult.fail(JsonResult.State.ERR_BAD_REQUEST, message);
}
```

重启项目后，再次故意不提交某些请求参数，将响应类似以下结果：

```json
{
  "state": 40000,
  "message": "增加管理员失败，必须提交用户名！"
}
```

提示：当验证请求参数出现多种错误时，以上语句仅会随机的显示其中1个错误的描述文本。

























## 附：关于密码加密

基本原则：存储到数据库中的密码必须加密处理！并且，必须使用不可逆的算法！

所有的哈希算法都是不可逆的，其中，消息摘要算法都属于哈希算法！典型的消息摘要算法包括MD系列的和SHA家族的算法！

由于消息摘要算法的摘要结果长度是固定的，所以，在用于加密时，如果没有做进一步处理，可能会被穷举的方式破解（破解方列举出所有可能的密码与摘要，通过查询的方式，根据密文找出原文）。

为了保障密码安全，基础的加强操作有：

- 尽可能的要求用户使用长度更长的密码，并使用安全强度更高（有更多的字符组合）的密码
- 多重加密
- 加盐
- 使用更加安全的算法（例如从MD5升级为SHA256甚至SHA512）
- 综合以上做法

甚至，在使用盐时，还可以使用随机的盐值，但是，需要注意，如果使用了随机盐值，则这个随机盐值必须被记录下来（可以作为最终密文的一部分，或者在数据表中使用单独的字段存储等），否则后续将无法正确的验证密码！













