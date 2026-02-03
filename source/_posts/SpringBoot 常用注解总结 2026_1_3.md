---
cover: https://image.acg.lol/file/2025/11/09/reimu-7.jpg
title: SpringBoot常用注解总结
date: 2026-1-3 09:30:00
categories: Spring Boot
excerpt: 本文详细总结了SpringBoot开发中最常用的注解，按核心启动、Web开发、配置、依赖注入、事务、数据访问、AOP、参数校验等维度分类详解，包含作用、核心属性、使用示例和注意事项，帮助开发者快速掌握注解驱动开发技巧。
---

SpringBoot的核心优势之一是**注解驱动开发**，通过注解替代传统XML配置，大幅简化开发流程。以下是Web开发中SpringBoot最常用的注解，按「核心/Web/配置/依赖注入/事务/数据访问/AOP/校验/其他」维度分类详解，包含**作用、核心属性、使用示例、注意事项**，覆盖90%以上的开发场景。
### 一、核心启动注解（必用）
#### 1. @SpringBootApplication
**核心作用**：SpringBoot应用的入口注解，用于标记主类，整合了3个核心注解的功能，是启动类的标配。  
**底层组成**：  

+ `@SpringBootConfiguration`：替代`@Configuration`，标记当前类为配置类；  
+ `@EnableAutoConfiguration`：核心！开启自动配置（SpringBoot根据依赖自动配置Bean，如引入spring-boot-starter-web则自动配置Tomcat、SpringMVC）；  
+ `@ComponentScan`：扫描当前包及其子包下的@Component、@Service、@Controller等注解，将类注册为Bean。

**使用示例**：

```java
@SpringBootApplication
// 可选：排除特定自动配置类（如排除数据源自动配置）
// @SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

**注意**：  

+ 启动类需放在**根包下**（如`com.demo`），否则`@ComponentScan`无法扫描子包的Bean；  
+ 如需自定义扫描包，可加`@ComponentScan(basePackages = "com.xxx")`。

### 二、Web开发核心注解
#### 1. @Controller & @RestController
| 注解 | 作用 | 区别 |
| --- | --- | --- |
| @Controller | 标记类为SpringMVC的控制器，处理HTTP请求，需配合`@ResponseBody`返回JSON | 默认返回视图（如jsp/html） |
| @RestController | 组合注解（`@Controller + @ResponseBody`），直接返回JSON/XML等数据 | 无需额外加`@ResponseBody` |


**使用示例**：

```java
// 传统控制器（返回视图）
@Controller
@RequestMapping("/page")
public class PageController {
    @GetMapping("/index")
    public String index() {
        return "index"; // 返回templates/index.html（Thymeleaf）
    }
}

// RESTful控制器（返回JSON）
@RestController
@RequestMapping("/api/user")
public class UserController {
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return new User(id, "张三"); // 直接返回JSON
    }
}
```

#### 2. @RequestMapping（及变体）
**核心作用**：映射HTTP请求到控制器方法，支持指定URL、请求方法、请求头、参数等。  
**常用变体（简化版）**：  

+ `@GetMapping`：仅处理GET请求（`method = RequestMethod.GET`）；  
+ `@PostMapping`：仅处理POST请求；  
+ `@PutMapping`/`@DeleteMapping`/`@PatchMapping`：对应PUT/DELETE/PATCH请求。

**核心属性**：  

+ `value/path`：请求URL（如`/user`）；  
+ `method`：请求方法（GET/POST等）；  
+ `params`：限定请求参数（如`params = "id"`，仅含id参数时匹配）；  
+ `headers`：限定请求头（如`headers = "Content-Type=application/json"`）；  
+ `produces`：指定响应类型（如`produces = "application/json;charset=UTF-8"`）。

**使用示例**：

```java
@RestController
@RequestMapping("/api/user")
public class UserController {
    // 匹配GET /api/user?id=1
    @GetMapping(params = "id")
    public String getUserById(@RequestParam Long id) {
        return "用户ID：" + id;
    }

