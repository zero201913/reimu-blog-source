---
cover: https://image.acg.lol/file/2025/11/09/reimu-1.jpg
title: 从Ajax到jQuery到Axios：前端HTTP请求技术演变
date: 2026-1-1 09:30:00
categories: 前端技术
excerpt: 本文详细介绍了前端HTTP请求技术的演变历程，从原生AJAX到jQuery AJAX再到Axios，对比了三种方案的核心代码、使用方式和特点，帮助开发者理解前端请求技术的发展趋势和最佳实践。
---

端发送HTTP请求的技术随发展不断简化，从原生AJAX到jQuery封装，再到axios库，核心目标均为与后端交互，但在API设计、易用性、功能完整性上逐步优化。以下是三者发送请求的核心代码及转变特点：
## 1. 原生AJAX（XMLHttpRequest）
原生AJAX基于XMLHttpRequest对象实现，是最早的前端请求方案，优点是无需依赖任何框架，但代码繁琐，需手动处理请求状态、响应解析等逻辑。

核心代码（GET请求）：

```javascript
// 创建XMLHttpRequest对象
var xhr = new XMLHttpRequest();
// 配置请求：请求方式、URL、是否异步
xhr.open('GET', '/api/user?userId=1', true);
// 监听请求状态变化
xhr.onreadystatechange = function() {
    // readyState=4表示请求完成，status=200表示响应成功
    if (xhr.readyState === 4 && xhr.status === 200) {
        // 解析响应数据（需手动处理JSON格式）
        var response = JSON.parse(xhr.responseText);
        console.log('请求成功', response);
    }
};
// 处理请求错误
xhr.onerror = function() {
    console.error('请求失败');
};
// 发送请求
xhr.send();
```

核心代码（POST请求，提交JSON数据）：

```javascript
var xhr = new XMLHttpRequest();
xhr.open('POST', '/api/user', true);
// 设置请求头（提交JSON需指定Content-Type）
xhr.setRequestHeader('Content-Type', 'application/json;charset=utf-8');
xhr.onreadystatechange = function() {
    if (xhr.readyState === 4 && xhr.status === 200) {
        var response = JSON.parse(xhr.responseText);
        console.log('提交成功', response);
    }
};
// 发送JSON字符串数据
var data = JSON.stringify({username: 'test', password: '123456'});
xhr.send(data);
```

## 2. jQuery AJAX
jQuery对原生XMLHttpRequest进行了封装，提供了简洁的$.ajax()方法，简化了参数配置、错误处理、JSON解析等逻辑，还支持$.get()、$.post()等快捷方法，降低了使用门槛。

核心代码（$.ajax()通用方式）：

```javascript
// GET请求
$.ajax({
    url: '/api/user',
    type: 'GET',
    data: {userId: 1}, // 自动拼接为URL参数
    dataType: 'json', // 自动解析JSON响应
    success: function(response) {
        console.log('请求成功', response);
    },
    error: function(xhr, status, error) {
        console.error('请求失败', error);
    }
});

// POST请求（提交JSON）
$.ajax({
    url: '/api/user',
    type: 'POST',
    contentType: 'application/json;charset=utf-8',
    data: JSON.stringify({username: 'test', password: '123456'}),
    dataType: 'json',
    success: function(response) {
        console.log('提交成功', response);
    },
    error: function(xhr, status, error) {
        console.error('提交失败', error);
    }
});
```

快捷方法（$.get()、$.post()）：

```javascript
// $.get()快捷获取数据
$.get('/api/user', {userId: 1}, function(response) {
    console.log('请求成功', response);
}, 'json').fail(function(error) {
    console.error('请求失败', error);
});

// $.post()快捷提交数据（默认表单格式）
$.post('/api/user', {username: 'test', password: '123456'}, function(response) {
    console.log('提交成功', response);
}, 'json').fail(function(error) {
    console.error('提交失败', error);
});
```

## 3. Axios
Axios是基于Promise的HTTP客户端，支持浏览器和Node.js环境，相比jQuery AJAX，它原生支持Promise、拦截器、请求取消等高级功能，API更简洁，且不依赖jQuery，是当前前端主流的请求方案。

核心代码（GET请求）：

```javascript
// 方式1：直接调用axios.get()
axios.get('/api/user', {
    params: {userId: 1} // 自动拼接URL参数
})
.then(function(response) {
    // response.data直接是解析后的JSON数据
    console.log('请求成功', response.data);
})
.catch(function(error) {
    console.error('请求失败', error);
});

// 方式2：使用async/await（更简洁的同步写法）
async function getUser() {
    try {
        const response = await axios.get('/api/user', {params: {userId: 1}});
        console.log('请求成功', response.data);
    } catch (error) {
        console.error('请求失败', error);
    }
}
```

核心代码（POST请求，提交JSON）：

```javascript
// 方式1：axios.post()
axios.post('/api/user', {
    username: 'test',
    password: '123456' // 自动序列化为JSON，默认设置Content-Type为application/json
})
.then(function(response) {
    console.log('提交成功', response.data);
})
.catch(function(error) {
    console.error('提交失败', error);
});

// 方式2：async/await
async function addUser() {
    try {
        const response = await axios.post('/api/user', {username: 'test', password: '123456'});
        console.log('提交成功', response.data);
    } catch (error) {
        console.error('提交失败', error);
    }
}
```

## 4. 核心转变总结
1. **代码复杂度降低**：从原生AJAX的多步手动配置，到jQuery的简洁配置项，再到axios的Promise化/同步化写法，逐步减少冗余代码；
2. **功能逐步增强**：axios新增了Promise支持、拦截器（请求/响应拦截）、请求取消、自动JSON序列化/解析等高级功能，满足复杂场景需求；
3. **依赖解耦**：jQuery AJAX依赖jQuery框架，而axios是独立库，更适合现代前端工程化项目（如Vue、React项目）；
4. **错误处理优化**：从原生的onerror事件，到jQuery的error回调，再到axios的catch捕获（支持Promise链式错误传递），错误处理更灵活可靠。


