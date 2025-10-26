---
cover: https://tncache1-f1.v3mh.com/image/2025/10/26/8ae14bd9ed5e1c98cbda9dc0872a1470.jpg
title: Java线程复习
date: 2025-10-15 18:00:00
categories: Java编程技术
description: 线程是 Java 并发编程的核心，理解它对于编写高性能、响应式的应用程序至关重要。
---

好的，我们来系统地复习 Java 线程相关的知识。线程是 Java 并发编程的核心，理解它对于编写高性能、响应式的应用程序至关重要。

---

### 第一部分：核心概念

#### 1. 进程 vs 线程

**进程**：
- 操作系统资源分配的基本单位
- 每个进程有独立的内存空间（堆、栈、代码区）
- 进程间通信（IPC）比较复杂（如管道、信号、共享内存等）
- 创建和销毁开销较大

**线程**：
- CPU 调度的基本单位，是进程内的执行单元
- 同一进程的线程共享堆和方法区，但有独立的程序计数器和栈
- 线程间通信相对简单（共享内存）
- 创建和销毁开销较小

#### 2. 线程的生命周期（状态）

Java 线程有 6 种状态：
1. **NEW**（新建）：线程被创建但尚未启动
2. **RUNNABLE**（可运行）：正在运行或准备运行
3. **BLOCKED**（阻塞）：等待监视器锁（synchronized）
4. **WAITING**（等待）：无限期等待其他线程的特定动作
5. **TIMED_WAITING**（超时等待）：有限时间等待
6. **TERMINATED**（终止）：线程执行完毕

#### 3. 创建线程的三种方式

**1. 继承 Thread 类**
```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("线程执行: " + Thread.currentThread().getName());
    }
}

// 使用
MyThread thread = new MyThread();
thread.start();
```

**2. 实现 Runnable 接口**（推荐）
```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("线程执行: " + Thread.currentThread().getName());
    }
}

// 使用
Thread thread = new Thread(new MyRunnable());
thread.start();
```

**3. 实现 Callable 接口**（可返回结果）
```java
class MyCallable implements Callable<String> {
    @Override
    public String call() throws Exception {
        Thread.sleep(1000);
        return "任务完成: " + Thread.currentThread().getName();
    }
}
```

#### 4. 线程同步与锁

**关键问题**：多个线程访问共享资源时可能出现**竞态条件**。

**解决方案**：
1. **synchronized 关键字**
   - 同步方法
   - 同步代码块

2. **Lock 接口**（更灵活）
   - `ReentrantLock`：可重入锁
   - `ReadWriteLock`：读写锁

#### 5. 线程通信

- `wait()`：让当前线程等待
- `notify()`：唤醒一个等待的线程
- `notifyAll()`：唤醒所有等待的线程

#### 6. volatile 关键字

保证变量的可见性，但不保证原子性。

---

### 第二部分：实际应用场景

#### 场景 1：多线程下载文件

```java
public class FileDownloader {
    private static final int THREAD_COUNT = 4;
    
    public void downloadFile(String fileUrl, long fileSize) {
        ExecutorService executor = Executors.newFixedThreadPool(THREAD_COUNT);
        List<Future<Boolean>> futures = new ArrayList<>();
        
        // 计算每个线程下载的字节范围
        long chunkSize = fileSize / THREAD_COUNT;
        
        for (int i = 0; i < THREAD_COUNT; i++) {
            long startByte = i * chunkSize;
            long endByte = (i == THREAD_COUNT - 1) ? fileSize - 1 : (i + 1) * chunkSize - 1;
            
            Callable<Boolean> downloadTask = new DownloadTask(fileUrl, startByte, endByte, "part-" + i);
            futures.add(executor.submit(downloadTask));
        }
        
        // 等待所有任务完成
        for (Future<Boolean> future : futures) {
            try {
                if (!future.get()) {
                    System.out.println("某个下载任务失败");
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        
        executor.shutdown();
        System.out.println("文件下载完成");
    }
}

class DownloadTask implements Callable<Boolean> {
    private String url;
    private long startByte;
    private long endByte;
    private String partName;
    
    public DownloadTask(String url, long startByte, long endByte, String partName) {
        this.url = url;
        this.startByte = startByte;
        this.endByte = endByte;
        this.partName = partName;
    }
    
    @Override
    public Boolean call() throws Exception {
        System.out.println(Thread.currentThread().getName() + " 下载范围: " + startByte + "-" + endByte);
        // 模拟下载过程
        Thread.sleep(2000 + (long)(Math.random() * 2000));
        System.out.println(Thread.currentThread().getName() + " 完成下载: " + partName);
        return true;
    }
}
```

