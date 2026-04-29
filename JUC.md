# 基础

## 1.并发和并行

- **并发**指的是：**多个任务在同一段时间内交替执行**。
- **并行**指的是：**多个任务在同一时刻真正同时执行**。

## 2.同步和异步

- **同步**指的是：调用方发起一个任务后，需要等待这个任务执行完成，才能继续往下执行。
- **异步**指的是：调用方发起一个任务后，不需要一直等待任务执行完成，可以继续做其他事情。

## 3.创建线程的方法

```java
@Slf4j(topic = "c.test")
public class test {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 主线程
        log.debug("主线程启动了");

        // 方法1 继承 Thread 重新 run 方法，用调用 start 方法启动线程
        MyThread myThread = new MyThread();
        myThread.start();
        // Lambda 简写
        new Thread(() -> log.debug("线程启动了"), "t1").start();

        // 方法2 - 将任务和线程分离
        // 实现 Runnable 接口，重写 run 方法，传递给 Thread
        Runnable runnable = () -> log.debug("线程启动了");
        new Thread(runnable, "t2").start();
        // 简写
        new Thread(() -> log.debug("线程启动了"), "t3").start();

        // 方法3 - 实现 Callable 接口，重写 call 方法，传递给 Thread
        Callable callable = () -> {
            log.debug("线程启动了");
            Thread.sleep(1000);
            return "Callable";
        };
        FutureTask<String> task = new FutureTask<>(callable);
        new Thread(task, "t4").start();

        log.debug("主线程一秒后得到结果{}", task.get());
    }
}
22:46:28 [main] c.test - 主线程启动了
22:46:28 [Thread-1] c.MyThread - 线程启动了
22:46:28 [t1] c.test - 线程启动了
22:46:28 [t2] c.test - 线程启动了
22:46:28 [t3] c.test - 线程启动了
22:46:28 [t4] c.test - 线程启动了
22:46:29 [main] c.test - 一秒后得到结果Callable
```

## 4.常见方法

| 方法名           | static | 功能说明                                                     | 注意                                                         |
| ---------------- | ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| start()          |        | 启动一个新线程，在新的线程运行 run 方法中的代码              | start 方法只是让线程进入就绪，里面代码不一定立刻运行，因为 CPU 的时间片还没分给它。每个线程对象的 start 方法只能调用一次，如果调用多次会出现 IllegalThreadStateException |
| run()            |        | 新线程启动后会调用的方法                                     | 如果在构造 Thread 对象时传递了 Runnable 参数，则线程启动后会调用 Runnable 中的 run 方法，否则默认不执行任何操作。但可以创建 Thread 的子类对象，来覆盖默认行为 |
| join()           |        | 等待线程运行结束                                             |                                                              |
| join(long n)     |        | 等待线程运行结束，最多等待 n 毫秒                            |                                                              |
| getId()          |        | 获取线程长整型的 id                                          | id 唯一                                                      |
| getName()        |        | 获取线程名                                                   |                                                              |
| setName(String)  |        | 修改线程名                                                   |                                                              |
| getPriority()    |        | 获取线程优先级                                               |                                                              |
| setPriority(int) |        | 修改线程优先级                                               | Java 中规定线程优先级是 1~10 的整数，较大的优先级能提高该线程被 CPU 调度的概率 |
| getState()       |        | 获取线程状态                                                 | Java 中线程状态是用 6 个 enum 表示，分别为：NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED |
| isInterrupted()  |        | 判断线程是否被打断                                           | 不会清除打断标记                                             |
| isAlive()        |        | 判断线程是否存活                                             | 线程还没有运行完毕，就表示存活                               |
| interrupt()      |        | 打断线程                                                     | 如果被打断线程正在 sleep、wait、join，会导致被打断的线程抛出 InterruptedException，并清除打断标记；如果打断的是正在运行的线程，则会设置打断标记；park 的线程被打断，也会设置打断标记 |
| interrupted()    | static | 判断当前线程是否被打断                                       | 会清除打断标记                                               |
| currentThread()  | static | 获取当前正在执行的线程                                       |                                                              |
| sleep(long n)    | static | 让当前执行的线程休眠 n 毫秒，休眠时让出 CPU 的时间片给其他线程 |                                                              |
| yield()          | static | 提示线程调度器让出当前线程对 CPU 的使用                      | 主要是为了测试和调试                                         |

