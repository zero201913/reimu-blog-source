---
cover: https://image.acg.lol/file/2025/11/09/reimu-12.jpg
title: 前端通用axios封装：request.js最佳实践
date: 2026-1-3 09:30:00
categories: 前端技术
excerpt: 本文详细介绍了一个功能完善的axios通用封装（request.js），包括请求/响应拦截、错误统一处理、重复请求取消、请求配置扩展等核心功能，帮助开发者构建统一、可扩展、鲁棒性强的前端请求层。
---


以下是一个**功能完善、可扩展、鲁棒性强**的 axios 通用封装（request.js），涵盖请求/响应拦截、错误统一处理、重复请求取消、请求配置扩展、常用请求方法封装等核心能力，适配大部分前端项目场景。
### 最终代码（request.js）
```javascript
import axios from 'axios';
import { ElMessage, ElMessageBox } from 'element-plus'; // 适配Vue3+Element Plus，可替换为其他UI库
import { useUserStore } from '@/stores/user'; // 示例：Vue3 Pinia用户状态，可替换为Vuex/本地存储

// ===================== 核心配置 =====================
// 创建axios实例
const service = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL, // 环境变量配置接口前缀（Vite）
  timeout: 10000, // 请求超时时间
  headers: {
    'Content-Type': 'application/json;charset=utf-8' // 默认请求头
  }
});

// ===================== 重复请求取消机制 =====================
// 存储正在进行的请求 { url: CancelTokenSource }
const pendingRequests = new Map();
/**
 * 生成请求唯一标识（method + url + 参数）
 * @param {Object} config axios配置
 * @returns {String} 请求标识
 */
const getRequestKey = (config) => {
  const { method, url, params, data } = config;
  return [method, url, JSON.stringify(params), JSON.stringify(data)].join('&');
};
/**
 * 添加待取消的请求
 * @param {Object} config axios配置
 */
const addPendingRequest = (config) => {
  const requestKey = getRequestKey(config);
  // 若已有相同请求，先取消再重新添加
  if (pendingRequests.has(requestKey)) {
    pendingRequests.get(requestKey).cancel('重复请求已取消');
    pendingRequests.delete(requestKey);
  }
  const source = axios.CancelToken.source();
  config.cancelToken = source.token;
  pendingRequests.set(requestKey, source);
};
/**
 * 移除已完成/取消的请求
 * @param {Object} config axios配置
 */
const removePendingRequest = (config) => {
  const requestKey = getRequestKey(config);
  if (pendingRequests.has(requestKey)) {
    pendingRequests.delete(requestKey);
  }
};

// ===================== 请求拦截器 =====================
service.interceptors.request.use(
  (config) => {
    // 1. 添加取消请求标识
    addPendingRequest(config);

    // 2. 注入token（根据项目实际存储位置调整）
    const userStore = useUserStore();
    if (userStore.token) {
      config.headers.Authorization = `Bearer ${userStore.token}`;
    }

    // 3. GET请求参数处理（可选：防止参数为undefined导致请求异常）
    if (config.method === 'get' && config.params) {
      Object.entries(config.params).forEach(([key, value]) => {
        if (value === undefined) {
          delete config.params[key];
        }
      });
    }

    // 4. 自定义请求配置扩展（如是否显示加载中）
    config.loading ??= true; // 默认显示加载中，可通过请求配置覆盖
    if (config.loading) {
      // 示例：可在此处调用UI库的加载中组件（如ElLoading）
      // ElLoading.service({ lock: true, text: '请求中...' });
    }

    return config;
  },
  (error) => {
    ElMessage.error('请求参数异常，请检查！');
    return Promise.reject(error);
  }
);

// ===================== 响应拦截器 =====================
service.interceptors.response.use(
  (response) => {
    // 1. 移除取消请求标识
    removePendingRequest(response.config);

    // 2. 关闭加载中（若开启）
    if (response.config.loading) {
      // ElLoading.service().close(); // 关闭加载中
    }

    // 3. 解构响应数据（适配后端通用返回格式：{ code, msg, data }）
    const { code, msg, data } = response.data;

    // 4. 业务逻辑判断（根据后端约定调整）
    if (code === 200) {
      // 成功：返回核心业务数据，简化上层调用
      return data;
    } else if (code === 401) {
      // 未登录/Token过期：统一处理登出逻辑
      ElMessageBox.confirm('登录状态已过期，请重新登录', '提示', {
        confirmButtonText: '重新登录',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        const userStore = useUserStore();
        userStore.logout(); // 清空token、跳转登录页
        window.location.reload();
      });
      return Promise.reject(new Error(msg || '登录过期'));
    } else {
      // 其他业务错误：统一提示
      ElMessage.error(msg || '请求失败');
      return Promise.reject(new Error(msg || '请求失败'));
    }
  },
  (error) => {
    // 1. 移除取消请求标识
    if (error.config) {
      removePendingRequest(error.config);
      // 关闭加载中
      if (error.config.loading) {
        // ElLoading.service().close();
      }
    }

    // 2. 错误分类处理
    let errorMsg = '';
    if (axios.isCancel(error)) {
      // 取消请求（重复请求）
      console.log('请求已取消：', error.message);
      return Promise.reject(error);
    } else if (error.message.includes('timeout')) {
      // 超时错误
      errorMsg = '请求超时，请检查网络！';
    } else if (error.response) {
      // HTTP状态码错误（4xx/5xx）
      const status = error.response.status;
      switch (status) {
        case 400:
          errorMsg = '请求参数错误！';
          break;
        case 403:
          errorMsg = '权限不足，无法访问！';
          break;
        case 404:
          errorMsg = '请求接口不存在！';
          break;
        case 500:
          errorMsg = '服务器内部错误，请稍后重试！';
          break;
        default:
          errorMsg = `请求失败（${status}）：${error.response.data?.msg || '未知错误'}`;
      }
    } else {
      // 网络错误（无响应）
      errorMsg = '网络异常，请检查网络连接！';
    }

    // 3. 统一错误提示
    ElMessage.error(errorMsg);
    return Promise.reject(error);
  }
);

// ===================== 封装通用请求方法 =====================
/**
 * GET请求
 * @param {String} url 接口地址
 * @param {Object} params 请求参数
 * @param {Object} config 自定义配置（如loading: false）
 * @returns {Promise}
 */
export const get = (url, params = {}, config = {}) => {
  return service.get(url, { params, ...config });
};

/**
 * POST请求
 * @param {String} url 接口地址
 * @param {Object} data 请求体数据
 * @param {Object} config 自定义配置
 * @returns {Promise}
 */
export const post = (url, data = {}, config = {}) => {
  return service.post(url, data, config);
};

/**
 * PUT请求
 * @param {String} url 接口地址
 * @param {Object} data 请求体数据
 * @param {Object} config 自定义配置
 * @returns {Promise}
 */
export const put = (url, data = {}, config = {}) => {
  return service.put(url, data, config);
};

/**
 * DELETE请求
 * @param {String} url 接口地址
 * @param {Object} params 请求参数
 * @param {Object} config 自定义配置
 * @returns {Promise}
 */
export const del = (url, params = {}, config = {}) => {
  return service.delete(url, { params, ...config });
};

/**
 * 上传文件（FormData）
 * @param {String} url 接口地址
 * @param {FormData} formData 表单数据（含文件）
 * @param {Object} config 自定义配置（如进度回调）
 * @returns {Promise}
 */
export const upload = (url, formData, config = {}) => {
  return service.post(url, formData, {
    headers: { 'Content-Type': 'multipart/form-data' },
    timeout: 60000, // 上传超时时间延长
    ...config
  });
};

// 导出默认实例（保留原生能力，按需使用）
export default service;
```

