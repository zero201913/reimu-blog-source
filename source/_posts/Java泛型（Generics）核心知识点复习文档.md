---
cover: https://image.acg.lol/file/2025/11/09/reimu-6.jpg
title: Java 泛型
date: 2025-11-7 9:13:00
categories: Java编程语言
excerpt: 泛型是Java SE 5引入的特性，允许在定义类、接口、方法时使用类型参数（Type Parameter），将数据类型的确定推迟到创建对象或调用方法时。其本质是**参数化类型**，即把类型作为参数传递。
---

本文基于B站视频《Java中的泛型 Generics in Java》（https://www.bilibili.com/video/BV1H94y1a7bJ?vd_source=8150b5e85e95dc05f0c21be01f3d1712）内容整理，结合代码示例系统总结泛型的核心概念、使用方法、语法规则及高级特性，为Java泛型知识点复习提供清晰参考。

## 一、泛型的基本概念与核心价值

### 1. 什么是泛型？

泛型是Java SE 5引入的特性，允许在定义类、接口、方法时使用类型参数（Type Parameter），将数据类型的确定推迟到创建对象或调用方法时。其本质是**参数化类型**，即把类型作为参数传递。

### 2. 为什么需要泛型？

在泛型出现前，使用`Object`类型存储任意数据会存在以下问题，泛型恰好解决了这些痛点：

- **类型不安全**：编译期无法检查类型，运行时可能出现`ClassCastException`。例如使用`ArrayList`存储字符串却存入整数，编译不报错，运行时取数据强转才报错。

- **代码冗余**：需要频繁进行类型转换，代码可读性差。

泛型的核心价值：**编译期类型检查** + **消除类型转换**，提升代码的安全性和可维护性。

## 二、泛型的三种使用形式

### 1. 泛型类（Generic Class）

在类定义时声明类型参数，类型参数在类的成员变量、方法返回值、方法参数中使用。

```java


// 定义泛型类，T为类型参数（通常用大写字母表示，如T、E、K、V）
public class GenericClass<T> {
    // 泛型成员变量
    private T data;

    // 构造方法
    public GenericClass(T data) {
        this.data = data;
    }

    // 泛型方法（返回值为T类型）
    public T getData() {
        return data;
    }

    public static void main(String[] args) {
        // 创建泛型类对象时指定类型参数为String
        GenericClass<String> stringObj = new GenericClass<>("Hello Generics");
        System.out.println(stringObj.getData()); // 输出：Hello Generics

        // 指定类型参数为Integer
        GenericClass<Integer> intObj = new GenericClass<>(123);
        System.out.println(intObj.getData()); // 输出：123
    }
}
    
```

**注意**：泛型类的类型参数不能是基本数据类型（如int、char），必须使用包装类（如Integer、Character）。

### 2. 泛型接口（Generic Interface）

与泛型类类似，在接口定义时声明类型参数，实现接口时需指定具体类型或继续保留泛型。

```java


// 定义泛型接口
public interface GenericInterface<E> {
    void add(E element);
    E get(int index);
}

// 实现泛型接口时指定具体类型（String）
public class StringList implements GenericInterface<String> {
    private List<String> list = new ArrayList<>();

    @Override
    public void add(String element) {
        list.add(element);
    }

    @Override
    public String get(int index) {
        return list.get(index);
    }
}

// 实现泛型接口时继续保留泛型（泛型类实现泛型接口）
public class GenericList<E> implements GenericInterface<E> {
    private List<E> list = new ArrayList<>();

    @Override
    public void add(E element) {
        list.add(element);
    }

    @Override
    public E get(int index) {
        return list.get(index);
    }
}
    
```

### 3. 泛型方法（Generic Method）

在方法声明时独立声明类型参数（与类的泛型无关），可以是静态方法或实例方法。

```java


public class GenericMethodDemo {
    // 泛型方法：<T>是类型参数声明，T是返回值类型
    public static <T> T getFirstElement(List<T> list) {
        if (list != null && !list.isEmpty()) {
            return list.get(0);
        }
        return null;
    }

    public static void main(String[] args) {
        List<String> strList = Arrays.asList("A", "B", "C");
        String firstStr = getFirstElement(strList);
        System.out.println(firstStr); // 输出：A

        List<Integer> intList = Arrays.asList(1, 2, 3);
        Integer firstInt = getFirstElement(intList);
        System.out.println(firstInt); // 输出：1
    }
}
    
```

**关键标识**：泛型方法必须在返回值类型前加上`<类型参数>`，这是区分泛型方法与普通方法的核心标志。

## 三、泛型的通配符（Wildcard）

当不确定泛型的具体类型时，可使用通配符`?`来表示未知类型，通配符主要有三种形式：无界通配符、上界通配符、下界通配符。

### 1. 无界通配符（Unbounded Wildcard）：`?`

表示任意类型，仅能读取数据，不能写入数据（除了`null`）。适用于只需要获取对象信息，不关心具体类型的场景。

```java


public class UnboundedWildcardDemo {
    public static void printList(List<?> list) {
        for (Object obj : list) {
            System.out.println(obj);
        }
        // list.add("test"); // 编译错误：无法确定类型，不能写入非null值
        list.add(null); // 允许写入null
    }

    public static void main(String[] args) {
        List<String> strList = Arrays.asList("Java", "Python");
        List<Integer> intList = Arrays.asList(100, 200);
        
        printList(strList); // 输出：Java Python
        printList(intList); // 输出：100 200
    }
}
    
```

