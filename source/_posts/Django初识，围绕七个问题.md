---
cover: https://pic.zero02.space/?tag=reimu&random=1&raw=1
title: Django入门笔记
date: 2025-11-26 09:30:00
categories: Python框架
excerpt: Django是一款基于Python的**开源Web应用框架**，诞生于2005年，核心理念是“快速开发、DRY（Don’t Repeat Yourself，不要重复造轮子）”。它提供了从数据库操作到前端渲染的全套解决方案，适合快速搭建高安全性、可维护性的Web项目（比如博客、电商、管理后台等）
---


# Django入门笔记：7个问题带你快速搞懂Django

## 问题一：Django框架概述
Django是一款基于Python的**开源Web应用框架**，诞生于2005年，核心理念是“快速开发、DRY（Don’t Repeat Yourself，不要重复造轮子）”。它提供了从数据库操作到前端渲染的全套解决方案，适合快速搭建高安全性、可维护性的Web项目（比如博客、电商、管理后台等）。


## 问题二：Django框架的特点
1. **MTV架构**：替代传统MVC，分工清晰易维护
2. **自带“全家桶”**：内置ORM、模板引擎、表单处理、认证系统等，无需额外装插件
3. **安全性高**：默认防护SQL注入、XSS、CSRF等攻击
4. **开发效率高**：自带管理后台（admin），几行代码就能生成数据管理页面
5. **可扩展性强**：支持自定义插件、中间件，适配各种复杂场景


## 问题三：Django框架的设计模式
Django采用**MTV设计模式**，对应MVC的变种：
- **M（Model，模型）**：对应MVC的Model，负责与数据库交互（定义数据结构）
- **T（Template，模板）**：对应MVC的View，负责页面渲染（展示数据）
- **V（View，视图）**：对应MVC的Controller，负责业务逻辑（处理请求、调用模型、返回响应）


## 问题四：Django框架的交互流程
用户访问Django项目的完整生命周期：
1. 用户发送HTTP请求到服务器
2. **URL路由**匹配请求路径，找到对应的视图函数
3. **视图**调用模型操作数据库，或处理业务逻辑
4. **模型**与数据库交互，返回数据给视图
5. **视图**将数据传递给模板，模板渲染成HTML
6. 服务器将渲染后的响应返回给用户


## 问题五：Django框架的基本使用
以搭建简单项目为例：
1. 安装：`pip install django`
2. 创建项目：`django-admin startproject 项目名`
3. 创建应用：`python manage.py startapp 应用名`（需在`settings.py`的`INSTALLED_APPS`中注册）
4. 定义模型：在应用的`models.py`中写数据类，执行`python manage.py makemigrations`（生成迁移文件）、`python manage.py migrate`（同步到数据库）
5. 配置路由：在项目的`urls.py`中关联应用的视图
6. 写视图：在应用的`views.py`中定义处理请求的函数
7. 写模板：在应用下建`templates`文件夹，写HTML模板
8. 启动服务：`python manage.py runserver`


## 问题六：Django框架的两大对象
1. **HttpRequest对象**：视图函数的第一个参数，包含请求信息（比如`request.method`获取请求方式、`request.GET/POST`获取参数）
2. **HttpResponse对象**：视图函数的返回值，用于向用户返回响应（比如`HttpResponse("文本内容")`、`render(request, "模板名.html", 数据字典)`）


## 问题七：保持登录状态Cookie-Session
- **Cookie**：保存在客户端的小型文本文件，Django中可通过`response.set_cookie("键", "值")`设置，`request.COOKIES.get("键")`获取；缺点是不安全（可被篡改）、容量小。
- **Session**：数据保存在服务器，客户端仅存SessionID（通过Cookie传递）。Django中默认启用Session，直接用`request.session["键"] = "值"`设置，`request.session.get("键")`获取；安全且容量大，适合存储用户登录状态等敏感信息。
- **Session**：数据保存在服务器，客户端仅存SessionID（通过Cookie传递）。Django中默认启用Session，直接用`request.session["键"] = "值"`设置，`request.session.get("键")`获取；安全且容量大，适合存储用户登录状态等敏感信息。