## 5.sleep 和 yield

**sleep**

1. 调用 `sleep` 会让**当前正在执行的线程**进入 `TIMED_WAITING` 状态。

2. `sleep` 需要指定休眠时间，例如： `Thread.sleep(1000); `
3. `sleep` 休眠期间会让出 CPU 时间片，CPU 可以去执行其他线程。
4. `sleep` **不会释放**当前线程已经持有的锁。
5. 如果线程在 `sleep` 期间被其他线程调用 `interrupt()` 打断，会抛出 `InterruptedException`。
6. `sleep` 时间到了以后，线程不会立刻执行，而是重新进入就绪状态，等待 CPU 调度。
7. `sleep` 是 `Thread` 类的静态方法，推荐写法是：`Thread.sleep(1000);`
8. 实际开发中，`sleep` 常用于模拟耗时、定时等待、控制任务执行频率等场景。

**yield**

1. 调用 `yield` 会让当前线程从运行状态重新回到 `RUNNABLE` 状态。
2. `yield` 的作用是提示线程调度器：当前线程愿意让出 CPU。
3. `yield` 只是一个提示，不保证当前线程一定会暂停。
4. 调用 `yield` 后，当前线程仍然可能马上再次被 CPU 调度执行。
5. `yield` 不需要指定等待时间。
6. `yield` **不会释放**当前线程已经持有的锁。
7. `yield` 不会抛出 `InterruptedException`。
8. `yield` 是 `Thread` 类的静态方法，写法是：`Thread.yield();`

**总结：**

- sleep：当前线程睡一会儿，到时间后再参与 CPU 竞争。

- yield：当前线程让一下 CPU，但不保证真的让成功。

## 6.线程优先级

setPriority(int)

- 在线程空闲时作用不大

## 7.interrupt

```java
public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(() -> {
        log.debug("开始执行");
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            log.debug(" interrupted");
        }
    }, "t1");
    t1.start();
    Thread.sleep(1000);
    // 打断正在 sleep join wait 的线程，会抛出 InterruptedException 并清除打断标记
    t1.interrupt();
    log.debug("t1 打断标记：{}", t1.isInterrupted());
    Thread t2 = new Thread(() -> {
        log.debug("开始执行");
        while (true) {
            boolean interrupted = Thread.currentThread().isInterrupted();
            if (interrupted) {
                log.debug("料理后事.....");
                log.debug("被打断，退出循环");
                break;
            }
        }
    }, "t2");
    t2.start();
    Thread.sleep(1000);
    // 打断正在运行的线程不会抛出（不会让线程停止） InterruptedException 也不会清除打断标记
    t2.interrupt();
    log.debug("t2 打断标记：{}", t2.isInterrupted());
}
09:36:18 [t1] c.interruptTest - 开始执行
09:36:19 [t1] c.interruptTest -  interrupted
09:36:19 [main] c.interruptTest - t1 打断标记：false
09:36:19 [t2] c.interruptTest - 开始执行
09:36:20 [t2] c.interruptTest - 料理后事.....
09:36:20 [t2] c.interruptTest - 被打断，退出循环
09:36:20 [main] c.interruptTest - t2 打断标记：true
```

## 8.打断 park 线程

```java
public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(() -> {
        log.debug("开始执行...");
        LockSupport.park();
        log.debug("被唤醒...");
        // log.debug("打断状态：{}", Thread.currentThread().isInterrupted());
        log.debug("打断状态：{}", Thread.interrupted());
        // 再次 park 不会停止，因为打断标记已经存在
        // 解决方法：在 park 之前清除打断标记 ， 使用 Thread.interrupted() 方法
        LockSupport.park();
        log.debug("被唤醒...");
    }, "t1");
    t1.start();
    Thread.sleep(2000);
    t1.interrupt();
}
13:54:58 [t1] c.parkTest - 开始执行...
13:55:00 [t1] c.parkTest - 被唤醒...
13:55:00 [t1] c.parkTest - 打断状态：true
阻塞................
```

