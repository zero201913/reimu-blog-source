---
cover: https://image.acg.lol/file/2025/11/09/reimu-8.jpg
title: Spring Boot 敏感信息加密实战（Jasypt 全流程最佳实践）
date: 2025-11-23 16:00:00
categories: Java框架
excerpt: 数据库密码、Redis密码、第三方密钥绝不能明文留在配置文件！本文手把手带你使用 jasypt-spring-boot-starter 实现一键加密，支持本地快速生成密文、生产环境安全注入盐值，彻底解决配置泄漏风险，真实项目可直接套用。
---
# Spring Boot 项目敏感信息加密实战（Jasypt 全流程最佳实践）

在实际项目中，数据库密码、Redis 密码、第三方 AccessKey 等敏感信息绝对不能明文写在 `application.yml` 里，否则一旦配置文件泄露，后果不堪设想。

本文基于 100% 基于真实项目经验，带你从 0 到 1 使用最经典、最稳定的 `jasypt-spring-boot-starter` 实现配置加密，包含所有你会踩的坑和最终解决方案。

## 为什么一定要加密配置？

- GitHub 一搜就能搜到无数明文密码泄露的项目
- 运维同学直接 cat 配置就能看到所有密码
- 代码审查时避免敏感信息暴露
- 符合大多数公司的安全合规要求

## 最终效果（你想要的样子）

```yaml
spring:
  datasource:
    username: root
    password: ENC(/lhv4hqsqXkh/zQsLajDPQ==)   # 完全看不懂，但程序能正常运行
    url: jdbc:mysql://127.0.0.1:3306/letao_mall

redis:
  password: ENC(V2Vy3Rlc2VjcmV0MTIzNDU2Nzg5MA==)

aliyun:
  access-key-id: ENC(LTAI5txxxxxx)
  access-key-secret: ENC(xxxxyourSecretxxxx)
```

程序启动时自动解密，完全透明，对业务代码零侵入。

## 完整实现步骤（亲测 2025 年最新环境可用）

### 1. 添加依赖（你现在用的最稳定老版本）

```xml
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>2.1.2</version>
</dependency>
```

> 为什么不用 3.x/4.x？因为 2.1.2 是最经典、最稳定的版本，很多老项目都在用，兼容性极好。

### 2. 生成加密密文（推荐两种方式）

#### 方式一：最简单直接（推荐）—— 用 jasypt 独立 jar 加密

1. 下载：https://github.com/jasypt/jasypt/releases/download/jasypt-1.9.3/jasypt-1.9.3-dist.zip
2. 解压后进入 lib 目录，执行：

```bash
# 简单算法（兼容所有 JDK 版本）
java -cp jasypt-1.9.3.jar org.jasypt.intf.cli.JasyptPBEStringEncryptionCLI ^
  input="letao123" password=salt321 algorithm=PBEWithMD5AndDES

# 推荐强算法（JDK 8-17 都支持）
java -cp jasypt-1.9.3.jar org.jasypt.intf.cli.JasyptPBEStringEncryptionCLI ^
  input="letao123" password=salt321 algorithm=PBEWITHHMACSHA256ANDAES_256
```

输出：

```
ENC(/lhv4hqsqXkh/zQsLajDPQ==)
```

直接复制 `ENC(...)` 整体使用。

#### 方式二：代码一键生成（适合本地开发）

```java
@Test
void encrypt() {
    StringEncryptor encryptor = context.getBean(StringEncryptor.class);
    System.out.println(encryptor.encrypt("letao123"));
}
```

### 3. 主启动类添加注解（2.1.2 版本必须加！）

```java
@SpringBootApplication
@EnableEncryptableProperties   // 这一行至关重要！老版本必须加
@MapperScan("cn.zero.letaomallspringboot.mapper")
public class LetaoMallSpringbootApplication {
    public static void main(String[] args) {
        SpringApplication.run(LetaoMallSpringbootApplication.class, args);
    }
}
```

> 新版本（3.x+）已经不需要这个注解了，但 2.1.2 必须加！

### 4. 配置文件中使用 ENC()

```yaml
spring:
  datasource:
    username: root
    password: ENC(/lhv4hqsqXkh/zQsLajDPQ==)   # 就是这么简单
```

支持 `application.yml` 和 `application.properties` 两种格式。

### 5. 设置解密盐值（盐值 = password，最重要！）

盐值绝不能写死在代码里！推荐三种安全方式：

| 方式 | 推荐指数 | 使用方法 |
|------|----------|----------|
| JVM 参数 | 5星 | IDEA Run → VM options: `-Djasypt.encryptor.password=salt321` |
| 程序参数 | 4星 | Program arguments: `--jasypt.encryptor.password=salt321` |
| 环境变量 | 5星 | 生产环境最推荐 `JASYPT_ENCRYPTOR_PASSWORD=salt321` |

> 生产环境强烈建议用环境变量或配置中心（如 Nacos、Apollo）注入，永远不要把盐值提交到 Git！

### 6. 常见坑总结（你都踩过，我帮你避坑）

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| `No qualifying bean StringEncryptor` | 没加 `@EnableEncryptableProperties` 或没配置 bean | 启动类加上注解 |
| 启动报错 `Illegal key size` | JDK 17+ 使用了强算法 | 改用 `PBEWithMD5AndDES` 或 `PBEWITHHMACSHA256ANDAES_256` |
| 加密后启动失败 | 盐值不一致 | 确保本地和生产用的 password 完全一样 |
| `encrypt.bat` 没反应 | bat 文件是空的（杀毒软件清空） | 手动创建 bat 文件调用 java -cp |

### 7. 最佳实践总结（强烈建议这么做）

1. 开发时：用 IDEA VM options 设置盐值
2. 测试/生产：用环境变量或配置中心注入盐值
3. 永远不要把 `jasypt.encryptor.password` 写在配置文件里
4. 所有新加的密码都走加密流程，明文只出现一次（加密后立即删除）
5. 加密算法推荐：`PBEWITHHMACSHA256ANDAES_256`（安全 + 兼容）