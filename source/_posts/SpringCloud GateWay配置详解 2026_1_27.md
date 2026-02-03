---
cover: https://image.acg.lol/file/2025/11/09/reimu-6.jpg
title: Spring Cloud Gateway配置详解
date: 2026-1-27 09:30:00
categories: Spring Cloud
excerpt: 本文详细介绍了Spring Cloud Gateway的核心配置，包括服务发现集成、多路由规则配置、路由断言和请求过滤器等功能，帮助开发者快速掌握网关配置技巧，实现动态负载均衡路由和静态路由的灵活配置。
---


### 一、配置整体说明
这段配置是Spring Cloud Gateway（网关）的核心yml配置，主要实现三大核心能力：

1. 与服务注册中心集成（服务发现）；
2. 多路由规则配置（动态负载均衡路由+静态路由）；
3. 路由断言（匹配条件）+ 请求过滤器（请求修改）。

### 二、配置分类详尽解析
#### 模块1：基础应用配置（通用）
```yaml
spring:
  application:
    name: getway-20000  # 网关服务的唯一名称
```

+ **作用**：标识当前网关服务在注册中心的名称（如Eureka/Nacos），是服务注册发现的基础；
+ **注意**：名称自定义，建议格式「服务类型-端口号」，便于运维识别。

#### 模块2：Gateway核心配置（核心）
```yaml
spring:
  cloud:
    gateway:
      # 子模块2.1：服务发现配置（与注册中心联动）
      discovery:
        locator:
          enabled: true # 启用服务发现路由定位器
      # 子模块2.2：路由规则配置（核心，支持多路由）
      routes: 
        # 路由1：基于服务名的动态负载均衡路由（核心）
        - id: member_route01
          uri: lb://member-provider-service
          predicates: 
            - Path=/member/**
          filters:
            - AddRequestParameter=color, blue
        # 路由2：静态路由（固定地址转发）
        - id: member_route02
          uri: http://www.baidu.com
          predicates:
            - Path=/
```

##### 子模块2.1：服务发现配置（discovery.locator）
| 配置项 | 取值 | 核心作用 |
| --- | --- | --- |
| `discovery.locator.enabled` | true | 1. 允许网关通过注册中心（Eureka/Nacos）自动发现微服务，无需硬编码服务IP/端口；   2. 开启后支持「lb://服务名」协议实现负载均衡；   3. 兜底规则：`http://网关地址/服务名小写/**` 可直接访问对应服务（无需配置routes）。 |


##### 子模块2.2：路由规则配置（routes）
路由是Gateway的核心，每个路由包含4个核心属性：`id`（唯一标识）、`uri`（转发地址）、`predicates`（断言/匹配条件）、`filters`（过滤器）。

| 路由属性 | 配置值/示例 | 详尽解析 |
| --- | --- | --- |
| `id` | member_route01/member_route02 | 路由唯一标识，自定义（建议「服务名_route序号」），不可重复； |
| `uri` | 1. `lb://member-provider-service`（动态）   2. `http://www.baidu.com`（静态） | - 动态URI：`lb://` 是Gateway内置负载均衡协议（基于Spring Cloud LoadBalancer），`member-provider-service`是注册中心的微服务名（**必须小写**），自动从注册中心拉取服务实例并实现**轮询负载均衡**；   - 静态URI：固定转发到外网/指定地址，无负载均衡； |
| `predicates`（断言） | 1. `Path=/member/**`（路径断言）   2. `Path=/`（根路径断言）   （注释的扩展断言见下表） | 路由的「匹配条件」，**所有断言都满足才会触发该路由**；   核心是`Path`路径断言，其他断言是扩展条件； |
| `filters`（过滤器） | `AddRequestParameter=color, blue` | 对转发的请求做修改：此处是「添加请求参数」，转发到后端服务时自动追加`color=blue`参数； |


##### 扩展：常用路由断言（注释部分解析）
| 断言类型 | 示例配置 | 作用（匹配条件） |
| --- | --- | --- |
| 时间断言 | `After=2026-01-26T21:28:00.000+08:00[Asia/Shanghai]` | 仅请求时间在指定时间**之后**匹配（Before=之前，Between=中间）； |
| Cookie断言 | `Cookie=user, zero` | 仅请求携带`user=zero`（支持正则）的Cookie时匹配； |
| 请求头断言 | `Header=X-Request-Id, hello` | 仅请求携带`X-Request-Id=hello`（支持正则）的请求头时匹配； |
| 主机断言 | `Host=**.zero.**` | 仅请求的Host满足通配符（如`a.zero.b`）时匹配； |
| 请求方法断言 | `Method=GET,POST` | 仅GET/POST请求匹配（支持多方法）； |
| 参数断言 | `Query=email, [\w-]+@([a-zA-Z]+\.)+[a-zA-Z]+` | 仅请求携带`email`参数且值符合邮箱正则时匹配； |
| 远程地址断言 | `RemoteAddr=127.0.0.1` | 仅请求来源IP是127.0.0.1时匹配（支持网段，如192.168.1.0/24）； |
| 权重断言 | `Weight=group1, 2` / `Weight=group1, 8` | 同分组（group1）的路由按权重分配流量（2:8 → 1:4），用于灰度发布/流量控制； |


### 三、复习文档（精简版）
#### 【Spring Cloud Gateway核心配置复习】
| 核心模块 | 关键配置 | 核心考点 |
| --- | --- | --- |
| 服务发现 | `discovery.locator.enabled=true` | 1. 开启后支持`lb://服务名`负载均衡；   2. 兜底路由：`网关地址/服务名/**`； |
| 路由核心属性 | `id/uri/predicates/filters` | 1. `uri`：`lb://`=动态负载均衡，`http://`=静态路由；   2. `predicates`：Path是核心断言，多断言需全部满足；   3. `filters`：AddRequestParameter=追加请求参数； |
| 常用断言 | Path/Method/Weight | 1. Path：路径匹配（`/**`=多级路径，`/`=根路径）；   2. Weight：同分组按权重分流； |
| 负载均衡 | `lb://服务名` | 默认轮询算法，无需额外配置； |


#### 【配置逻辑速记】
1. 网关接收请求 → 匹配`predicates`断言 → 命中对应路由 → 经过`filters`修改请求 → 转发到`uri`地址；
2. 动态路由（`lb://`）：从注册中心拉取服务实例，自动负载均衡；
3. 静态路由（`http://`）：固定转发到指定地址，无负载均衡。

### 总结（核心关键点）
1. **服务发现**：`discovery.locator.enabled=true`是网关与注册中心联动的开关，开启后支持`lb://`协议实现动态负载均衡；
2. **路由核心**：每个路由的核心是「断言匹配+地址转发+过滤器修改」，`Path`断言是最常用的匹配条件；
3. **URI区别**：`lb://服务名`是动态路由（推荐），`http://地址`是静态路由（仅用于外网/固定地址转发）。

