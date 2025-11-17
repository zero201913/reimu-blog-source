---
cover: https://image.acg.lol/file/2025/11/09/reimu-3.jpg
title: Spring XML 注入的系统复习
date: 2025-11-2 19:30:00
categories: Java框架
excerpt: Setter 方法注入 - 通过 <property> 标签配置，构造器注入 - 通过 <constructor-arg> 标签配置，复杂类型注入 - 包括 List、Map、Properties 等集合类型，特殊值处理 - null 值、空字符串、特殊字符转义等，引用类型注入 - Bean 之间的依赖关系配置
---


# Spring XML 配置 Bean 依赖注入

## 1. 环境配置

### 1.1 添加 Spring 依赖
```xml
<!-- Maven 依赖 -->
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.21</version>
    </dependency>
</dependencies>
```

### 1.2 创建 Spring 配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <!-- Bean 配置将在这里定义 -->
    
</beans>
```

## 2. 获取 Bean 的方式

### 2.1 ApplicationContext 获取 Bean
```java
// 加载配置文件
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

// 获取 Bean 的几种方式
UserService userService1 = (UserService) context.getBean("userService");
UserService userService2 = context.getBean("userService", UserService.class);
UserService userService3 = context.getBean(UserService.class);
```

### 2.2 BeanFactory 获取 Bean
```java
BeanFactory factory = new XmlBeanFactory(new ClassPathResource("applicationContext.xml"));
UserService userService = factory.getBean("userService", UserService.class);
```

## 3. 依赖 Setter 注入

### 3.1 基本 Setter 注入
```java
public class UserService {
    private UserDao userDao;
    private String serviceName;
    
    // Setter 方法
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
    
    public void setServiceName(String serviceName) {
        this.serviceName = serviceName;
    }
}
```

```xml
<bean id="userDao" class="com.example.UserDaoImpl" />

<bean id="userService" class="com.example.UserService">
    <!-- 引用类型注入 -->
    <property name="userDao" ref="userDao"/>
    <!-- 基本类型注入 -->
    <property name="serviceName" value="用户服务"/>
</bean>
```

### 3.2 级联 Setter 注入
```java
public class Company {
    private Department department;
    private String companyName;
    
    public void setDepartment(Department department) {
        this.department = department;
    }
    
    public void setCompanyName(String companyName) {
        this.companyName = companyName;
    }
    
    // 级联设置需要 getter 方法
    public Department getDepartment() {
        return department;
    }
}

public class Department {
    private String deptName;
    
    public void setDeptName(String deptName) {
        this.deptName = deptName;
    }
}
```

```xml
<bean id="company" class="com.example.Company">
    <property name="companyName" value="科技公司"/>
    <property name="department" ref="department"/>
    <!-- 级联赋值 -->
    <property name="department.deptName" value="技术部"/>
</bean>

<bean id="department" class="com.example.Department"/>
```

## 4. 依赖构造器注入

### 4.1 按参数顺序注入
```java
public class OrderService {
    private OrderDao orderDao;
    private String serviceType;
    
    public OrderService(OrderDao orderDao, String serviceType) {
        this.orderDao = orderDao;
        this.serviceType = serviceType;
    }
}
```

```xml
<bean id="orderDao" class="com.example.OrderDaoImpl"/>

<bean id="orderService" class="com.example.OrderService">
    <!-- 按构造器参数顺序 -->
    <constructor-arg ref="orderDao"/>
    <constructor-arg value="普通订单服务"/>
</bean>
```

### 4.2 按参数名称注入
```xml
<bean id="orderService" class="com.example.OrderService">
    <!-- 按参数名称 -->
    <constructor-arg name="orderDao" ref="orderDao"/>
    <constructor-arg name="serviceType" value="VIP订单服务"/>
</bean>
```

### 4.3 按参数类型注入
```xml
<bean id="orderService" class="com.example.OrderService">
    <!-- 按参数类型 -->
    <constructor-arg type="com.example.OrderDao" ref="orderDao"/>
    <constructor-arg type="java.lang.String" value="紧急订单服务"/>
</bean>
```

### 4.4 按参数索引注入
```xml
<bean id="orderService" class="com.example.OrderService">
    <!-- 按参数索引 -->
    <constructor-arg index="0" ref="orderDao"/>
    <constructor-arg index="1" value="批量订单服务"/>
</bean>
```

## 5. 依赖注入特殊值

### 5.1 注入 null 值
```xml
<bean id="exampleBean" class="com.example.ExampleBean">
    <property name="name">
        <null/>
    </property>
</bean>
```

### 5.2 注入空字符串
```xml
<bean id="exampleBean" class="com.example.ExampleBean">
    <property name="name" value=""/>
</bean>
```

### 5.3 注入特殊符号
```xml
<bean id="exampleBean" class="com.example.ExampleBean">
    <!-- 使用 CDATA 处理特殊字符 -->
    <property name="description">
        <value><![CDATA[这是包含<特殊>字符&的文本]]></value>
    </property>
    
    <!-- 使用转义字符 -->
    <property name="content" value="这是包含&lt;特殊&gt;字符&amp;的文本"/>
</bean>
```

## 6. 依赖注入自定义对象的形参

### 6.1 引用其他 Bean
```java
public class Car {
    private Engine engine;
    private Wheel wheel;
    
    public Car(Engine engine, Wheel wheel) {
        this.engine = engine;
        this.wheel = wheel;
    }
}
```

```xml
<bean id="engine" class="com.example.Engine"/>
<bean id="wheel" class="com.example.Wheel"/>