    // 匹配POST /api/user，且请求头为JSON
    @PostMapping(headers = "Content-Type=application/json")
    public User addUser(@RequestBody User user) {
        return user;
    }
}
```

#### 3. 请求参数绑定注解
| 注解 | 作用 | 示例 |
| --- | --- | --- |
| @RequestParam | 绑定URL请求参数（如`/user?id=1`）到方法参数 | `@RequestParam Long id` |
| @PathVariable | 绑定URL路径变量（如`/user/1`）到方法参数 | `@PathVariable Long id` |
| @RequestBody | 绑定HTTP请求体（JSON/XML）到Java对象（POST/PUT请求常用） | `@RequestBody User user` |
| @RequestHeader | 绑定请求头参数到方法参数 | `@RequestHeader String token` |
| @CookieValue | 绑定Cookie值到方法参数 | `@CookieValue String JSESSIONID` |


**关键注意**：  

+ `@RequestParam`可加`required = false`（非必传）、`defaultValue = "0"`（默认值）；  
+ `@RequestBody`仅能绑定**一个**请求体，且请求头需为`Content-Type: application/json`；  
+ `@PathVariable`需保证URL中的变量名与参数名一致，不一致则加`@PathVariable("uid") Long id`。

#### 4. @CrossOrigin
**作用**：解决跨域问题（前后端分离场景必备），标记在类/方法上，允许指定源的跨域请求。  
**核心属性**：  

+ `origins`：允许的跨域源（如`"http://localhost:8080"`，`"*"`表示所有）；  
+ `maxAge`：预检请求（OPTIONS）的缓存时间（秒）；  
+ `allowedHeaders`：允许的请求头；  
+ `methods`：允许的请求方法。

**示例**：

```java
// 类级别：所有方法允许跨域
@RestController
@RequestMapping("/api")
@CrossOrigin(origins = "http://localhost:3000", maxAge = 3600)
public class ApiController {
    // 方法级别：覆盖类配置
    @GetMapping("/test")
    @CrossOrigin(origins = "*")
    public String test() {
        return "跨域成功";
    }
}
```

### 三、配置相关注解
#### 1. @Configuration & @Bean
**@Configuration**：标记类为配置类，替代传统XML配置文件（如`applicationContext.xml`）。  
**@Bean**：标记方法，返回值将注册为Spring容器中的Bean，默认名称为方法名，可通过`name/value`指定。  

**使用示例**：

```java
// 配置类
@Configuration
public class MyConfig {
    // 注册一个Bean，名称为"myHttpClient"
    @Bean(name = "myHttpClient")
    public CloseableHttpClient httpClient() {
        return HttpClientBuilder.create().build();
    }

    // 依赖其他Bean（自动注入）
    @Bean
    public UserService userService(UserMapper userMapper) {
        return new UserServiceImpl(userMapper);
    }
}
```

**注意**：`@Bean`方法默认是单例的，如需多例，配合`@Scope("prototype")`。

#### 2. @Value
**作用**：读取配置文件（如`application.yml/application.properties`）中的值，注入到字段/方法参数。  
**语法**：  

+ 直接取值：`@Value("${app.name}")`；  
+ 默认值：`@Value("${app.port:8080}")`（配置不存在时用8080）；  
+ SpEL表达式：`@Value("#{10+20}")`（计算值30）。

**示例**：

```java
@Component
public class AppConfig {
    // 读取application.yml中的app.name
    @Value("${app.name}")
    private String appName;

    // 读取端口，默认8080
    @Value("${app.port:8080}")
    private Integer appPort;

    // 打印配置
    @PostConstruct
    public void printConfig() {
        System.out.println("应用名称：" + appName + "，端口：" + appPort);
    }
}
```

#### 3. @ConfigurationProperties
**作用**：批量绑定配置文件中的属性到Java类（比`@Value`更适合复杂配置），常用前缀统一管理。  
**使用步骤**：  

1. 加注解指定前缀；  
2. 类需有`setter`方法（或用Lombok的`@Data`）；  
3. 加`@Component`（或在配置类加`@EnableConfigurationProperties`）注册为Bean。

**示例**：

```yaml
# application.yml
app:
  server:
    host: localhost
    port: 9090
    timeout: 3000
