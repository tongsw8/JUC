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