<bean id="car" class="com.example.Car">
    <constructor-arg ref="engine"/>
    <constructor-arg ref="wheel"/>
</bean>
```

### 6.2 内部 Bean
```xml
<bean id="outerBean" class="com.example.OuterBean">
    <property name="innerBean">
        <!-- 内部 Bean，只能在当前 Bean 中使用 -->
        <bean class="com.example.InnerBean">
            <property name="name" value="内部Bean"/>
        </bean>
    </property>
</bean>
```

## 7. 依赖注入 List

### 7.1 注入基本类型 List
```java
public class CollectionBean {
    private List<String> names;
    private List<Integer> numbers;
    
    public void setNames(List<String> names) {
        this.names = names;
    }
    
    public void setNumbers(List<Integer> numbers) {
        this.numbers = numbers;
    }
}
```

```xml
<bean id="collectionBean" class="com.example.CollectionBean">
    <property name="names">
        <list>
            <value>张三</value>
            <value>李四</value>
            <value>王五</value>
        </list>
    </property>
    
    <property name="numbers">
        <list>
            <value>1001</value>
            <value>1002</value>
            <value>1003</value>
        </list>
    </property>
</bean>
```

### 7.2 注入引用类型 List
```java
public class Team {
    private List<Member> members;
    
    public void setMembers(List<Member> members) {
        this.members = members;
    }
}
```

```xml
<bean id="team" class="com.example.Team">
    <property name="members">
        <list>
            <ref bean="member1"/>
            <ref bean="member2"/>
            <bean class="com.example.Member">
                <property name="name" value="临时成员"/>
            </bean>
        </list>
    </property>
</bean>

<bean id="member1" class="com.example.Member">
    <property name="name" value="成员一"/>
</bean>

<bean id="member2" class="com.example.Member">
    <property name="name" value="成员二"/>
</bean>
```

## 8. 依赖注入 Map

### 8.1 注入基本类型 Map
```java
public class ConfigBean {
    private Map<String, String> configMap;
    private Map<String, Integer> settings;
    
    public void setConfigMap(Map<String, String> configMap) {
        this.configMap = configMap;
    }
    
    public void setSettings(Map<String, Integer> settings) {
        this.settings = settings;
    }
}
```

```xml
<bean id="configBean" class="com.example.ConfigBean">
    <property name="configMap">
        <map>
            <entry key="url" value="http://example.com"/>
            <entry key="username" value="admin"/>
            <entry key="password" value="123456"/>
        </map>
    </property>
    
    <property name="settings">
        <map>
            <entry key="timeout" value="3000"/>
            <entry key="retryCount" value="3"/>
            <entry key="maxConnections" value="100"/>
        </map>
    </property>
</bean>
```

### 8.2 注入引用类型 Map
```java
public class ServiceManager {
    private Map<String, Service> services;
    
    public void setServices(Map<String, Service> services) {
        this.services = services;
    }
}
```

```xml
<bean id="serviceManager" class="com.example.ServiceManager">
    <property name="services">
        <map>
            <entry key="userService">
                <ref bean="userService"/>
            </entry>
            <entry key="orderService">
                <ref bean="orderService"/>
            </entry>
            <entry key="paymentService">
                <bean class="com.example.PaymentService">
                    <property name="name" value="支付服务"/>
                </bean>
            </entry>
        </map>
    </property>
</bean>
```

## 9. 其他集合类型注入

### 9.1 Set 注入
```xml
<property name="uniqueNames">
    <set>
        <value>唯一值1</value>
        <value>唯一值2</value>
        <value>唯一值1</value> <!-- 重复值会被忽略 -->
    </set>
</property>
```

### 9.2 Properties 注入
```xml
<property name="properties">
    <props>
        <prop key="driver">com.mysql.jdbc.Driver</prop>
        <prop key="url">jdbc:mysql://localhost:3306/test</prop>
        <prop key="username">root</prop>
        <prop key="password">123456</prop>
    </props>
</property>
```

### 9.3 Array 注入
```xml
<property name="array">
    <array>
        <value>数组元素1</value>
        <value>数组元素2</value>
        <value>数组元素3</value>
    </array>
</property>
```

## 10. 实用示例

### 10.1 完整的配置示例
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 基础 Bean -->
    <bean id="userDao" class="com.example.UserDaoImpl"/>
    
    <!-- Setter 注入 -->
    <bean id="userService" class="com.example.UserService">
        <property name="userDao" ref="userDao"/>
        <property name="serviceName" value="用户管理服务"/>
    </bean>
    
    <!-- 构造器注入 -->
    <bean id="orderService" class="com.example.OrderService">
        <constructor-arg name="orderDao" ref="orderDao"/>
        <constructor-arg name="serviceType" value="标准服务"/>
    </bean>
    
    <!-- 复杂对象注入 -->
    <bean id="systemConfig" class="com.example.SystemConfig">
        <property name="settings">
            <map>
                <entry key="env" value="production"/>
                <entry key="version" value="1.0.0"/>
            </map>
        </property>
        <property name="services">
            <list>
                <ref bean="userService"/>
                <ref bean="orderService"/>
            </list>
        </property>
    </bean>

</beans>
```

这个复习指南涵盖了 Spring XML 配置中 Bean 依赖注入的主要方式，包括基本配置、各种注入方式以及复杂数据类型的处理。