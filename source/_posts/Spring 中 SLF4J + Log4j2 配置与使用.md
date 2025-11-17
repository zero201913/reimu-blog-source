---
cover: https://image.acg.lol/file/2025/11/09/reimu-9.jpg
title: Spring 框架中使用 Slf4j
date: 2025-10-15 18:00:00
categories: Java框架
excerpt: 在 Spring 项目中正确配置和使用 **SLF4J + Log4j2** 的详细教程（基于你提供的 `log4j2.xml` 配置文件）
---

### 一、核心概念说明

- **SLF4J**：日志门面（接口），提供统一的日志调用 API，不负责具体实现。
- **Log4j2**：日志实现框架，负责日志的输出、格式化等具体功能。
- 关系：SLF4J 作为接口，Log4j2 作为实现，项目中通过 SLF4J 调用，底层由 Log4j2 处理。

### 二、步骤 1：添加依赖（Maven/Gradle）

需排除 Spring 项目中默认的日志框架（如 Logback），避免冲突，再引入 SLF4J 和 Log4j2 依赖。

#### SpringBoot Maven 配置（`pom.xml`）：

```XML
<!-- 排除 Spring Boot 默认的 Logback 依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<!-- 引入 SLF4J + Log4j2 依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>

<!-- 可选：SLF4J 桥接器（若有其他日志框架需要适配） -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>log4j-over-slf4j</artifactId>
    <version>1.7.36</version>
</dependency>
```

#### Spring Maven 配置（`pom.xml`）：

```XML
<dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>6.2.11</version>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.12.2</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>RELEASE</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.19.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j2-impl</artifactId>
            <version>2.19.0</version>
            <exclusions>
                <exclusion>
                    <artifactId>slf4j-api</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
```

### 三、步骤 2：配置 `log4j2.xml` 文件

将你提供的配置文件放在项目的 **`src/main/resources`** 目录下（Spring 会自动扫描该路径下的日志配置文件）。

#### 配置文件说明（基于你的配置）：

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!-- 
  - status="WARN"：Log4j2 自身的日志级别（建议 WARN，避免输出过多框架日志）
  - monitorInterval="30"：30秒自动检测配置文件变化，无需重启应用
-->
<Configuration status="WARN" monitorInterval="30">
    <!-- 全局属性：日志格式、路径、文件名 -->
    <Properties>
        <Property name="LOG_PATTERN">%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %c{1} - %msg%n</Property>
        <Property name="LOG_PATH">logs</Property> <!-- 日志存储路径（项目根目录下的 logs 文件夹） -->
        <Property name="LOG_FILE_NAME">app</Property> <!-- 日志文件名前缀 -->
    </Properties>

    <!-- 日志输出目标（Appenders） -->
    <Appenders>
        <!-- 1. 控制台输出 -->
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="${LOG_PATTERN}" /> <!-- 使用全局日志格式 -->
            <ThresholdFilter level="DEBUG" onMatch="ACCEPT" onMismatch="DENY" /> <!-- 输出 DEBUG 及以上级别 -->
        </Console>

        <!-- 2. INFO 级别日志文件（滚动存储） -->
        <RollingFile name="RollingFileInfo" 
                     fileName="${LOG_PATH}/${LOG_FILE_NAME}-info.log" <!-- 当前日志文件 -->
                     filePattern="${LOG_PATH}/${LOG_FILE_NAME}-info-%d{yyyy-MM-dd}-%i.log"> <!-- 滚动后文件名（按天+序号） -->
            <ThresholdFilter level="INFO" onMatch="ACCEPT" onMismatch="DENY" /> <!-- 只记录 INFO 及以上（不含 ERROR） -->
            <PatternLayout pattern="${LOG_PATTERN}" />
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" /> <!-- 每天滚动一次 -->
                <SizeBasedTriggeringPolicy size="100MB" /> <!-- 单个文件超过 100MB 也滚动 -->
            </Policies>
            <DefaultRolloverStrategy max="30" /> <!-- 最多保留 30 天的日志 -->
        </RollingFile>

        <!-- 3. ERROR 级别日志文件（单独存储，便于排查问题） -->
        <RollingFile name="RollingFileError" 
                     fileName="${LOG_PATH}/${LOG_FILE_NAME}-error.log"
                     filePattern="${LOG_PATH}/${LOG_FILE_NAME}-error-%d{yyyy-MM-dd}-%i.log">
            <ThresholdFilter level="ERROR" onMatch="ACCEPT" onMismatch="DENY" /> <!-- 只记录 ERROR 及以上 -->
            <PatternLayout pattern="${LOG_PATTERN}" />
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" />
                <SizeBasedTriggeringPolicy size="100MB" />
            </Policies>
            <DefaultRolloverStrategy max="30" />
        </RollingFile>
    </Appenders>

    <!-- 日志记录器（Loggers）：控制不同包的日志级别 -->
    <Loggers>
        <!-- 自定义包日志（例如你的业务代码包 com.example） -->
        <Logger name="com.example" level="DEBUG" additivity="false">
            <!-- 绑定输出目标：控制台、INFO文件、ERROR文件 -->
            <AppenderRef ref="Console" />
            <AppenderRef ref="RollingFileInfo" />
            <AppenderRef ref="RollingFileError" />
        </Logger>

        <!-- 第三方框架日志过滤（例如 Spring 框架，降低日志级别避免冗余） -->
        <Logger name="org.springframework" level="WARN" additivity="false">
            <AppenderRef ref="Console" />
            <AppenderRef ref="RollingFileError" /> <!-- 只记录 Spring 的错误日志 -->
        </Logger>

        <!-- 根日志（默认日志规则，所有未匹配上面的包都遵循此规则） -->
        <Root level="INFO">
            <AppenderRef ref="Console" />
            <AppenderRef ref="RollingFileInfo" />
            <AppenderRef ref="RollingFileError" />
        </Root>
    </Loggers>