```

```java
@Data // Lombok，自动生成setter/getter
@Component
@ConfigurationProperties(prefix = "app.server")
public class ServerConfig {
    private String host;
    private Integer port;
    private Integer timeout;
}

// 使用配置
@RestController
public class ConfigController {
    @Autowired
    private ServerConfig serverConfig;

    @GetMapping("/config")
    public ServerConfig getConfig() {
        return serverConfig; // 返回{"host":"localhost","port":9090,"timeout":3000}
    }
}
```

#### 4. @PropertySource
**作用**：加载自定义配置文件（非默认的application.yml/properties），如`config/db.properties`。  
**示例**：

```java
@Configuration
@PropertySource(value = "classpath:config/db.properties", encoding = "UTF-8")
public class DbConfig {
    @Value("${db.url}")
    private String dbUrl;
}
```

### 四、依赖注入注解
#### 1. @Autowired
**核心作用**：自动注入Spring容器中的Bean，按**类型**匹配（默认必须存在，否则报错）。  
**核心属性**：  

+ `required = false`：允许Bean不存在，注入null；  
+ 配合`@Qualifier`：按**名称**匹配（解决同类型多Bean问题）。

**示例**：

```java
// 接口
public interface OrderService {
    void createOrder();
}

// 实现类1
@Service("alipayOrderService")
public class AlipayOrderService implements OrderService {
    @Override
    public void createOrder() {
        System.out.println("支付宝下单");
    }
}

// 实现类2
@Service("wechatOrderService")
public class WechatOrderService implements OrderService {
    @Override
    public void createOrder() {
        System.out.println("微信下单");
    }
}

// 使用
@Service
public class OrderManager {
    // 按类型匹配（报错，因为有2个OrderService实现）
    // @Autowired
    // private OrderService orderService;

    // 方案1：@Qualifier指定名称
    @Autowired
    @Qualifier("alipayOrderService")
    private OrderService orderService;

    // 方案2：@Primary（标记其中一个实现为优先）
    // @Service
    // @Primary
    // public class AlipayOrderService implements OrderService {...}
}
```

#### 2. @Resource
**作用**：JDK自带的注入注解（非Spring），默认按**名称**匹配，无匹配则按类型。  
**区别于@Autowired**：  

| 维度 | @Autowired | @Resource |
| --- | --- | --- |
| 来源 | Spring注解 | JDK javax.annotation |
| 匹配规则 | 默认按类型 | 默认按名称（name属性） |
| 依赖配置 | 需配合@Qualifier指定名称 | 直接用name属性指定名称 |


**示例**：

```java
@Service
public class OrderManager {
    // 按名称"wechatOrderService"注入
    @Resource(name = "wechatOrderService")
    private OrderService orderService;
}
```

#### 3. @Primary
**作用**：标记Bean为「优先候选」，当同类型多Bean时，`@Autowired`会优先注入标记的Bean。  

#### 4. @Scope
**作用**：指定Bean的作用域，默认`singleton`（单例）。  
**常用值**：  

+ `singleton`：单例（默认），容器中仅一个实例；  
+ `prototype`：多例，每次获取创建新实例；  
+ `request`：每个HTTP请求创建一个实例；  
+ `session`：每个HTTP会话创建一个实例。

**示例**：

```java
@Configuration
public class ScopeConfig {
    // 多例Bean
    @Bean
    @Scope("prototype")
    public User user() {
        return new User();
    }
}
```

### 五、事务管理注解
#### @Transactional
**核心作用**：声明式事务管理，标记在类/方法上，Spring自动管理事务（提交/回滚）。  
**核心属性**：  

| 属性 | 作用 | 常用值 |
| --- | --- | --- |
| propagation | 事务传播行为（嵌套事务规则） | REQUIRED（默认）、REQUIRES_NEW等 |
| isolation | 事务隔离级别 | READ_COMMITTED（默认）、REPEATABLE_READ等 |
| rollbackFor | 指定触发回滚的异常类型（默认仅运行时异常回滚） | Exception.class、RuntimeException.class |
| noRollbackFor | 指定不回滚的异常类型 | IllegalArgumentException.class |
| timeout | 事务超时时间（秒），超时则回滚 | 30（默认-1，无超时） |
| readOnly | 是否只读事务（优化性能，仅查询时用） | true/false（默认false） |


**使用示例**：

```java
@Service
// 类级别：所有方法应用事务（可被方法级别覆盖）
@Transactional(rollbackFor = Exception.class, timeout = 30)
public class UserServiceImpl implements UserService {
    @Autowired
    private UserMapper userMapper;

