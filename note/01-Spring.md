# Spring Framework

## 1. 关于Spring

Spring框架主要解决了创建对象、管理对象的问题。

在传统的开发中，当需要某个对象时，使用`new`关键字及类型的构造方法即可创建对象，例如：

```java
Random random = new Random();
```

如果以上代码存在于某个方法中。则`random`就只是个局部变量，当方法运行结束，此变量就会被销毁！

在实际项目开发，许多对象被创建出来之后，应该长期存在于内存中，而不应该销毁，当需要使用这些对象时，通过某种方式获取对象即可，而不应该重新创建对象！

除了对象存在的时间（时长）以外，在实际项目开发中，还需要关注各个类型之间的依赖关系！例如：

```java
public class UserMapper {
    
    public void insert() {
        // 向数据表中的“用户表”中插入数据
    }
    
}
```

```java
public class UserController {
    
    public UserMapper userMapper;
    
    public void reg() {
        userMapper.insert();
    }
    
}
```

在以上示例代码中，可视为“`UserController`是依赖于`UserMapper`的”，也可以把`UserMapper`称之为“`UserController`的依赖项”。

当需要创建以上类型时，如果只是单纯的把`UserController`创建出来了，却没有关注其内部`userMapper`属性的值，甚至该属性没有值，则是错误的做法！

如果要使得`UserController`中的`UserMapper`属性是有值的，也非常简单，例如：

```java
public class UserController {
    
    public UserMapper userMapper = new UserMapper();
    
    public void reg() {
        userMapper.insert();
    }
    
}
```

但是，在实际项目中，除了`UserController`以外，还会有其它的组件也可能需要使用到`UserMapper`，如果每个组件中都使用`new UserMapper()`的语法来创建对象，就不能保证`UserMapper`对象的唯一性，就违背了设计初衷。

为了更好的创建对象和管理对象，应该使用Spring框架！

## 2. 创建基于Spring的工程

当某个项目需要使用Spring框架时，推荐使用Maven工程。

创建名为`spring01`的Maven工程，创建完成后，会自动打开`pom.xml`文件，首先，应该在此文件中添加`<dependencies>`节点，此节点是用于添加依赖项的：

```xml
<dependencies>

</dependencies>
```

然后，Spring框架的依赖项的代码需要编写在以上节点的子级，而依赖项的代码推荐从 https://mvnrepository.com/ 网站查询得到，Spring的依赖项名称是`spring-context`，则在此网站搜索该名称，以找到依赖项的代码，代码示例：

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.14</version>
</dependency>
```

将以上依赖项的代码复制到`<dependencies>`节点之下即可。

然后，点击IDEA中悬浮的刷新Maven的图标，或展开右侧的Maven面板点击刷新按钮，即可自动下载所需要的jar包文件（这些文件会被下载到本机的Maven仓库中，同一个依赖项的同一个版本只会下载一次）。

## 3. 通过Spring创建对象--通过@Bean方法

**演示案例：spring01**

操作步骤：

- 创建`cn.tedu.spring.SpringBeanFactory`类
- 在类中添加方法，方法的返回值类型就是你希望Spring创建并管理的对象的类型，并在此方法中自行编写返回有效对象的代码
- 在此类上添加`@Configuration`注解
- 在此方法上添加`@Bean`注解

以上步骤的示例代码：

```java
package cn.tedu.spring;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Random;

@Configuration
public class SpringBeanFactory {

    @Bean
    public Random random() {
        return new Random();
    }

}
```

接下来，创建某个类用于执行：

```java
public class SpringRunner {

