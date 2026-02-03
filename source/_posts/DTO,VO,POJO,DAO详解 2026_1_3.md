---
cover: https://image.acg.lol/file/2025/11/09/reimu-9.jpg
title: DTO、VO、POJO、DAO详解：企业级应用分层架构的数据对象
date: 2026-1-3 09:30:00
categories: 架构设计
excerpt: 本文详细介绍了企业级应用分层架构中常见的数据对象（DTO、VO、PO、DO、BO、POJO、DAO）的定义、作用、典型场景和核心区别，帮助开发者理解这些对象在分层架构中的职责边界，实现解耦分层、数据语义清晰的系统设计。
---

### 一、核心前提：这些“O”的定义来源
这些以“O”结尾的对象（DTO/VO/PO等）**并非由官方标准组织（如ISO、W3C）强制定义**，而是：

1. 核心源于**企业级应用分层开发的行业共识/设计模式**，最早由Martin Fowler（马丁·福勒）在《Patterns of Enterprise Application Architecture》（企业应用架构模式，EAA）中明确提出DTO、DAO等核心概念；
2. 后续VO、BO、PO/DO等是Java EE生态结合分层架构（表现层、业务层、持久层）的衍生概念，再经领域驱动设计（DDD）、ORM框架（MyBatis/Hibernate）实践演化；
3. 不同企业/团队会根据自身场景微调命名/边界，但核心目标一致——**解耦分层、数据语义清晰、降低维护成本**。