## 9.守护线程

```java
public static void main(String[] args) {
    Thread t1 = new Thread(() -> {
        while (true) {
            log.debug("t1 is running");
        }
    }, "t1");
    // 如果不设置为守护线程，t1线程会一直运行，导致main线程无法退出
    // 必须在 start 之前调用
    t1.setDaemon(true);
    t1.start();
    log.debug("main is exiting");
}
```

**常见的守护线程：**

- 垃圾回收线程
- 引用清理 / 信号调度线程

## 10.线程状态

**从 Java API 层面说**

**根据 Thread .State 枚举，分为六种状态**

| 英文状态        | 含义                                                   | 常见进入方式                                                 | 常见退出方式                                                 |
| --------------- | ------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `NEW`           | 线程对象已经创建，但还没有调用 `start()` 方法          | `Thread t = new Thread()`                                    | 调用 `t.start()` 后进入 `RUNNABLE`                           |
| `RUNNABLE`      | 线程已经启动，可能正在运行，也可能在等待 CPU 时间片    | 调用 `start()`；从阻塞、等待、超时等待中恢复                 | 竞争锁失败进入 `BLOCKED`；调用等待方法进入 `WAITING/TIMED_WAITING`；执行结束进入 `TERMINATED` |
| `BLOCKED`       | 线程正在等待获取对象监视器锁，也就是 `synchronized` 锁 | 进入 `synchronized` 代码块或方法时，锁被其他线程占用         | 获取到锁后进入 `RUNNABLE`                                    |
| `WAITING`       | 线程无限期等待，需要其他线程显式唤醒                   | `Object.wait()`；`Thread.join()`；`LockSupport.park()`       | 被 `notify()` / `notifyAll()` 唤醒；被等待线程结束；`unpark()`；中断 |
| `TIMED_WAITING` | 线程在指定时间内等待，到时间后自动恢复                 | `Thread.sleep(ms)`；`Object.wait(ms)`；`Thread.join(ms)`；`LockSupport.parkNanos()`；`parkUntil()` | 时间到；被唤醒；被中断                                       |
| `TERMINATED`    | 线程执行完毕或异常结束                                 | `run()` 方法执行结束；线程抛出未捕获异常                     | 不能再转换为其他状态                                         |

| 当前状态        | 触发条件                                         | 转换后的状态    |
| --------------- | ------------------------------------------------ | --------------- |
| `NEW`           | 调用 `start()`                                   | `RUNNABLE`      |
| `RUNNABLE`      | 抢不到 `synchronized` 锁                         | `BLOCKED`       |
| `BLOCKED`       | 获取到 `synchronized` 锁                         | `RUNNABLE`      |
| `RUNNABLE`      | 调用 `Object.wait()`                             | `WAITING`       |
| `RUNNABLE`      | 调用无参 `Thread.join()`                         | `WAITING`       |
| `RUNNABLE`      | 调用 `LockSupport.park()`                        | `WAITING`       |
| `WAITING`       | 被 `notify()` / `notifyAll()` 唤醒，并重新拿到锁 | `RUNNABLE`      |
| `WAITING`       | `join()` 等待的线程执行结束                      | `RUNNABLE`      |
| `WAITING`       | 被 `LockSupport.unpark()` 唤醒                   | `RUNNABLE`      |
| `RUNNABLE`      | 调用 `Thread.sleep(time)`                        | `TIMED_WAITING` |
| `RUNNABLE`      | 调用 `Object.wait(time)`                         | `TIMED_WAITING` |
| `RUNNABLE`      | 调用 `Thread.join(time)`                         | `TIMED_WAITING` |
| `RUNNABLE`      | 调用 `LockSupport.parkNanos()` / `parkUntil()`   | `TIMED_WAITING` |
| `TIMED_WAITING` | 等待时间到                                       | `RUNNABLE`      |
| `TIMED_WAITING` | 被唤醒或中断                                     | `RUNNABLE`      |
| `RUNNABLE`      | `run()` 方法执行完毕                             | `TERMINATED`    |
| `RUNNABLE`      | 抛出未捕获异常                                   | `TERMINATED`    |