    public static void main(String[] args) {
        // 1. 加载Spring
        AnnotationConfigApplicationContext ac
                = new AnnotationConfigApplicationContext(SpringBeanFactory.class);

        // 2. 从Spring中获取对象
        Random random = (Random) ac.getBean("random");

        // 3. 测试使用对象，以便于观察是否获取到了有效的对象
        System.out.println("random > " + random);
        System.out.println("random.nextInt() > " + random.nextInt());

        // 4. 关闭
        ac.close();
    }

}
```

接下来，运行`SpringRunner`类的`main()`方法即可看到执行效果。

关于以上代码：

- 在`AnnotationConfigApplicationContext`的构造方法中，应该将`SpringBeanFactory.class`作为参数传入，否则就不会加载`SpringBeanFactory`类中内容
  - 其实，在以上案例中，`SpringBeanFactory`类上的`@Configuration`注解并不是必须的
- 在`getBean()`时，传入的字符串参数`"random"`是`SpringBeanFactory`类中的方法的名称
- 在`SpringBeanFactory`类中的方法必须添加`@Bean`注解，其作用是使得Spring框架自动调用此方法，并管理此方法返回的结果
- 关于`getBean()`方法，此方法被重载了多次，典型的有：
  - `Object getBean(String beanName)`
    - 通过此方法，传入的`beanName`必须是有效的，否则将导致`NoSuchBeanDefinitionException`
  - `T getBean(Class<T> beanClass)`;
    - 使用此方法时，传入的类型在Spring中必须**有且仅有1个对象**，如果没有匹配类型的对象，将导致`NoSuchBeanDefinitionException`，如果有2个，将导致`NoUniqueBeanDefinitionException`
  - `T getBean(String beanName, Class<T> beanClass)`
    - 此方法仍是根据传入的`beanName`获取对象，并且根据传入的`beanClass`进行类型转换
- 使用的`@Bean`注解可以传入`String`类型的参数，如果传入，则此注解对应的方法的返回结果的`beanName`就是`@Bean`注解中传入的`String`参数值

## 4. 通过Spring创建对象--组件扫描

**演示案例：spring02**

先使用同样的步骤创建`spring02`工程。

操作步骤：

- 在`pom.xml`中添加`spring-context`的依赖项

- 自行创建某个类，例如创建`cn.tedu.spring.UserMapper`类，并在类的声明之前添加`@Component`注解

- 与前次案例相似，创建可执行的类，与前次案例的区别在于：

  - 在`AnnotationConfigApplicationContext`的构造方法中传入的是`UserMapper`类的包名，即：

    ```java
    AnnotationConfigApplicationContext ac
        	= new AnnotationConfigApplicationContext("cn.tedu.spring");
    ```

  - 调用`getBean()`时，传入的名称是将`UserMapper`类的名称的首字母改为小写，即：

    ```java
    UserMapper userMapper = ac.getBean("userMapper", UserMapper.class);
    ```

关于以上代码：

- 在创建`AnnotationConfigApplicationContext`时传入的参数是一个`basePackages`，即多个“根包”，它会使得Spring框架**扫描**这个包及其子孙包中的所有类，并尝试创建这些包中的组件的对象
  - `AnnotationConfigApplicationContext`的构造方法设计的是`String...`类型的参数，即可变参数，当需要输入多个包名时，各包名使用逗号隔开即可
  - 推荐传入的包名是更加具体的，但不需要特别精准，只需要保证不会扫描到非自定义的包即可，例如包名肯定不会包含项目的依赖项的包
- 即使有了组件扫描，Spring也不会直接创建包下所有类的对象，仅当类上添加了组件注解，才会被Spring视为“组件”，Spring才会创建对应的类的对象
- 当`getBean()`时，由Spring创建的组件类的对象，默认的名称都是将首字母改为小写
  - 以上规则仅适用于：类名中的第1个字母是大写，且第2个字母是小写的情况，如果类名不符合这种情况，则`getBean()`时传入的名称就是类名（与类名完全相同的字符串）

关于组件：

- 在Spring框架中，可用的组件注解有：
  - `@Component`：通用组件注解
  - `@Controller`：应该添加在“控制器类”上
  - `@Service`：应该添加在“业务类”上
  - `@Repository`：应该添加在“数据存取类”上
- 另外，`@Configuration`是一种特殊的组件，应该添加在“配置类”上，当执行组件扫描时，添加了`@Configuration`注解的类也会被创建对象

其它：

- 可以在`@Component`等组件注解（不包含`@Configuration`）中配置字符串参数，以显式的指定Bean的名称

- 可以使用一个配置类，在配置类上通过`@ComponentScan`来指定组件扫描的包，并在加载Spring时，传入此配置类即可，例如：

  ```java
  package cn.tedu.spring;
  
  import org.springframework.context.annotation.ComponentScan;
  import org.springframework.context.annotation.Configuration;
  
  @Configuration
  @ComponentScan("cn.tedu.spring")
  public class SpringConfig {
  }
  ```

  ```java
  // 以下是加载Spring的代码片段
  AnnotationConfigApplicationContext ac
  		= new AnnotationConfigApplicationContext(SpringConfig.class);
  ```

  在使用`@ComponentScan`时，也可以传入多个包名，例如：

  ```java
  @ComponentScan({"cn.tedu.spring.controller", "cn.tedu.spring.service"})
  ```

## 5. 关于2种通过Spring创建对象的做法

以上分别介绍了使用`@Bean`方法和使用组件扫描的方式使得Spring创建对象的做法，在实际应用中：

- 使用`@Bean`方法可用在所有场景，但是使用过程相对繁琐
- 使用组件扫描的做法只适用于自定义的类型（这些类是你自己编写出来的），使用过程非常便捷

所以，当需要被Spring创建对象的类型是自定义的，应该使用组件扫描的做法，如果不是自定义的，只能使用`@Bean`方法。**这2种做法在实际的项目开发中都会被使用到**！

## 6. Spring管理的对象的作用域

由Spring管理的对象的作用域默认是单例的（并不是单例模式），对于同一个Bean，无论获取多少次，得到的都是同一个对象！如果希望某个被Spring管理的对象不是单例的，可以在类上添加`@Scope("prototype")`注解。

并且，在单例的情况下，默认不是懒加载的，还可以通过`@Lazy`注解控制它是否为懒加载模式！所谓的懒加载，就是“不要逼不得已不创建对象”。

## 7. Spring管理的对象的生命周期

由Spring创建并管理对象，则开发人员就没有了对象的控制权，无法对此对象的历程进行干预，而Spring允许在类中自定义最多2个方法，分别表示初始化方法和销毁方法，并且，Spring会在创建对象之后就执行初始化方法，在销毁对象之前执行销毁方法。

关于这2个方法，你可以按需添加，例如，当你只需要干预销毁过程时，你可以只定义销毁的方法，不需要定义初始化方法。

关于这2个方法的声明：

- 访问权限：推荐使用`public`
- 返回值类型：推荐使用`void`
- 方法名称：自定义
- 参数列表：推荐为空

然后，需要在初始化方法的声明之前添加`@PostConstruct`注解，在销毁方法的声明之前添加`@PreDestroy`注解。

例如：

```java
package cn.tedu.spring;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Lazy;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Controller;
import org.springframework.stereotype.Repository;
import org.springframework.stereotype.Service;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