</Configuration>
```

### 四、步骤 3：在代码中使用日志

通过 SLF4J 的 `LoggerFactory` 获取日志对象，调用日志方法（与具体实现解耦）。

#### 示例代码（Spring 组件中使用）：

```Java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

@Service
public class UserService {
    // 1. 获取日志对象（传入当前类的 Class，便于定位日志来源）
    private static final Logger logger = LoggerFactory.getLogger(UserService.class);

    public void login(String username) {
        // 2. 输出不同级别的日志
        logger.debug("用户 [{}] 开始登录（调试信息）", username); // DEBUG 级别（开发调试用）
        logger.info("用户 [{}] 登录成功（常规信息）", username); // INFO 级别（正常业务记录）
        try {
            // 业务逻辑...
        } catch (Exception e) {
            logger.error("用户 [{}] 登录失败", username, e); // ERROR 级别（异常信息，需传入异常对象）
        }
    }
}
```

### 五、验证配置是否生效

1. **启动项目**：Spring 会自动加载 `log4j2.xml`，并在控制台输出日志。
2. **检查日志文件**：项目根目录下会生成 `logs` 文件夹，包含 `app-info.log` 和 `app-error.log`。
3. **测试日志级别**：
   1. `DEBUG` 级别日志：仅在控制台输出（因为 `Console` 的级别是 `DEBUG`）。
   2. `INFO` 级别日志：同时输出到控制台和 `app-info.log`。
   3. `ERROR` 级别日志：同时输出到控制台、`app-info.log` 和 `app-error.log`。

### 六、常见问题解决

1. **日志不输出**：
   1. 检查依赖是否冲突（确保已排除 Logback，且 Log4j2 依赖正确）。
   2. 确认 `log4j2.xml` 路径是否正确（必须在 `src/main/resources` 下）。
   3. 检查日志级别：若代码中调用 `logger.debug()`，但 `Logger` 或 `Appender` 的级别高于 `DEBUG`（如 `INFO`），则不会输出。
2. **日志文件不生成**：
   1. 检查 `LOG_PATH` 路径是否有权限写入（例如 Linux 下需确保应用对 `logs` 文件夹有写权限）。
   2. 尝试使用绝对路径（如 `D:/logs/` 或 `/var/logs/`）避免相对路径问题。
3. **第三方框架日志过多**：
   1. 在 `<Loggers>` 中添加对应包的 `Logger` 配置，降低级别（如 Spring 设为 `WARN`，MyBatis 设为 `INFO`）。

通过以上步骤，即可在 Spring 项目中正确配置并使用 SLF4J + Log4j2，实现日志的控制台输出和文件滚动存储。