| 易混点                             | 说明                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| `RUNNABLE` 不一定真的在运行        | Java 中没有单独的 `RUNNING` 状态，正在运行和等待 CPU 时间片都属于 `RUNNABLE` |
| `BLOCKED` 只针对 `synchronized` 锁 | 等待 `ReentrantLock` 一般不是 `BLOCKED`，通常表现为 `WAITING` |
| `sleep()` 不会释放锁               | 当前线程睡眠，但如果它持有 `synchronized` 锁，不会释放       |
| `wait()` 会释放锁                  | 调用 `wait()` 后，线程会释放当前对象的监视器锁               |
| `notify()` 后不是立刻运行          | 被唤醒的线程还要重新竞争锁，拿到锁后才会进入 `RUNNABLE`      |
| `TERMINATED` 后不能重新启动        | 一个线程对象只能调用一次 `start()`，结束后不能再次启动       |

| 方法                      | 进入状态        | 是否释放锁             | 是否需要被唤醒                       | 说明                           |
| ------------------------- | --------------- | ---------------------- | ------------------------------------ | ------------------------------ |
| `Thread.sleep(time)`      | `TIMED_WAITING` | 不释放锁               | 不需要，时间到自动恢复               | 让当前线程睡眠指定时间         |
| `Object.wait()`           | `WAITING`       | 释放锁                 | 需要 `notify()` / `notifyAll()` 唤醒 | 必须在 `synchronized` 中使用   |
| `Object.wait(time)`       | `TIMED_WAITING` | 释放锁                 | 可以被唤醒，也可以时间到自动恢复     | 带超时时间的等待               |
| `Thread.join()`           | `WAITING`       | 不释放当前持有的普通锁 | 等待目标线程结束                     | 当前线程等待另一个线程执行完成 |
| `Thread.join(time)`       | `TIMED_WAITING` | 不释放当前持有的普通锁 | 目标线程结束或时间到                 | 带超时时间的线程等待           |
| `LockSupport.park()`      | `WAITING`       | 不释放锁               | 需要 `unpark()` 唤醒                 | 常用于 JUC 并发工具底层        |
| `LockSupport.parkNanos()` | `TIMED_WAITING` | 不释放锁               | 可以被 `unpark()` 唤醒，也可以时间到 | 纳秒级超时等待                 |

| 问题                                   | 答案                                                         |
| -------------------------------------- | ------------------------------------------------------------ |
| Java 线程有几种状态？                  | 6 种：`NEW`、`RUNNABLE`、`BLOCKED`、`WAITING`、`TIMED_WAITING`、`TERMINATED` |
| Java 中有 `RUNNING` 状态吗？           | 没有。正在运行和等待 CPU 调度都属于 `RUNNABLE`               |
| `BLOCKED` 和 `WAITING` 有什么区别？    | `BLOCKED` 是等待 `synchronized` 锁；`WAITING` 是主动进入无限期等待，需要其他线程唤醒 |
| `sleep()` 和 `wait()` 最大区别是什么？ | `sleep()` 不释放锁；`wait()` 会释放锁                        |
| `notify()` 后线程会马上运行吗？        | 不会。被唤醒后还要重新竞争锁，拿到锁后才可能运行             |
| `start()` 后线程一定立刻执行吗？       | 不一定。调用 `start()` 后线程进入 `RUNNABLE`，是否立刻执行取决于 CPU 调度 |
| 线程结束后还能再次调用 `start()` 吗？  | 不能。线程一旦进入 `TERMINATED`，再次调用 `start()` 会抛出 `IllegalThreadStateException` |