@Component
public class UserMapper {

    public UserMapper() {
        System.out.println("\tUserMapper.UserMapper()");
    }

    @PostConstruct
    public void init() {
        System.out.println("\tUserMapper.init()");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("\tUserMapper.destroy()");
    }

}
```

注意：仅当类的对象被Spring管理且是单例的，才有讨论生命周期的价值，否则，不讨论生命周期。

如果某个类的对象是通过`@Bean`方法被Spring管理的，并且这个类不是自定义的，可以在`@Bean`注解中配置`initMethod`和`destroyMethod`这2个属性（它们的值就是方法名称，例如`destroyMethod="close"`），将这个类中的方法指定为生命周期方法。

## 8. Spring的自动装配机制

Spring的自动装配机制表现为：当你需要某个对象时，可以使用特定的语法，而Spring就会尝试从容器找到合适的值，并赋值到对应的位置！

最典型的表现就是在类的属性上添加`@Autowired`注解，Spring就会尝试从容器中找到合适的值为这个属性赋值！

例如有如下代码：

**SpringConfig.java**

```java
package cn.tedu.spring;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan("cn.tedu.spring")
public class SpringConfig {
}
```

**UserMapper.java**

```java
package cn.tedu.spring;

import org.springframework.stereotype.Repository;

@Repository
public class UserMapper {

    public void insert() {
        System.out.println("UserMapper.insert() >> 将用户数据写入到数据库中……");
    }

}
```

**UserController.java**

```java
package cn.tedu.spring;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;

@Controller
public class UserController {

    @Autowired // 注意：此处使用了自动装配的注解
    private UserMapper userMapper;

