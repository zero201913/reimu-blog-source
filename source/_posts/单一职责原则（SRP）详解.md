---
cover: https://image.acg.lol/file/2025/11/09/reimu-1.jpg
title: 单一职责原则（SRP）详解
date: 2025-11-2 19:30:00
categories: 设计模式
excerpt: 一个类应该只有一个职责，或者说，只有一个引起它变化的原因。
---

# 单一职责原则（SRP）详解  
**—— 设计模式中的“一个类只干一件事”**

> **定义**：**一个类应该只有一个职责，或者说，只有一个引起它变化的原因。**  
> —— Robert C. Martin（《敏捷软件开发：原则、模式与实践》）

---

## 一、理论解释

| 关键词 | 含义 |
|-------|------|
| **职责（Responsibility）** | 一个类“应该做”的事情 |
| **单一职责** | 一个类只负责**一个功能领域**的变化 |
| **变化原因** | 如果一个类因 **>1 个原因**需要修改 → 违反 SRP |

### 为什么要遵循 SRP？
| 好处 | 说明 |
|------|------|
| **高内聚** | 相关代码集中 |
| **低耦合** | 修改一个职责不影响其他 |
| **可维护性↑** | 改 bug、加功能更安全 |
| **可测试性↑** | 职责清晰，单元测试简单 |
| **可复用性↑** | 单一职责类更容易被复用 |

---

## 二、违反 SRP 的坏例子（Java）

```java
// 坏例子：一个类干了三件事
public class Employee {
    private String name;
    private double salary;

    // 1. 管理员工数据
    public void setName(String name) { this.name = name; }
    public void setSalary(double salary) { this.salary = salary; }

    // 2. 计算薪水（业务逻辑）
    public double calculatePay() {
        return salary * 1.2; // 假设有奖金
    }

    // 3. 保存到数据库（持久化）
    public void saveToDatabase() {
        // 连接数据库、SQL 插入...
        System.out.println("Saving " + name + " to DB...");
    }

    // 4. 生成报表（输出）
    public void printReport() {
        System.out.println("Employee: " + name + ", Pay: " + calculatePay());
    }
}
```

> **违反 SRP 的地方**：
> - 改薪资规则 → 要改 `calculatePay`
> - 换数据库 → 要改 `saveToDatabase`
> - 改报表格式 → 要改 `printReport`
> - **一个类有 3 个变化原因 → 高风险！**

---

## 三、符合 SRP 的重构方案

```java
// 1. 员工数据（仅负责存储）
public class Employee {
    private String name;
    private double salary;

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }
    // 仅 getter
    public String getName() { return name; }
    public double getSalary() { return salary; }
}

// 2. 薪资计算（仅负责业务规则）
public class PayrollCalculator {
    public double calculatePay(Employee emp) {
        return emp.getSalary() * 1.2;
    }
}

// 3. 数据库持久化（仅负责存储）
public class EmployeeRepository {
    public void save(Employee emp) {
        System.out.println("Saving " + emp.getName() + " to DB...");
    }
}

// 4. 报表生成（仅负责输出）
public class EmployeeReport {
    public void print(Employee emp, PayrollCalculator calc) {
        System.out.println("Employee: " + emp.getName() +
            ", Pay: " + calc.calculatePay(emp));
    }
}
```

### 使用方式（客户端代码）

```java
public class Main {
    public static void main(String[] args) {
        Employee emp = new Employee("Alice", 5000);

        PayrollCalculator calc = new PayrollCalculator();
        EmployeeRepository repo = new EmployeeRepository();
        EmployeeReport report = new EmployeeReport();

        repo.save(emp);
        report.print(emp, calc);
    }
}
```

> **每个类只干一件事，变化独立！**

---

## 四、SRP 在实际项目中的体现

| 模块 | 推荐拆分 |
|------|---------|
| `UserService` | → `UserService`（业务） + `UserRepository`（存储） |
| `Order` | → `Order`（数据） + `OrderCalculator`（价格） + `OrderPrinter`（报表） |
| `EmailSender` | → 不应包含用户验证逻辑 |

---

## 五、常见误区

| 误区 | 正确认识 |
|------|---------|
| “一个类只有一个方法” | 不是！是**一个职责**，可有多个相关方法 |
| “SRP = 小类” | 不是！是**高内聚**，不是碎片化 |
| “接口也必须 SRP” | 正确！一个接口也应只代表一个职责 |

---

## 六、面试高频题

| 问题 | 标准答案 |
|------|--------|
| 什么是单一职责原则？ | 一个类只应该有一个职责，即只有一个引起它变化的原因 |
| 举例说明违反 SRP 的后果？ | 修改薪资逻辑可能意外破坏数据库存储逻辑 |
| 如何判断一个类是否违反 SRP？ | 问：“这个类会因为哪些**不同原因**被修改？” >1 → 违反 |
| SRP 和“函数要短”一样吗？ | 不一样！SRP 是**类/模块级别**，函数短是实现细节 |

---

## 七、一句话总结

> **“一个类，一个活儿；改它只因一件事儿。”**

---

## 八、记忆口诀

> **“职责要单一，类如专才；  
>  多能乱成麻，维护哭爹。”**

---

**文档结束**  
**适用人群**：Java 开发者、系统设计面试、架构入门  
**推荐实践**：每次写类时问自己：“它会因什么被改？”