# synchronized

## 1.加在不同位置

```java
// 加在成员方法上，相当于锁住的是当前的 this 对象
public synchronized void increment() {
    count++;
}

// 加在静态方法上，相当于锁住的是当前类的对象 xxx.class
public static synchronized void increment() {
    count++;
}
```

## 2.常见线程安全类

String 	Integer	StringBuffer	Random	HashTable	juc包下的类

- 多个线程调用他们同一个实例的某个方法时，是线程安全的
- 它们的每个方法是原子的
- 但是多个方法组合起来不一定线程安全

## 3.Java对象头

**组要组成部分：**

Mark Word：存储 hashCode、GC 年龄、锁状态等信息
Klass Pointer：指向类元数据的指针（对象的类型信息）

**注：**数组对象多一个 length 数组长度（4个字节）

**普通对象**

| JVM 情况                  | Mark Word | Klass Pointer | 对象头大小      |
| ------------------------- | --------- | ------------- | --------------- |
| 32 位 JVM                 | 32 位     | 32 位         | 64 位，8 字节   |
| 64 位 JVM，未开启压缩指针 | 64 位     | 64 位         | 128 位，16 字节 |
| 64 位 JVM，开启压缩指针   | 64 位     | 32 位         | 96 位，12 字节  |

**数组对象**

| JVM 情况                  | Mark Word | Klass Pointer | 数组长度 | 数组对象头大小                      |
| ------------------------- | --------- | ------------- | -------- | ----------------------------------- |
| 32 位 JVM                 | 32 位     | 32 位         | 32 位    | 96 位，12 字节                      |
| 64 位 JVM，未开启压缩指针 | 64 位     | 64 位         | 32 位    | 160 位，20 字节，通常对齐到 24 字节 |
| 64 位 JVM，开启压缩指针   | 64 位     | 32 位         | 32 位    | 128 位，16 字节                     |

**32 位 JVM 中 Mark Word 的结构：**

| 对象状态                          | Mark Word 内容结构                                     |     biased_lock | 锁标志位 | 主要含义                                                     |
| --------------------------------- | ------------------------------------------------------ | --------------: | -------: | ------------------------------------------------------------ |
| Normal / 无锁状态                 | `hashcode:25 \| age:4 \| biased_lock:0 \| 01`          | 0（不是偏向锁） |       01 | 普通无锁对象，Mark Word 中可以存储对象的 hashCode、GC 年龄等信息 |
| Biased / 偏向锁状态               | `thread:23 \| epoch:2 \| age:4 \| biased_lock:1 \| 01` |   1（是偏向锁） |       01 | 对象偏向某个线程，Mark Word 中存储偏向线程 ID、epoch、GC 年龄等 |
| Lightweight Locked / 轻量级锁状态 | `ptr_to_lock_record:30 \| 00`                          |               - |       00 | Mark Word 中存储指向线程栈中 Lock Record 的指针              |
| Heavyweight Locked / 重量级锁状态 | `ptr_to_heavyweight_monitor:30 \| 10`                  |               - |       10 | Mark Word 中存储指向 ObjectMonitor 的指针                    |
| Marked for GC / GC 标记状态       | `空:30 \| 11`                                          |               - |       11 | 对象被 GC 标记，表示与垃圾回收相关的特殊状态                 |

## 4.Monitor 原理

![image-20260429153809688](./JUC.assets/image-20260429153809688.png)

```java
static final Object lock = new Object();
static int counter = 0;
public static void main(String[] args) throws InterruptedException {
    synchronized (lock) {
        counter++;
    }
}
```