### 核心功能说明
#### 1. 基础配置
+ 基于环境变量配置 `baseURL`（适配 Vite 项目，Webpack 可替换为 `process.env.VUE_APP_API_BASE_URL`）；
+ 默认超时时间 10s，可通过请求配置覆盖；
+ 统一设置 `Content-Type` 为 JSON 格式。

#### 2. 重复请求取消
+ 用 `Map` 存储请求标识和取消令牌，避免同一时间重复发起相同请求；
+ 请求发起前检查是否已有相同请求，有则取消旧请求，保证请求唯一性。

#### 3. 请求拦截器
+ 自动注入 Token（适配 Vue3 Pinia，可替换为 Vuex/`localStorage`）；
+ 处理 GET 请求参数（移除 `undefined` 参数，避免请求异常）；
+ 支持自定义加载中状态（默认开启，可通过请求配置关闭）。

#### 4. 响应拦截器
+ 统一解析后端返回格式（`{ code, msg, data }`），成功时直接返回 `data`，简化上层调用；
+ 针对 401（登录过期）做统一登出处理；
+ 分类处理错误：取消请求、超时、HTTP 状态码、网络异常，给出精准提示。

#### 5. 通用方法封装
+ 封装 `get/post/put/del` 常用方法，简化调用；
+ 单独封装 `upload` 方法，适配文件上传（调整请求头、延长超时时间）。

### 使用示例
```javascript
import { get, post, upload } from '@/utils/request';

// 1. GET请求
const getUserList = async (page, size) => {
  try {
    const data = await get('/user/list', { page, size }, { loading: false });
    console.log('用户列表：', data);
  } catch (error) {
    console.error('请求失败：', error);
  }
};

// 2. POST请求
const addUser = async (userInfo) => {
  try {
    const data = await post('/user/add', userInfo);
    ElMessage.success('添加成功');
  } catch (error) {
    // 无需手动提示，request.js已统一处理
  }
};

// 3. 文件上传
const uploadFile = async (file) => {
  const formData = new FormData();
  formData.append('file', file);
  try {
    const data = await upload('/file/upload', formData, {
      onUploadProgress: (progressEvent) => {
        const progress = (progressEvent.loaded / progressEvent.total) * 100;
        console.log('上传进度：', progress);
      }
    });
    ElMessage.success('上传成功');
  } catch (error) {
    // 异常处理
  }
};
```

### 扩展与适配
1. **UI 库替换**：若不用 Element Plus，可将 `ElMessage/ElMessageBox` 替换为其他 UI 库的提示组件（如 Ant Design Vue 的 `message/modal`），或自定义提示函数。
2. **状态管理适配**：Token 存储若用 Vuex，替换 `useUserStore` 为 `store.dispatch/store.getters`；若用本地存储，直接读取 `localStorage.getItem('token')`。
3. **后端返回格式调整**：若后端返回格式不是 `{ code, msg, data }`，修改响应拦截器中的解析逻辑即可。
4. **请求加载中**：可集成 `ElLoading`（Element Plus）或自定义加载组件，在请求拦截器开启、响应拦截器关闭。
5. **更多配置**：可扩展请求头、代理、SSL 验证等，直接在 `axios.create` 或请求配置中添加。

### 核心原则
+ **统一化**：请求/响应拦截统一处理通用逻辑（Token、错误、加载中），减少重复代码；
+ **可扩展**：保留自定义配置入口，支持覆盖默认行为；
+ **鲁棒性**：处理各类异常场景（重复请求、超时、网络错误、业务错误），避免页面崩溃；
+ **易用性**：封装通用方法，上层调用只需关注接口地址和参数，无需关心底层逻辑。