    public void reg() {
        System.out.println("UserController.reg() >> 控制器即将执行用户注册……");
        userMapper.insert();
    }

}
```

**SpringRunner.java**

```java
package cn.tedu.spring;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class SpringRunner {

    public static void main(String[] args) {
        // 1. 加载Spring
        AnnotationConfigApplicationContext ac
                = new AnnotationConfigApplicationContext(SpringConfig.class);

        // 2. 从Spring中获取对象
        UserController userController
                = ac.getBean("userController", UserController.class);

        // 3. 测试使用对象，以便于观察是否获取到了有效的对象
        userController.reg();

        // 4. 关闭
        ac.close();
    }
}
```

关于`@Autowired`的装配机制：

首先，会根据需要装配的数据的类型在Spring容器中查找匹配的Bean（对象）的数量，当数量为：

- 0个：判断`@Autowired`注解的`required`属性的值
  - 当`required=true`时：装配失败，启动项目时即报告异常
  - 当`required=false`时：放弃自动装配，不会报告异常
    - 后续当使用到此属性时，会出现`NullPointerException`
- 1个：直接装配，且装配成功
- 多个：自动尝试按照名称实现装配（属性的名称与Spring Bean的名称）
  - 存在与属性名称匹配的Spring Bean：装配成功
  - 不存在与属性名称匹配的Spring Bean：装配失败，启动项目时即报告异常

另外，使用`@Resource`注解也可以实现自动装配（此注解是`javax`包中的），其装配机制是先尝试根据名称来装配，如果失败，再尝试根据类型装配！

除了对属性装配以外，Spring的自动装配机制还可以表现出：如果**某个方法是由Spring框架自动调用的**（通常是构造方法，或`@Bean`方法，其它的方法中，如果参数有限制则专门说明），当这个方法被声明了参数时，Spring框架也会自动的尝试从容器找到匹配的对象，用于调用此方法！

## 9. 读取properties配置文件中的信息

操作步骤：

- 创建新的工程`spring04`，创建步骤参考前序案例

- 在`src/main/resources`文件夹下创建`jdbc.properties`，内容为：

  ```
  spring.jdbc.url=jdbc:mysql://localhost:3306/tedu
  spring.jdbc.driver=com.mysql.jdbc.Driver
  spring.jdbc.username=root
  spring.jdbc.password=1234
  spring.jdbc.init-size=5
  spring.jdbc.max-active=20
  ```

  **注意：自定义的属性名称建议添加一些前缀，避免与系统属性和Java属性冲突。**

- 在`src/main/java`下创建Java类，使用`@PropertySource`注解读取以上配置文件中的信息，则创建`cn.tedu.spring.SpringConfig`类：

  ```java
  package cn.tedu.spring;
  
  import org.springframework.context.annotation.ComponentScan;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.context.annotation.PropertySource;
  
  @Configuration
  @ComponentScan("cn.tedu.spring")
  @PropertySource("classpath:jdbc.properties")
  public class SpringConfig {
  }
  ```

  提示：当Spring框架读取了配置文件中的信息后，会将这些读取到的数据封装在内置的`Environment`对象中，后续，任何需要这些配置信息的组件都可以从`Environment`中读取到配置的数据。

- 接下来，可以创建某个Java类，从`Environment`中读取配置的数据，例如创建`JdbcConfig`类：

  ```java
  package cn.tedu.spring;
  
  import org.springframework.beans.factory.annotation.Value;
  import org.springframework.stereotype.Component;
  
  @Component
  public class JdbcConfig {
  
      @Value("${spring.jdbc.url}")
      private String url;
      @Value("${spring.jdbc.driver}")
      private String driver;
      @Value("${spring.jdbc.username}")
      private String username;
      @Value("${spring.jdbc.password}")
      private String password;
      @Value("${spring.jdbc.init-size}")
      private int initSize;
      @Value("${spring.jdbc.max-active}")
      private int maxActive;
  
      public String getUrl() {
          return url;
      }
  
      public void setUrl(String url) {
          this.url = url;
      }
  
      public String getDriver() {
          return driver;
      }
  
      public void setDriver(String driver) {
          this.driver = driver;
      }
  
      public String getUsername() {
          return username;
      }
  
      public void setUsername(String username) {
          this.username = username;
      }
  
      public String getPassword() {
          return password;
      }
  
      public void setPassword(String password) {
          this.password = password;
      }
  
      public int getInitSize() {
          return initSize;
      }
  
      public void setInitSize(int initSize) {
          this.initSize = initSize;
      }
  
      public int getMaxActive() {
          return maxActive;
      }
  
      public void setMaxActive(int maxActive) {
          this.maxActive = maxActive;
      }
  }
  ```

  提示：前序的操作中，在`SpringConfig`中已经配置了组件扫描，这个`JdbcConfig`类必须在组件扫描的范围内，并添加组件注解，这样Spring框架才会创建`JdbcConfig`类的对象，进而根据各`@Value`注解将`Environment`中的配置数据注入到属性中

- 最后，可以执行本案例：

  ```java
  package cn.tedu.spring;
  
  import org.springframework.context.annotation.AnnotationConfigApplicationContext;
  
  public class SpringRunner {
  
      public static void main(String[] args) {
          // 1. 加载Spring
          System.out.println("1. 加载Spring，开始……");
          AnnotationConfigApplicationContext ac
                  = new AnnotationConfigApplicationContext(SpringConfig.class);
          System.out.println("1. 加载Spring，完成！");
          System.out.println();
  
          // 2. 从Spring中获取对象
          System.out.println("2. 从Spring中获取对象，开始……");
          JdbcConfig jdbcConfig = ac.getBean("jdbcConfig", JdbcConfig.class);
          System.out.println("2. 从Spring中获取对象，完成！");
          System.out.println();
  
          // 3. 测试使用对象，以便于观察是否获取到了有效的对象
          System.out.println("3. 测试使用对象，开始……");
          System.out.println("\turl >> " + jdbcConfig.getUrl());
          System.out.println("\tdriver >> " + jdbcConfig.getDriver());
          System.out.println("\tusername >> " + jdbcConfig.getUsername());
          System.out.println("\tpassword >> " + jdbcConfig.getPassword());
          System.out.println("\tinit-size >> " + jdbcConfig.getInitSize());
          System.out.println("\tmax-active >> " + jdbcConfig.getMaxActive());
          System.out.println("3. 测试使用对象，完成！");
          System.out.println();
  
          // 4. 关闭
          System.out.println("4. 关闭，开始……");
          ac.close();
          System.out.println("4. 关闭，完成！");
      }
  }
  ```

另外，还可以直接装配一个`Environment`对象，并在需要的时候通过`Environment`对象读取配置的数据，例如：

```java
package cn.tedu.spring;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Component;