```class
Compiled from "test1.java"
public class com.tsw.bsynchronized.test1 {
  static final java.lang.Object lock;

  static int counter;

  public com.tsw.bsynchronized.test1();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]) throws java.lang.InterruptedException;
    Code:
       0: getstatic     #2                  // Field lock:Ljava/lang/Object;  得到 lock 的引用
       3: dup								// 复制 lock 的引用
       4: astore_1							// 将 lock 的引用赋值给 slot1
       5: monitorenter						// 将 Mark Word 指向 Monitor
       
       6: getstatic     #3                  // Field counter:I   准备变量
       9: iconst_1							// 准备常量 1
      10: iadd								// 加1
      11: putstatic     #3                  // Field counter:I	赋值给变量
      
      14: aload_1							// 拿到 slot1 中的 lock 引用地址
      15: monitorexit						// 解锁，将 lock 对象中的 MarkWord 重置，唤醒 EntryList 
      16: goto          24					// 到24行 return
      
      19: astore_2							// 将异常变量存储到 slot2
      20: aload_1							// 拿到 slot1 中的 lock 引用地址
      21: monitorexit						// 解锁，将 lock 对象中的 MarkWord 重置，唤醒 EntryList
      22: aload_2							// 拿到异常对象
      23: athrow							// throw e
      
      24: return
    Exception table:
       from    to  target type
           6    16    19   any		// 6-19行如果出现异常，将会执行19行的代码
          19    22    19   any

  static {};
    Code:
       0: new           #4                  // class java/lang/Object
       3: dup
       4: invokespecial #1                  // Method java/lang/Object."<init>":()V
       7: putstatic     #2                  // Field lock:Ljava/lang/Object;
      10: iconst_0
      11: putstatic     #3                  // Field counter:I
      14: return
}
```

## 5.wait-notify原理

![image-20260429160924473](./JUC.assets/image-20260429160924473.png)

- Owner 线程发现条件不满足，调用 wait 方法，即可进入 WaitSet 变为 WAITING 状态
- BLOCKED 和 WAITING 的线程都处于阻塞状态，**不占用 CPU 时间片**
- BLOCKED 线程会在 Owner 线程释放锁时唤醒
- WAITING 线程会在 Owner 线程调用 notify 或 notifyAll 时唤醒，但唤醒后并不意味者立刻获得锁，仍需进入 EntryList 重新竞争

## 6.join 原理（保护性暂停）

```java
public final synchronized void join(long millis)
throws InterruptedException {
    long base = System.currentTimeMillis();
    // 经历时间
    long now = 0;
    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is
    }
    if (millis == 0) {
        while (isAlive()) {
            wait(0);
        }
    } else {
        while (isAlive()) {
            // 经历时间大于等待时间就退出
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```

## 7.Park & Unpark

**都是 LockSupport 中的静态方法**

```java
@Slf4j(topic = "c.ParkUnpark")
public class ParkUnpark {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            log.debug("start...");
            
            sleep(2000);
            
            log.debug("park...");
            LockSupport.park();
            log.debug("end...");
        }, "t1");
        t1.start();

        // 主线程先 unpark 也可以唤醒这个线程后来的 park
        sleep(1000);
        log.debug("unpark...");
        LockSupport.unpark(t1);
    }
}
21:31:40 [t1] c.ParkUnpark - start...
21:31:41 [main] c.ParkUnpark - unpark...
21:31:42 [t1] c.ParkUnpark - park...
21:31:42 [t1] c.ParkUnpark - end...
```

**特点：**

与 wait-notify 相比

- wait、notify 和 notifyAll 必须配合 Monitor 使用
- park 和 unpark 是以线程为单位来进行 阻塞 和 唤醒 线程，而 noitfy 只能随机唤醒一个线程，noitfyAll 唤醒所有线程，没有那么精确
- park 和 unpark 可以先 unpark ，但是 wait 和 noitfy 不可以先 noitfy 或者 noitfyAll

## 8.死锁

**产生条件：**

- 互斥条件
- 请求与保持条件
- 不可剥夺条件
- 循环等待条件