    @Override
    // 方法级别：覆盖类配置（只读）
    @Transactional(readOnly = true)
    public User getUserById(Long id) {
        return userMapper.selectById(id);
    }

    @Override
    // 增删改：读写事务
    public void addUser(User user) {
        userMapper.insert(user);
        // 抛异常则回滚
        if (user.getId() == null) {
            throw new RuntimeException("用户ID为空");
        }
    }
}
```

**注意**：  

+ 事务仅对**public方法**生效（Spring AOP限制）；  
+ 自调用（如方法A调用本类方法B）事务不生效（需通过Spring代理调用）；  
+ 需确保数据源配置正确，且SpringBoot已引入`spring-boot-starter-jdbc`/`spring-boot-starter-data-jpa`。

### 六、数据访问注解
#### 1. MyBatis相关
+ `@Mapper`：标记接口为MyBatis的Mapper接口，SpringBoot自动扫描并创建代理对象；  
+ `@MapperScan`：批量扫描Mapper接口（替代每个接口加`@Mapper`），通常加在启动类。

**示例**：

```java
// 方式1：单个Mapper加@Mapper
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM user WHERE id = #{id}")
    User selectById(Long id);
}

// 方式2：启动类批量扫描
@SpringBootApplication
@MapperScan(basePackages = "com.demo.mapper") // 扫描com.demo.mapper下所有Mapper
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

#### 2. Spring Data JPA相关
+ `@Repository`：标记类为数据访问层（DAO），Spring自动捕获持久化异常并转换为Spring统一的DataAccessException；  
+ `@Entity`：标记类为JPA实体类，映射数据库表；  
+ `@Table`：指定实体类对应的数据库表名（默认类名小写）；  
+ `@Id`：标记字段为主键；  
+ `@GeneratedValue`：指定主键生成策略（如AUTO、IDENTITY）。

**示例**：

```java
@Entity
@Table(name = "t_user") // 对应数据库t_user表
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) // 自增主键
    private Long id;
    private String name;
    // getter/setter
}

// DAO层
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // 自动生成SQL：SELECT * FROM t_user WHERE name = ?
    List<User> findByName(String name);
}
```

### 七、AOP相关注解
AOP（面向切面编程）用于日志、权限、性能监控等横切逻辑，核心注解如下：  

+ `@Aspect`：标记类为切面类；  
+ `@Pointcut`：定义切入点（需指定表达式，如匹配某个包下的所有方法）；  
+ 通知注解：  
    - `@Before`：切入点方法执行前执行；  
    - `@After`：切入点方法执行后执行（无论是否异常）；  
    - `@AfterReturning`：切入点方法正常返回后执行；  
    - `@AfterThrowing`：切入点方法抛出异常后执行；  
    - `@Around`：环绕通知（最强大，可控制切入点方法的执行）。

**示例**：

```java
// 切面类
@Aspect
@Component // 必须注册为Spring Bean
public class LogAspect {
    // 定义切入点：匹配com.demo.service包下所有public方法
    @Pointcut("execution(public * com.demo.service..*(..))")
    public void servicePointcut() {}

    // 前置通知：方法执行前打印日志
    @Before("servicePointcut()")
    public void before(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("方法" + methodName + "开始执行");
    }

    // 环绕通知：统计方法执行时间
    @Around("servicePointcut()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = joinPoint.proceed(); // 执行原方法
        long end = System.currentTimeMillis();
        System.out.println("方法执行耗时：" + (end - start) + "ms");
        return result;
    }
}
```