@Component
public class EnvironmentData {

    @Autowired
    private Environment environment;

    public Environment getEnvironment() {
        return environment;
    }

}
```

```java
package cn.tedu.spring;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.core.env.Environment;

public class SpringRunner {

    public static void main(String[] args) {
        // 1. 加载Spring
        System.out.println("1. 加载Spring，开始……");
        AnnotationConfigApplicationContext ac
                = new AnnotationConfigApplicationContext(SpringConfig.class);
        System.out.println("1. 加载Spring，完成！");
        System.out.println();

        // 2. 从Spring中获取对象
        System.out.println("2. 从Spring中获取对象，开始……");
        EnvironmentData environmentData
                = ac.getBean("environmentData", EnvironmentData.class);
        System.out.println("2. 从Spring中获取对象，完成！");
        System.out.println();

        // 3. 测试使用对象，以便于观察是否获取到了有效的对象
        System.out.println("3. 测试使用对象，开始……");
        System.out.println("---------------------------------------");
        System.out.println("通过自动装配Environment对象获取的值：");
        Environment env = environmentData.getEnvironment();
        System.out.println("\tEnvironment >> " + env);
        System.out.println("\turl >> " + env.getProperty("spring.jdbc.url"));
        System.out.println("\tdriver >> " + env.getProperty("spring.jdbc.driver"));
        System.out.println("\tusername >> " + env.getProperty("spring.jdbc.username"));
        System.out.println("\tpassword >> " + env.getProperty("spring.jdbc.password"));
        System.out.println("\tinit-size >> " + env.getProperty("spring.jdbc.init-size"));
        System.out.println("\tmax-active >> " + env.getProperty("spring.jdbc.max-active"));
        System.out.println("3. 测试使用对象，完成！");
        System.out.println();

        // 4. 关闭
        System.out.println("4. 关闭，开始……");
        ac.close();
        System.out.println("4. 关闭，完成！");
    }
}
```

## 10. 关于Spring框架的小结

关于Spring框架，你应该：

- 了解Spring框架的作用：创建对象，管理对象
- 掌握通过Spring创建对象的2种方式：
  - 在配置类（带`@Configuration`注解的类）中使用`@Bean`方法
  - 使用组件扫描，并在类上添加组件注解
    - 组件注解有：`@Component`、`@Controller`、`@Service`、`@Repository`
  - 如果是自定义的类，应该使用组件扫描+组件注解的方式，如果不是自定义的类，必须使用配置类中的`@Bean`方法
- 了解Spring Bean的作用域与生命周期
- 掌握`@Autowired`自动装配，理解其装配机制
  - 建议背下来：`@Autowired`与`@Resource`的区别
- 掌握读取`.properties`配置文件中的数据
  - 先使用`@PropertySource`注解指定需要读取的文件
  - 读取配置的数据时，可以：
    - 使用`@Value`注解将值注入到属性中
    - 自动装配`Environment`对象，并调用此对象的`getProperty()`方法以获取配置值
- 了解Spring的IoC（Inversion of Controll：控制反转）和DI（Dependency Injection：依赖注入）
  - Spring框架基于DI实现了IoC，DI是一种实现手段，IoC是最终实现的目标/效果
- Spring AOP后续再讲









