---
cover: https://image.acg.lol/file/2025/11/09/reimu-8.jpg
title: Sa-Token鉴权学习：Vue结合SpringBoot实现角色权限控制
date: 2026-1-3 09:30:00
categories: 安全认证
excerpt: 本文详细介绍了如何在Vue框架中结合SpringBoot和Sa-Token实现不同身份（角色/权限）展示不同功能板块的完整方案，包括后端鉴权配置、前端状态管理、路由守卫、自定义权限指令等核心功能，帮助开发者快速掌握前后端协同的权限控制实现。
---

要实现 Vue 框架结合 Sa-Token 按**不同身份（角色/权限）展示不同功能板块**，核心是**前后端协同**：

+ 后端：基于 Sa-Token 完成登录认证、角色/权限的校验与管理；
+ 前端：通过 Sa-Token 返回的 Token 维持登录状态，结合路由守卫、权限指令控制功能板块的显示/隐藏。

以下是完整的实现方案（涵盖 Vue2/Vue3，以 Vue3 + Pinia 为例，Vue2 仅需替换状态管理为 Vuex 即可）。