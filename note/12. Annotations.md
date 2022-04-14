# 注解

| 注解             | 所属框架 | 重要程度 | 应用位置 | 说明                                                         |
| ---------------- | -------- | -------- | -------- | ------------------------------------------------------------ |
| `@Component`     | Spring   | ★★★★★       | 组件类 | 添加此注解类将被视为“组件”，当Spring执行组件扫描时，如果发现组件类，就会自动创建类的对象 |
| `@Controller` | Spring | ★★★★★ | 控制器类 | 特定的组件：控制器，是以`@Component`作为元注解的，在基于Spring MVC的框架中，控制器类必须添加此注解，或以此为元注解的其它注解 |
| `@Service` | Spring | ★★★★★ | 业务类 | 特定的组件：业务类，是以`@Component`作为元注解的 |
| `@Repository` | Spring | ★★★★★ | 数据访问类 | 特定的组件：数据访问类，是以`@Component`作为元注解的 |
| `@ComponentScan` | Spring   | ★★★★     | 配置类   | 添加此注解后，当此类被加载时，Spring就会执行组件扫描，扫描的是此注解配置的包及其子孙包，包中的类如果添加了组件相关注解，则Spring会自动创建这些添加了组件注解的类的对象，在Spring Boot项目中，组件扫描默认的根包就是创建项目时得到包，此注解还是Spring Boot中`@SpringBootApplication`的元注解 |
| `@Configuration` | Spring   | ★★★★★    | 配置类   | 添加此注解的类会被视为配置类，在Spring框架中可以使用ApplicationContext直接加载，使类中的配置项生效，或在集成框架中，只要配置类在组件扫描范围内，此类中的配置项即可生效 |
| `@Autowired` | Spring | ★★★★★ | 属性，Setter方法，构造方法 | 当添加在属性上，Spring会自动从容器中找到合适的对象为此属性注入值，当添加在Setter方法上或构造方法上，Spring会自动调用对应的方法 |
| `@Qualifier` | Spring | ★★★★ | 属性，方法参数 | 当使用`@Autowired`自动装配时，如果存在多个匹配类型的对象，且根据名称无法装配时，可以使用此注解指定名称 |
| `@PropertySource` | Spring | ★★★★ | 配置类 | 用于指定需要读取的`.properties`配置文件，当读取配置文件后，会将数据注入到Spring内置的`Environment`对象中 |
| `@Value` | Spring | ★★★★★ | 属性，方法的参数 | 主要用于配置读取`Environment`数据的表达式，使得Spring为属性、方法的参数注入值 |
| `@Bean` | Spring | ★★★★★ | 配置类中返回对象的方法 | 使得Spring自动调用此方法，并将方法返回的对象保存在Spring容器中 |
| `@Scope` | Spring | ★★ | 组件类 | 配置此类的对象是否为单例的 |
| `@Lazy` | Spring | ★★ | 组件类 | 当组件类的对象将是单例的，配置它是否为懒加载 |
| `@PostConstruct` | javax | ★★ | 组件类的方法 | 标记此方法是“初始化”的生命周期方法，Spring会在实例化对象后自动调用此方法 |
| `@PreDestroy` | java | ★★ | 组件类的方法 | 标记此方法是“销毁”的生命周期方法，Spring会在销毁对象之前自动调用此方法 |
| `@Resource` | javax | ★★ | 属性 | 用于自动装配，从执行效果上，一定程度可以等效于`@Autowired` |
| `@MapperScan`    | Mybatis  | ★★★★★    | 配置类   | 配置Mybatis接口所在的根包，使得Mybatis可以创建这些接口的代理对象 |
| `@Mapper` | Mybatis | ★★★         | 数据访问接口 | 用于指定哪些接口是Mybatis需要创建代理对象的，不与`@MapperScan`同时使用 |
| `@Param` | Mybatis |  ★★★★★        | 方法参数 | 当Mapper接口中抽象方法的参数超过1个时，应该添加此注解，用于配置参数名称，后续，在SQL中`#{}`占位符中的名称就是此注解配置的名称 |
| `@Insert` | Mybatis | ★★ | 抽象方法 | 配置Mapper接口中抽象方法映射的SQL语句 |
| `@Delete` | Mybatis | ★★ | 抽象方法 | 配置Mapper接口中抽象方法映射的SQL语句 |
| `@Update` | Mybatis | ★★ | 抽象方法 | 配置Mapper接口中抽象方法映射的SQL语句 |
| `@Select` | Mybatis | ★★ | 抽象方法 | 配置Mapper接口中抽象方法映射的SQL语句 |
| `@Test` | JUnit | ★★★★★ | 测试方法 | 标识此方法是一个JUnit测试方法 |
| `@Sql` | Spring Test | ★★★★★ | 测试类，测试方法 | 用于配置需要执行的SQL脚本，及执行阶段，当测试类和测试方法上都添加了此注解，则测试方法上的注解会覆盖测试类上的注解 |
| `@SpringJUnitConfig` | Spring Test | ★★★★ | 测试类 | 用于指定Spring的配置类，使得当前测试类在执行测试方法之前会加载Spring环境，则在测试类中可以使用任何Spring的机制，例如自动装配等 |
| `@ResponseBody` | Spring MVC | ★★★ | 控制器类，处理请求的方法 | 当在控制器类上添加此注解，则类中所有处理请求的方法都将响应正文，当在控制器类中处理请求的方法上添加此注解，则此方法将响应正文。此注解是`@RestController`、`@RestControllerAdvice`的元注解 |
| `@RestController` | Spring MVC | ★★★★★ | 控制器类 | 标记当前类是控制器类，且类中处理请求的方法均响应正文 |
| `@RequestMapping` | Spring MVC | ★★★★★ | 控制器类，处理请求的方法 | 当在控制器类上添加此注解，通常用于配置请求路径的前缀，也可配置一些其它的参数，当在处理请求的方法上添加此注解，通常用于配置请求路径（在类的上配置的路径基础之上），也可以配置其它参数，例如限制请求方式 |
| `@GetMapping` | Spring MVC | ★★★★★ | 处理请求的方法 | 相当于`@RequestMapping(method = RequestMethod.GET)` |
| `@PostMapping` | Spring MVC | ★★★★★ | 处理请求的方法 | 相当于`@RequestMapping(method = RequestMethod.POST)` |
| `@PutMapping` | Spring MVC | ★★ | 处理请求的方法 | 相当于`@RequestMapping(method = RequestMethod.PUT)` |
| `@DeleteMapping` | Spring MVC | ★★ | 处理请求的方法 | 相当于`@RequestMapping(method = RequestMethod.DELETE)` |
| `@PathVariable` | Spring MVC | ★★★★★ | 处理请求的方法的参数 | 读取在URL中使用`{}`格式进行占位的参数值 |
| `@RequestParam` | Spring MVC | ★★★  | 处理请求的方法的参数 | 可以配置此参数的名称、是否必须、默认值 |
| `@RequestBody` | Spring MVC | ★★★★  | 处理请求的方法的参数 | 标记此参数是在请求体中获取的数据，可支持客户端通过JSON格式提交请求参数 |
| `@ExceptionHandler` | Spring MVC | ★★★★★  | 处理异常的方法 | 标记此方法是统一处理异常的方法 |
| `@ControllerAdvice` | Spring MVC | ★★★ | 类 | Spring MVC会在每次处理请求时按需调用此类中的方法，例如，处理请求时抛出异常，则会调用此类中处理异常的方法 |
| `@RestControllerAdvice` | Spring MVC | ★★★★★ | 类 | 标记此类是响应正文的且具有`@ControllerAdvice`效果的 |
| `@EnableWebMvc` | Spring MVC | ★★★ | Spring MVC的配置类 | 开启Spring MVC的增强模式，例如响应JSON格式的正文时需要开启，在Spring Boot中默认已开启，无需显式使用此注解 |
| `@SpringBootConfiguration` | Spring Boot | ★★★ | 类 | 标记此类是Spring Boot的配置类 |
| `@SpringBootApplication` | Spring Boot | ★★★★★ | 类 | 标记此类是Spring Boot的应用程序类，也是启动类，也是配置类，每个Spring Boot工程中只能有1个类添加此注解 |
| `@SpringBootTest` | Spring Boot | ★★★★★ | 测试类 | 标记此类是Spring Boot的测试类，会自动加载项目中的环境，例如Spring环境等 |
| `@Data` | Lombok | ★★★★★ | POJO类 | 将在编译期干预，生成Setters & Getters，`hashCode()`、`equals()`、`toString()` |
| `@Accessors` | Lombok | ★★★ | POJO类         | 将Setters调整为返回当前对象                                  |
| `@Slf4j` | Lombok | ★★★★★ | 类 | 将在编译期干预，将声明并实例化一个名为`log`的变量，则在此类中可以使用`log`变量输出日志 |
|                  |          |          |          |                                                              |
|                  |          |          |          |                                                              |
|                  |          |          |          |                                                              |
|                  |          |          |          |                                                              |