### 2. 上界通配符（Upper Bounded Wildcard）：`? extends 父类`

表示未知类型是“父类”或其子类，同样只能读取数据（读取的数据类型为父类），不能写入数据。适用于需要获取“父类及其子类”对象的场景。

```java


class Animal {}
class Dog extends Animal {}
class Cat extends Animal {}

public class UpperBoundedWildcardDemo {
    public static void printAnimalList(List<? extends Animal> animalList) {
        for (Animal animal : animalList) {
            System.out.println(animal.getClass().getSimpleName());
        }
        // animalList.add(new Dog()); // 编译错误：无法确定具体是Dog还是Cat，不能写入
    }

    public static void main(String[] args) {
        List<Dog> dogList = Arrays.asList(new Dog(), new Dog());
        List<Cat> catList = Arrays.asList(new Cat(), new Cat());
        
        printAnimalList(dogList); // 输出：Dog Dog
        printAnimalList(catList); // 输出：Cat Cat
    }
}
    
```

### 3. 下界通配符（Lower Bounded Wildcard）：`? super 子类`

表示未知类型是“子类”或其父类，允许写入“子类及其子类”的数据，读取的数据类型为`Object`。适用于需要写入“子类及其子类”对象的场景。

```java


public class LowerBoundedWildcardDemo {
    public static void addDog(List<? super Dog> list) {
        list.add(new Dog()); // 允许写入Dog（子类）
        list.add(new Dog() {}); // 允许写入Dog的匿名子类
        // list.add(new Animal()); // 编译错误：只能写入Dog及其子类
    }

    public static void main(String[] args) {
        List<Animal> animalList = new ArrayList<>();
        List<Dog> dogList = new ArrayList<>();
        
        addDog(animalList);
        addDog(dogList);
        
        System.out.println(animalList.size()); // 输出：2
        System.out.println(dogList.size()); // 输出：2
    }
}
    
```

## 四、泛型的类型擦除（Type Erasure）

### 1. 什么是类型擦除？

Java泛型是“编译期泛型”，在编译阶段会将泛型的类型参数信息擦除，替换为其边界类型（若无边界则替换为`Object`）。运行时JVM无法获取泛型的具体类型信息，这就是泛型的类型擦除机制。

### 2. 类型擦除的具体表现

```java


public class TypeErasureDemo {
    public static void main(String[] args) {
        List<String> strList = new ArrayList<>();
        List<Integer> intList = new ArrayList<>();
        
        // 编译后泛型类型被擦除，strList和intList的实际类型都是ArrayList
        System.out.println(strList.getClass() == intList.getClass()); // 输出：true
    }
}
    
```

擦除规则：

- 无边界泛型（如`<T>`）：擦除为`Object`。

- 有上界泛型（如`<T extends Animal>`）：擦除为上界类型`Animal`。

- 有多个上界泛型（如`<T extends Animal & Comparable>`）：擦除为第一个上界类型`Animal`。

### 3. 类型擦除的影响

- **不能创建泛型数组**：如`new ArrayList<String>[10]`编译错误，因为擦除后数组元素类型都是`ArrayList`，无法保证类型安全。

- **不能使用泛型类型参数创建对象**：如`new T()`编译错误，因为擦除后T变为Object，无法确定具体构造器。

- **不能使用泛型类型参数进行 instanceof 检查**：如`obj instanceof List<String>`编译错误，运行时无法获取泛型类型。

## 五、泛型的高级特性

### 1. 泛型与继承

泛型不支持协变，即`List<Dog>`不是`List<Animal>`的子类，即使`Dog`是`Animal`的子类。若需要实现类似功能，需使用上界通配符`List<? extends Animal>`。

```java


public class GenericInheritanceDemo {
    public static void main(String[] args) {
        List<Dog> dogList = new ArrayList<>();
        // List<Animal> animalList = dogList; // 编译错误：泛型不支持协变
        List<? extends Animal> animalList = dogList; // 正确：使用上界通配符
    }
}
    
```

### 2. 静态方法与泛型

静态方法不能使用类的泛型类型参数，若需要使用泛型，必须定义为泛型方法（独立声明类型参数）。

```java


public class StaticGenericDemo<T> {
    // 错误：静态方法不能使用类的泛型参数T
    // public static T getValue() { return null; }

    // 正确：定义为泛型方法
    public static <E> E getValue() { return null; }
}
    
```

### 3. 泛型异常

不能定义泛型异常类（即异常类不能使用泛型），也不能在`catch`块中使用泛型类型参数。但可以在方法声明中使用泛型作为抛出异常的类型参数。

```java


// 错误：不能定义泛型异常类
// public class GenericException<T> extends Exception {}

public class GenericThrowDemo {
    // 正确：方法声明中使用泛型作为抛出异常类型
    public static <T extends Exception> void throwException(T exception) throws T {
        throw exception;
    }

    public static void main(String[] args) throws Exception {
        throwException(new RuntimeException("泛型异常测试"));
    }
}
    
```

## 六、核心知识点总结

**泛型三要素**：泛型类、泛型接口、泛型方法，核心是类型参数的声明与使用。

**通配符使用场景**：无界通配符（只读任意类型）、上界通配符（只读父类及其子类）、下界通配符（写入子类及其子类）。

**类型擦除关键**：编译期擦除类型参数，运行时无泛型信息，导致泛型数组、泛型对象创建等限制。

**核心原则**：泛型的目的是提升类型安全和代码复用，使用时需遵循类型擦除带来的语法限制。