### 八、参数校验注解
基于JSR303/JSR380，配合`@Valid`/`@Validated`实现请求参数校验，常用注解：  

| 注解 | 作用 | 示例 |
| --- | --- | --- |
| @NotNull | 字段不能为null | `@NotNull(message = "ID不能为空")` |
| @NotBlank | 字符串不能为null/空串/全空格 | `@NotBlank(message = "姓名不能为空")` |
| @NotEmpty | 集合/数组不能为null/空 | `@NotEmpty(message = "列表不能为空")` |
| @Size | 字符串/集合长度范围 | `@Size(min = 6, max = 20, message = "密码长度6-20位")` |
| @Email | 字符串需符合邮箱格式 | `@Email(message = "邮箱格式错误")` |
| @Pattern | 字符串匹配正则表达式 | `@Pattern(regexp = "^\\d{11}$", message = "手机号格式错误")` |


**使用示例**：

```java
// 实体类
@Data
public class UserDTO {
    @NotNull(message = "ID不能为空")
    private Long id;

    @NotBlank(message = "姓名不能为空")
    @Size(min = 2, max = 10, message = "姓名长度2-10位")
    private String name;

    @Pattern(regexp = "^\\d{11}$", message = "手机号格式错误")
    private String phone;
}

// 控制器
@RestController
@RequestMapping("/api/user")
public class UserController {
    // @Valid：校验请求体参数
    @PostMapping
    public String addUser(@Valid @RequestBody UserDTO userDTO, BindingResult result) {
        // 处理校验结果
        if (result.hasErrors()) {
            return result.getFieldError().getDefaultMessage();
        }
        return "添加成功";
    }
}
```

**注意**：  

+ `@Valid`是JSR303注解，`@Validated`是Spring扩展（支持分组校验）；  
+ 需引入依赖`spring-boot-starter-validation`（SpringBoot 2.3+需手动引入）。

### 九、其他高频注解
| 注解 | 作用 |
| --- | --- |
| @Component | 通用Bean注解，标记类为Spring组件（@Service/@Repository/@Controller都是其派生） |
| @Service | 标记类为业务层（Service）Bean，语义更清晰 |
| @PostConstruct | 标记方法在Bean初始化后执行（构造方法之后，init-method之前） |
| @PreDestroy | 标记方法在Bean销毁前执行 |
| @Profile | 按环境加载Bean（如dev/test/prod） |
| @Async | 标记方法为异步执行（需配合`@EnableAsync`在启动类） |
| @Scheduled | 标记方法为定时任务（需配合`@EnableScheduling`在启动类） |


**示例（定时任务）**：

```java
@SpringBootApplication
@EnableScheduling // 开启定时任务
@EnableAsync // 开启异步
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}

@Service
public class TaskService {
    // 每5秒执行一次
    @Scheduled(fixedRate = 5000)
    public void fixedRateTask() {
        System.out.println("定时任务执行：" + LocalDateTime.now());
    }

    // 异步方法
    @Async
    public CompletableFuture<String> asyncTask() {
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "异步任务完成";
        });
    }
}
```

### 总结
SpringBoot注解的核心逻辑是「约定大于配置」，以上注解覆盖了**启动配置、Web开发、依赖注入、事务、数据访问、AOP、校验、定时任务**等核心场景。实际开发中：  

1. 启动类必加`@SpringBootApplication`；  
2. Web层用`@RestController + @GetMapping/@PostMapping`；  
3. 配置用`@Configuration + @Bean`或`@ConfigurationProperties`；  
4. 依赖注入优先用`@Autowired + @Qualifier`或`@Resource`；  
5. 事务用`@Transactional`；  
6. 通用横切逻辑用AOP注解；  
7. 参数校验用`@Valid + JSR303注解`。

掌握这些注解，就能高效完成SpringBoot Web开发的绝大部分场景。


