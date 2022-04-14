# Spring Boot Validation

**【框架简介】**

Spring Boot Validation是Spring Boot整合了Hibernate Validation的一个框架，其核心是Hibernate Validation，此框架的作用是**检验客户端向服务器端提交的请求参数的基本格式是否合法**。

例如，当设计一个“登录”功能时，客户端可能需要向服务器端提交用户名、密码这2项数据，如果客户端没有提交用户名，或没有提交密码，这个请求必然是无效的，是不可能成功登录的，则服务器端应该直接响应一个错误信息，根本不需要连接数据库去验证用户信息是否正确！另外，如果客户端提交的用户名是例如`a`或` `（1个空格）这样的数据，很显然也是不合法的！此类对于数据的基本格式的检验，就可以通过Spring Boot Validation便捷的实现！

**【需求】**

使用Spring Boot框架实现用户登录（不关心如何验证用户名与密码），检验用户名和密码的基本格式。

**【实现步骤】**

1. 使用Spring Boot工程，在创建过程中需要注意：
   - 建议使用Spring Boot 2.5.x版本，如果已经没有此版本的选项，则选择2.6.x版本，非常不建议选择3.x.x版本
   - 需要勾选`Web > Spring Web`和`IO > Validation`这2个依赖项

2. 在项目的`cn.tedu.validation.demo`包中创建`UserController`类，作为控制器类，并在其中添加处理登录的方法：

   ```java
   package cn.tedu.validation.demo;
   
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   public class UserController {
   
       // http://localhost:8080/login?username=root&password=1234
       @RequestMapping("/login")
       public String login(String username, String password) {
           System.out.println("username = " + username + ", password = " + password);
           return "OK";
       }
   
   }
   ```

3. 接下来，可以启动项目，并通过 http://localhost:8080/login?username=root&password=1234 测试访问，你还可以修改网址中的用户名和密码的值，在IntelliJ IDEA的控制台上会显示你提交的用户名和密码

4. 如果希望使用Spring Boot Validation验证请求参数的基本格式，应该将以上方法中接收的用户名和密码这2个数据封装起来，例如创建`UserLoginDTO`类，并在其中声明用户名和密码这2个属性，按照开发规范添加Setter & Getter方法，同时，为了便于查看数据，再生成`toString`方法，代码如下：

   ```java
   package cn.tedu.validation.demo;
   
   public class UserLoginDTO {
   
       private String username;
       private String password;
   
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
   
       @Override
       public String toString() {
           return "UserLoginDTO{" +
                   "username='" + username + '\'' +
                   ", password='" + password + '\'' +
                   '}';
       }
       
   }
   ```

5. 接下来，原本处理请求的方法的参数就可以由2个字符串改为以上这1个类的对象：

   ```java
   @RequestMapping("/login")
   public String login(UserLoginDTO userLoginDTO) {
       System.out.println("userLoginDTO = " + userLoginDTO);
       return "OK";
   }
   ```

6. 当需要验证请求参数的基本格式时，需要在以上方法的参数之前添加`@Valid`或`@Validated`注解（这2个注解是等效的），表示需要对此参数进行验证：

   ```java
   @RequestMapping("/login")
   // 注意：下一行的参数添加了@Valid注解
   public String login(@Valid UserLoginDTO userLoginDTO) {
       System.out.println("userLoginDTO = " + userLoginDTO);
       return "OK";
   }
   ```

7. 接下来，在`UserLoginDTO`类的属性上，添加注解，以配置验证规则及出错时的提示文本，例如：

   ```java
   public class UserLoginDTO {
   
       @NotNull(message = "登录失败，必须提交用户名！")
       private String username;
       private String password;
   
       // 省略后续代码
       
   }
   ```

8. 接下来，重新启动项目，提交一个没有用户名的请求，例如 http://localhost:8080/login?password=0000 ，则会提示错误，并且，在IntelliJ IDEA控制台可以看到：

   ```
   2022-03-30 20:09:35.270  WARN 10384 --- [nio-8080-exec-3] .w.s.m.s.DefaultHandlerExceptionResolver : Resolved [org.springframework.validation.BindException: org.springframework.validation.BeanPropertyBindingResult: 1 errors<EOL>Field error in object 'userLoginDTO' on field 'username': rejected value [null]; codes [NotNull.userLoginDTO.username,NotNull.username,NotNull.java.lang.String,NotNull]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [userLoginDTO.username,username]; arguments []; default message [username]]; default message [登录失败，必须提交用户名！]]
   ```

9. 提示：与`@NotNull`相似的注解还有`@NotBlank`、`@NotEmpty`，这几个是区别的：

   - `@NotNull`：不允许为`null`值，如果客户端提交的参数中根本没有这一项，则视为`null`
   - `@NotBlank`：不允许为空白值，空白值包括由若干个空格、`TAB`符等组成字符
   - `@NotEmpty`：不允许为空字符串，即不允许是长度为0的字符串

10. 当有必要的情况下，可以为属性添加多个验证的注解，例如：

    ```java
    @NotNull(message = "登录失败，必须提交用户名！")
    @Pattern(regexp = "[a-zA-Z0-9_]{4,16}", message = "登录失败，用户名格式错误！")
    private String username;
    ```

11. 在控制器中，当需要获取验证失败时的提示文本（以上注解中配置的文本）时，需要在被验证的参数右侧添加`BindingResult`参数，例如：

    ```java
    @RequestMapping("/login")
    public String login(@Valid UserLoginDTO userLoginDTO, BindingResult bindingResult) {
        // ...
    }
    ```

    注意：以上`BindingResult`参数必须直接添加在被验证对象的右侧，它们中间不可以添加其它参数！

12. 关于`BindingResult`的实用方法：

    ```java
    // 判断验证是否不通过（存在错误）
    boolean hasError = bindingResult.hasErrors();
    
    // 获取验证失败的提示文本，如果有多个规则都没被满足，则是其中的不确定的某种提示文本
    String errorMessage = bindingResult.getFieldError().getDefaultMessage();
    ```

13. 关于更多Validation注解，可参考：https://blog.csdn.net/qq_20919883/article/details/106297523

14. 在实际应用中，当使用正则表达式时，应该将所有或相关的正则表达式都定义在一起（例如在同一个类或同一个接口中），各个类中的验证参数的正则表达式注解都从此处调用，便于统一管理这些正则表达式。





