```java
public class DeadLockDemo {

    private static final Object lockA = new Object();
    private static final Object lockB = new Object();

    public static void main(String[] args) {

        new Thread(() -> {
            synchronized (lockA) {
                System.out.println("线程1拿到了 lockA");

                sleep(500);

                synchronized (lockB) {
                    System.out.println("线程1拿到了 lockB");
                }
            }
        }, "线程1").start();

        new Thread(() -> {
            synchronized (lockB) {
                System.out.println("线程2拿到了 lockB");

                sleep(500);

                synchronized (lockA) {
                    System.out.println("线程2拿到了 lockA");
                }
            }
        }, "线程2").start();
    }
}
```

**避免死锁：**

- 固定加锁顺序（都按先 A 后 B 的顺序）
- 减少锁的嵌套
- 使用 tryLock 设置超时时间
- 一次性申请所有资源
- 避免长时间持有锁

**哲学家就餐问题死锁版本：**

```java
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

@Slf4j(topic = "c.PhilosopherDemo")
public class PhilosopherDemo {

    public static void main(String[] args) {
        Chopstick c1 = new Chopstick("1号筷子");
        Chopstick c2 = new Chopstick("2号筷子");
        Chopstick c3 = new Chopstick("3号筷子");
        Chopstick c4 = new Chopstick("4号筷子");
        Chopstick c5 = new Chopstick("5号筷子");

        new Philosopher("苏格拉底", c1, c2).start();
        new Philosopher("柏拉图", c2, c3).start();
        new Philosopher("亚里士多德", c3, c4).start();
        new Philosopher("赫拉克利特", c4, c5).start();
        new Philosopher("阿基米德", c5, c1).start();
    }
}

@Slf4j(topic = "c.Philosopher")
class Philosopher extends Thread {

    private final Chopstick left;
    private final Chopstick right;

    public Philosopher(String name, Chopstick left, Chopstick right) {
        super(name);
        this.left = left;
        this.right = right;
    }

    @Override
    public void run() {
        while (true) {
            synchronized (left) {
                log.debug("拿到了左手边的 {}", left.getName());

                synchronized (right) {
                    log.debug("拿到了右手边的 {}", right.getName());
                    eat();
                }
            }
        }
    }

    private void eat() {
        log.debug("正在吃饭...");
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}

class Chopstick {

    private final String name;

    public Chopstick(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

## 9.活锁

**概述：**互相改变对方的终止条件，导致谁都结束不了

```java
// 共享变量
static AtomicInteger count = new AtomicInteger(0);

public static void main(String[] args) {
    Thread addThread = new Thread(() -> {
        while (true) {
            // 如果发现 count 是 0，就加 1
            if (count.get() == 0) {
                System.out.println("加线程：发现 count = 0，我加 1");
                count.incrementAndGet();
                sleep(500);
            }
        }
    }, "加线程");
    Thread subThread = new Thread(() -> {
        while (true) {
            // 如果发现 count 是 1，就减 1
            if (count.get() == 1) {
                System.out.println("减线程：发现 count = 1，我减 1");
                count.decrementAndGet();
                sleep(500);
            }
        }
    }, "减线程");
    addThread.start();
    subThread.start();
}
```

## 10.线程饥饿

```java
public static void main(String[] args) {
    Chopstick c1 = new Chopstick("1号筷子");
    Chopstick c2 = new Chopstick("2号筷子");
    Chopstick c3 = new Chopstick("3号筷子");
    Chopstick c4 = new Chopstick("4号筷子");
    Chopstick c5 = new Chopstick("5号筷子");
    new Philosopher("苏格拉底", c1, c2).start();
    new Philosopher("柏拉图", c2, c3).start();
    new Philosopher("亚里士多德", c3, c4).start();
    new Philosopher("赫拉克利特", c4, c5).start();
    // 把最后一个改为顺序执行 不会产生死锁了，但是 阿基米德 会长时间拿不到Cpu的执行权
    new Philosopher("阿基米德", c1, c5).start();
}
```

## 11.ReentrantLock

**特点：**

- 可中断
- 可以设置超时时间
- 可以设置为公平锁
- 支持多个条件变量

与 synchronized 一样都支持可重入