#### 场景 2：生产者-消费者模式

```java
public class ProducerConsumerExample {
    public static void main(String[] args) {
        MessageQueue queue = new MessageQueue(5);
        
        // 创建生产者
        Thread producer = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    queue.produce("消息-" + i);
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });
        
        // 创建消费者
        Thread consumer = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    String message = queue.consume();
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });
        
        producer.start();
        consumer.start();
    }
}

class MessageQueue {
    private final String[] items;
    private int putIndex;
    private int takeIndex;
    private int count;
    
    public MessageQueue(int capacity) {
        items = new String[capacity];
    }
    
    public synchronized void produce(String message) throws InterruptedException {
        while (count == items.length) {
            wait(); // 队列满，等待
        }
        
        items[putIndex] = message;
        putIndex = (putIndex + 1) % items.length;
        count++;
        System.out.println("生产: " + message + " [队列大小: " + count + "]");
        notifyAll(); // 通知消费者
    }
    
    public synchronized String consume() throws InterruptedException {
        while (count == 0) {
            wait(); // 队列空，等待
        }
        
        String message = items[takeIndex];
        takeIndex = (takeIndex + 1) % items.length;
        count--;
        System.out.println("消费: " + message + " [队列大小: " + count + "]");
        notifyAll(); // 通知生产者
        return message;
    }
}
```

#### 场景 3：使用线程池处理批量任务

```java
public class ThreadPoolExample {
    public static void main(String[] args) {
        // 创建线程池
        ExecutorService executor = Executors.newFixedThreadPool(3);
        List<Future<Integer>> results = new ArrayList<>();
        
        // 提交任务
        for (int i = 1; i <= 10; i++) {
            final int taskId = i;
            Callable<Integer> task = () -> {
                System.out.println("执行任务 " + taskId + " 线程: " + Thread.currentThread().getName());
                Thread.sleep(1000); // 模拟任务执行
                return taskId * taskId; // 返回平方值
            };
            results.add(executor.submit(task));
        }
        
        // 获取结果
        for (Future<Integer> result : results) {
            try {
                System.out.println("任务结果: " + result.get());
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        
        executor.shutdown();
    }
}
```

---

### 第三部分：常用的线程使用模式

#### 1. 线程池的最佳实践

```java
// 推荐使用 ThreadPoolExecutor 自定义参数
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    5, // 核心线程数
    10, // 最大线程数
    60L, // 空闲线程存活时间
    TimeUnit.SECONDS, // 时间单位
    new ArrayBlockingQueue<>(100), // 工作队列
    new ThreadPoolExecutor.CallerRunsPolicy() // 拒绝策略
);
```

#### 2. 使用 CompletableFuture 进行异步编程

```java
public class CompletableFutureExample {
    public static void main(String[] args) throws Exception {
        // 异步执行任务
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new IllegalStateException(e);
            }
            return "任务完成";
        });
        
        // 任务完成后执行回调
        future.thenAccept(result -> {
            System.out.println("收到结果: " + result);
        });
        
        // 组合多个异步任务
        CompletableFuture<String> combined = future
            .thenApply(result -> result + "，开始新任务")
            .thenApply(String::toUpperCase);
        
        System.out.println("最终结果: " + combined.get());
    }
}
```

#### 3. 使用 volatile 保证可见性

```java
public class VolatileExample {
    private volatile boolean running = true;
    
    public void stop() {
        running = false;
    }
    
    public void work() {
        while (running) {
            // 执行工作
            System.out.println("工作中...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        System.out.println("工作结束");
    }
}
```

#### 4. 使用 Atomic 类保证原子性

```java
public class AtomicExample {
    private AtomicInteger counter = new AtomicInteger(0);
    
    public void increment() {
        counter.incrementAndGet();
    }
    
    public int getCount() {
        return counter.get();
    }
}
```

### 重要注意事项

1. **避免死锁**：按固定顺序获取锁
2. **避免线程泄漏**：确保使用线程池并正确关闭
3. **处理异常**：在 Runnable 的 run 方法中捕获所有异常
4. **资源清理**：使用 try-finally 或 try-with-resources 确保资源释放
5. **性能考虑**：避免过度同步，尽量使用局部变量

### 总结

Java 线程编程的核心要点：
- **理解线程生命周期**和状态转换
- **掌握同步机制**：synchronized 和 Lock
- **熟练使用线程池**管理线程资源
- **了解现代并发工具**：CompletableFuture、Atomic 类
- **遵循最佳实践**：避免死锁、正确处理异常

通过实际场景的练习，你可以更好地理解这些概念并在实际项目中正确应用它们。