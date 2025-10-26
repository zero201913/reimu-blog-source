---
cover: https://tncache1-f1.v3mh.com/image/2025/10/26/6468f1851ffd405bdabb7f1a5869dd72.jpg
title: AJAX 复习指南
date: 2025-10-15 18:00:00
categories: Web技术
description: AJAX（Asynchronous JavaScript and XML）是一种创建快速动态网页的技术，通过在后台与服务器进行少量数据交换，使网页实现异步更新。
---
## 一、AJAX 概述与作用

AJAX（Asynchronous JavaScript and XML）是一种创建快速动态网页的技术，通过在后台与服务器进行少量数据交换，使网页实现异步更新。

**核心作用：**

- 异步数据交换
- 局部页面更新
- 提升用户体验
- 减少服务器负载

## 二、应用场景

1. **表单验证** - 实时验证用户输入
2. **搜索建议** - 动态搜索提示
3. **无限滚动** - 滚动加载更多内容
4. **实时更新** - 聊天应用、股票行情
5. **购物车更新** - 无需刷新页面更新购物车

## 三、实际问题演示

下面通过一个完整的实时天气应用来展示 AJAX 的作用：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AJAX 实时天气应用</title>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        
        body {
            background: linear-gradient(135deg, #74b9ff 0%, #0984e3 100%);
            min-height: 100vh;
            padding: 20px;
            color: #333;
        }
        
        .container {
            max-width: 800px;
            margin: 0 auto;
        }
        
        header {
            text-align: center;
            margin-bottom: 30px;
            color: white;
        }
        
        h1 {
            font-size: 2.5rem;
            margin-bottom: 10px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
        }
        
        .subtitle {
            font-size: 1.2rem;
            opacity: 0.9;
        }
        
        .search-container {
            background: white;
            border-radius: 15px;
            padding: 20px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
            margin-bottom: 30px;
        }
        
        .search-box {
            display: flex;
            gap: 10px;
        }
        
        .search-input {
            flex: 1;
            padding: 15px;
            border: 2px solid #ddd;
            border-radius: 10px;
            font-size: 1rem;
            transition: border-color 0.3s;
        }
        
        .search-input:focus {
            outline: none;
            border-color: #74b9ff;
        }
        
        .search-btn {
            padding: 15px 25px;
            background: #74b9ff;
            color: white;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            font-size: 1rem;
            transition: background 0.3s;
        }
        
        .search-btn:hover {
            background: #0984e3;
        }
        
        .weather-display {
            background: white;
            border-radius: 15px;
            padding: 30px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
            display: none;
        }
        
        .current-weather {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 30px;
            padding-bottom: 20px;
            border-bottom: 2px solid #f1f1f1;
        }
        
        .weather-info h2 {
            font-size: 1.8rem;
            margin-bottom: 10px;
            color: #2d3436;
        }
        
        .temperature {
            font-size: 4rem;
            font-weight: bold;
            color: #0984e3;
        }
        
        .weather-icon {
            font-size: 5rem;
        }
        
        .weather-details {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }
        
        .detail-card {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 10px;
            text-align: center;
        }
        
        .detail-label {
            font-size: 0.9rem;
            color: #636e72;
            margin-bottom: 5px;
        }
        
        .detail-value {
            font-size: 1.5rem;
            font-weight: bold;
            color: #2d3436;
        }
        
        .forecast {
            margin-top: 20px;
        }
        
        .forecast-title {
            font-size: 1.5rem;
            margin-bottom: 15px;
            color: #2d3436;
        }
        
        .forecast-list {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
            gap: 15px;
        }
        
        .forecast-item {
            background: #f8f9fa;
            padding: 15px;
            border-radius: 10px;
            text-align: center;
        }
        
        .forecast-day {
            font-weight: bold;
            margin-bottom: 5px;
        }
        
        .forecast-temp {
            font-size: 1.2rem;
            color: #0984e3;
        }
        
        .loading {
            text-align: center;
            padding: 30px;
            display: none;
        }
        
        .spinner {
            border: 5px solid #f3f3f3;
            border-top: 5px solid #74b9ff;
            border-radius: 50%;
            width: 50px;
            height: 50px;
            animation: spin 1s linear infinite;
            margin: 0 auto 20px;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        .error-message {
            background: #ff7675;
            color: white;
            padding: 15px;
            border-radius: 10px;
            text-align: center;
            margin-bottom: 20px;
            display: none;
        }
        
        .recent-searches {
            margin-top: 20px;
        }
        
        .recent-title {
            font-size: 1rem;
            margin-bottom: 10px;
            color: #636e72;
        }
        
        .recent-list {
            display: flex;
            gap: 10px;
            flex-wrap: wrap;
        }
        
        .recent-item {
            background: #e3f2fd;
            padding: 8px 15px;
            border-radius: 20px;
            cursor: pointer;
            font-size: 0.9rem;
            transition: background 0.3s;
        }
        
        .recent-item:hover {
            background: #bbdefb;
        }
        
        @media (max-width: 600px) {
            .current-weather {
                flex-direction: column;
                text-align: center;
                gap: 20px;
            }
            
            .temperature {
                font-size: 3rem;
            }
            
            .weather-icon {
                font-size: 4rem;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>AJAX 实时天气应用</h1>
            <p class="subtitle">使用 AJAX 技术获取实时天气数据</p>
        </header>
        
        <div class="search-container">
            <div class="search-box">
                <input type="text" class="search-input" placeholder="输入城市名称，如：北京、上海、纽约..." id="city-input">
                <button class="search-btn" id="search-btn">搜索天气</button>
            </div>
            <div class="recent-searches">
                <div class="recent-title">最近搜索：</div>
                <div class="recent-list" id="recent-list">
                    <!-- 最近搜索的城市将通过JavaScript动态添加 -->
                </div>
            </div>
        </div>
        
        <div class="error-message" id="error-message">
            无法获取天气数据，请检查城市名称是否正确或稍后重试。
        </div>
        
        <div class="loading" id="loading">
            <div class="spinner"></div>
            <p>正在获取天气数据...</p>
        </div>
        
        <div class="weather-display" id="weather-display">
            <div class="current-weather">
                <div class="weather-info">
                    <h2 id="city-name">北京市</h2>
                    <div class="temperature" id="current-temp">25°C</div>
                    <div id="weather-desc">晴朗</div>
                </div>
                <div class="weather-icon" id="weather-icon">☀️</div>
            </div>
            
            <div class="weather-details">
                <div class="detail-card">
                    <div class="detail-label">体感温度</div>
                    <div class="detail-value" id="feels-like">26°C</div>
                </div>
                <div class="detail-card">
                    <div class="detail-label">湿度</div>
                    <div class="detail-value" id="humidity">65%</div>
                </div>
                <div class="detail-card">
                    <div class="detail-label">风速</div>
                    <div class="detail-value" id="wind-speed">3.6 m/s</div>
                </div>
                <div class="detail-card">
                    <div class="detail-label">气压</div>
                    <div class="detail-value" id="pressure">1015 hPa</div>
                </div>
            </div>
            
            <div class="forecast">
                <h3 class="forecast-title">未来5天预报</h3>
                <div class="forecast-list" id="forecast-list">
                    <!-- 天气预报将通过JavaScript动态添加 -->
                </div>
            </div>
        </div>
    </div>

    <script>
        // 模拟天气API响应数据
        const mockWeatherData = {
            '北京': {
                current: {
                    temp: 25,
                    feels_like: 26,
                    humidity: 65,
                    wind_speed: 3.6,
                    pressure: 1015,
                    weather: [{ description: '晴朗', main: 'Clear' }]
                },
                daily: [
                    { dt: Date.now()/1000, temp: { day: 26 }, weather: [{ main: 'Clear' }] },
                    { dt: Date.now()/1000 + 86400, temp: { day: 24 }, weather: [{ main: 'Clouds' }] },
                    { dt: Date.now()/1000 + 172800, temp: { day: 23 }, weather: [{ main: 'Rain' }] },
                    { dt: Date.now()/1000 + 259200, temp: { day: 22 }, weather: [{ main: 'Rain' }] },
                    { dt: Date.now()/1000 + 345600, temp: { day: 25 }, weather: [{ main: 'Clear' }] }
                ]
            },
            '上海': {
                current: {
                    temp: 28,
                    feels_like: 30,
                    humidity: 75,
                    wind_speed: 4.2,
                    pressure: 1012,
                    weather: [{ description: '多云', main: 'Clouds' }]
                },
                daily: [
                    { dt: Date.now()/1000, temp: { day: 28 }, weather: [{ main: 'Clouds' }] },
                    { dt: Date.now()/1000 + 86400, temp: { day: 27 }, weather: [{ main: 'Rain' }] },
                    { dt: Date.now()/1000 + 172800, temp: { day: 26 }, weather: [{ main: 'Rain' }] },
                    { dt: Date.now()/1000 + 259200, temp: { day: 29 }, weather: [{ main: 'Clear' }] },
                    { dt: Date.now()/1000 + 345600, temp: { day: 30 }, weather: [{ main: 'Clear' }] }
                ]
            },
            '纽约': {
                current: {
                    temp: 18,
                    feels_like: 17,
                    humidity: 60,
                    wind_speed: 5.1,
                    pressure: 1018,
                    weather: [{ description: '小雨', main: 'Rain' }]
                },
                daily: [
                    { dt: Date.now()/1000, temp: { day: 18 }, weather: [{ main: 'Rain' }] },
                    { dt: Date.now()/1000 + 86400, temp: { day: 20 }, weather: [{ main: 'Clouds' }] },
                    { dt: Date.now()/1000 + 172800, temp: { day: 22 }, weather: [{ main: 'Clear' }] },
                    { dt: Date.now()/1000 + 259200, temp: { day: 21 }, weather: [{ main: 'Clear' }] },
                    { dt: Date.now()/1000 + 345600, temp: { day: 19 }, weather: [{ main: 'Clouds' }] }
                ]
            }
        };

        // 存储最近搜索的城市
        let recentSearches = JSON.parse(localStorage.getItem('recentSearches')) || ['北京', '上海', '纽约'];
        
        // 天气图标映射
        const weatherIcons = {
            'Clear': '☀️',
            'Clouds': '☁️',
            'Rain': '🌧️',
            'Drizzle': '🌦️',
            'Thunderstorm': '⛈️',
            'Snow': '❄️',
            'Mist': '🌫️'
        };

        // 文档就绪后执行
        $(document).ready(function() {
            // 初始化最近搜索列表
            updateRecentSearches();
            
            // 绑定搜索按钮点击事件
            $('#search-btn').on('click', function() {
                const city = $('#city-input').val().trim();
                if (city) {
                    getWeatherData(city);
                }
            });
            
            // 绑定输入框回车事件
            $('#city-input').on('keypress', function(e) {
                if (e.which === 13) { // 回车键
                    const city = $(this).val().trim();
                    if (city) {
                        getWeatherData(city);
                    }
                }
            });
            
            // 默认加载北京天气
            getWeatherData('北京');
        });

        // 获取天气数据 - 模拟AJAX请求
        function getWeatherData(city) {
            // 显示加载中
            showLoading();
            
            // 模拟网络延迟
            setTimeout(function() {
                // 模拟AJAX成功/失败响应
                if (mockWeatherData[city]) {
                    // 模拟成功响应
                    displayWeatherData(city, mockWeatherData[city]);
                    addToRecentSearches(city);
                } else {
                    // 模拟失败响应
                    showError();
                }
                
                // 隐藏加载中
                hideLoading();
            }, 1000);
        }

        // 显示加载状态
        function showLoading() {
            $('#loading').show();
            $('#weather-display').hide();
            $('#error-message').hide();
        }

        // 隐藏加载状态
        function hideLoading() {
            $('#loading').hide();
        }

        // 显示错误信息
        function showError() {
            $('#error-message').show();
            $('#weather-display').hide();
        }

        // 显示天气数据
        function displayWeatherData(city, data) {
            const current = data.current;
            const daily = data.daily;
            
            // 更新当前天气信息
            $('#city-name').text(city);
            $('#current-temp').text(`${Math.round(current.temp)}°C`);
            $('#weather-desc').text(current.weather[0].description);
            $('#weather-icon').text(weatherIcons[current.weather[0].main] || '🌤️');
            
            // 更新天气详情
            $('#feels-like').text(`${Math.round(current.feels_like)}°C`);
            $('#humidity').text(`${current.humidity}%`);
            $('#wind-speed').text(`${current.wind_speed} m/s`);
            $('#pressure').text(`${current.pressure} hPa`);
            
            // 更新天气预报
            updateForecast(daily);
            
            // 显示天气信息
            $('#weather-display').show();
            $('#error-message').hide();
        }

        // 更新天气预报
        function updateForecast(daily) {
            const $forecastList = $('#forecast-list');
            $forecastList.empty();
            
            const days = ['今天', '明天', '后天', '大后天', '大大后天'];
            
            // 只显示未来5天预报
            for (let i = 0; i < 5; i++) {
                const dayData = daily[i];
                const dayName = days[i];
                const temp = Math.round(dayData.temp.day);
                const weatherMain = dayData.weather[0].main;
                const icon = weatherIcons[weatherMain] || '🌤️';
                
                const $forecastItem = $('<div>').addClass('forecast-item');
                $forecastItem.html(`
                    <div class="forecast-day">${dayName}</div>
                    <div class="forecast-icon">${icon}</div>
                    <div class="forecast-temp">${temp}°C</div>
                `);
                
                $forecastList.append($forecastItem);
            }
        }

        // 添加到最近搜索
        function addToRecentSearches(city) {
            // 如果城市已存在，先移除
            recentSearches = recentSearches.filter(item => item !== city);
            
            // 添加到数组开头
            recentSearches.unshift(city);
            
            // 只保留最近5个搜索
            if (recentSearches.length > 5) {
                recentSearches = recentSearches.slice(0, 5);
            }
            
            // 保存到本地存储
            localStorage.setItem('recentSearches', JSON.stringify(recentSearches));
            
            // 更新显示
            updateRecentSearches();
        }

        // 更新最近搜索显示
        function updateRecentSearches() {
            const $recentList = $('#recent-list');
            $recentList.empty();
            
            recentSearches.forEach(city => {
                const $recentItem = $('<div>').addClass('recent-item');
                $recentItem.text(city);
                $recentItem.on('click', function() {
                    $('#city-input').val(city);
                    getWeatherData(city);
                });
                
                $recentList.append($recentItem);
            });
        }

        // 实际AJAX请求示例（注释掉，因为需要真实的API密钥）
        /*
        function realGetWeatherData(city) {
            const apiKey = 'YOUR_API_KEY';
            const url = `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${apiKey}&units=metric&lang=zh_cn`;
            
            $.ajax({
                url: url,
                method: 'GET',
                dataType: 'json',
                beforeSend: function() {
                    showLoading();
                },
                success: function(data) {
                    displayWeatherData(city, data);
                },
                error: function(xhr, status, error) {
                    console.error('AJAX请求失败:', error);
                    showError();
                },
                complete: function() {
                    hideLoading();
                }
            });
        }
        */
    </script>
</body>
</html>
```

## 四、AJAX 核心优势总结

通过上面的天气应用示例，我们可以看到 AJAX 的以下优势：

1. **异步通信** - 不阻塞用户界面，提升用户体验
2. **局部更新** - 只更新需要变化的部分，减少数据传输
3. **实时交互** - 实现实时数据更新和交互
4. **减少带宽** - 只传输必要数据，而非整个页面

## 五、AJAX 工作原理

1. **创建 XMLHttpRequest 对象**
2. **配置请求参数**（方法、URL、异步等）
3. **设置回调函数**处理响应
4. **发送请求**
5. **处理服务器响应**

## 六、现代 AJAX 技术

- **Fetch API** - 更现代的网络请求API
- **Axios** - 流行的HTTP客户端库
- **async/await** - 更优雅的异步处理方式

## 七、实际应用中的注意事项

1. **错误处理** - 网络错误、服务器错误等
2. **加载状态** - 提供用户反馈
3. **超时处理** - 设置合理的请求超时
4. **安全性** - 防止CSRF等攻击

这个示例展示了 AJAX 在实际应用中的强大功能，通过异步数据交换创建了流畅的用户体验！