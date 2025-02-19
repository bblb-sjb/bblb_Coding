# Java并发面试篇

# 多线程

## java里面的线程和操作系统的线程一样吗？

在类 UNIX 系统中，Java 通过 `pthread_create` 来创建内核级线程，并且返回线程创建是否成功的状态码。大多数情况下，JVM 中的 Java 线程和操作系统的内核级线程是一一对应的。

##  使用多线程要注意哪些问题？

- 原子性：互斥访问（使用锁），使用支持原子操作的类（atomic包）
- 可见性：一个线程对内存的修改可以被其他线程感知，`volatile` 通过禁止线程的本地缓存和强制从主内存读取数据来确保变量的最新值对所有线程可见。
- 有序性：使用volatile关键字保证指令不会被重排。
- 避免死锁及线程饥饿。

## 保证数据一致性的方案有哪些？

- 事务：使用事务（事务属性：ACID原子性、一致性、隔离性、持久性）来控制一系列操作，要么都成功要么都失败回滚。
- 锁：悲观锁
- MVCC版本控制：乐观锁

## 创建线程的方法

- 如果没有继承类的话可以直接继承Thread类，重写`run()`方法

  ```java
  class MyThread extends Thread {
      @Override
      public void run() {
          ... // 执行线程的任务
      }
  }
  public class Main {
      public static void main(String[] args) {
          MyThread thread = new MyThread();
          thread.start();  // 启动线程
      }
  }
  ```

  调用 `start()` 方法会启动线程，而不是直接调用 `run()` 方法。`start()` 方法会内部调用 `run()` 方法。

- 如果该类继承了某个类方法，那么可以实现`Runnable`接口并重写其`run()`方法

  ```java
  class MyRunnable implements Runnable {
      @Override
      public void run() {
          ... // 执行线程的任务
      }
  }
  public class Main {
      public static void main(String[] args) {
          MyRunnable myRunnable = new MyRunnable();
          Thread thread = new Thread(myRunnable);
          thread.start();  // 启动线程
      }
  }
  ```

- 也可以直接实现`Callable` 。`Callable` 是一个功能更强大的接口，类似于 `Runnable`，但是它可以返回结果。通过 `ExecutorService` 提交 `Callable` 任务，并通过 `Future` 获取任务的执行结果。

  ```java
  class MyCallable implements Callable<Integer> {
      @Override
      public Integer call() throws Exception {
      	...
          return 42; // 返回结果
      }
  }
  public class Main {
      public static void main(String[] args) throws InterruptedException, ExecutionException {
          MyCallable myCallable = new MyCallable(); // 创建 MyCallable 实例
          // 使用 FutureTask 来包装 Callable
          FutureTask<Integer> futureTask = new FutureTask<>(myCallable);
          Thread thread = new Thread(futureTask); // 使用 Thread 执行 FutureTask
          thread.start(); // 启动线程
          // 等待线程执行结果
          Integer result = futureTask.get(); // get() 会阻塞直到任务完成
          System.out.println("Task result: " + result);      
          // 等待线程结束
          thread.join(); // 等待线程执行完毕
      }
  }
  ```

- 线程池

  使用线程池来管理线程，`ExecutorService` 是 JDK 5 引入的线程池框架，提供了更高效、更方便的线程管理。可以使用 `ExecutorService` 来创建并执行线程池中的线程。

  ```java
  public class Main {
      public static void main(String[] args) {
          ExecutorService executorService = Executors.newFixedThreadPool(2);  // 创建定长线程池
          executorService.submit(() -> {
              ... // 执行任务   
          });
          executorService.shutdown();  // 关闭线程池
      }
  }
  ```

# 并发安全

## juc包下常用的类

- 线程池相关：
  - `ThreadPoolExecutor`：最核心的线程池类，线程池的真正实现类是 ThreadPoolExecutor
  - `Executors`：线程工厂类，Executors已经为我们封装好了 4 种常见的功能线程池，如下：
    - 定长线程池（FixedThreadPool）：只有核心线程，线程数量固定，执行完立即回收，任务队列为链表结构的有界队列。用于控制线程最大并发数。
    - 定时线程池（ScheduledThreadPool ）：核心线程数量固定，非核心线程数量无限，执行完闲置 10ms 后回收，任务队列为延时阻塞队列。用于执行定时或周期性的任务。
    - 可缓存线程池（CachedThreadPool）：无核心线程，非核心线程数量无限，执行完闲置 60s 后回收，任务队列为不存储元素的阻塞队列。用于执行大量、耗时少的任务。
    - 单线程化线程池（SingleThreadExecutor）：只有 1 个核心线程，无非核心线程，执行完立即回收，任务队列为链表结构的有界队列。不适合并发但可能引起 IO 阻塞性及影响 UI 线程响应的操作，如数据库操作、文件操作等。
- 线程安全的并发集合类：
  - CopyOnWriteArrayList
  - ConcurrentHashMap
- 同步工具类：
  - Semaphore：信号量
- 原子类

## 怎么保证多线程安全

除了使用juc包下的类，还可以通过一些关键字：

- `synchronized`关键字：可以使用`synchronized`关键字来同步代码块或方法，确保同一时刻只有一个线程可以访问这些代码。
- `volatile`关键字：`volatile` 通过禁止线程的本地缓存和强制从主内存读取数据来确保变量的最新值对所有线程可见。

## Java中常用的锁

- 悲观锁：

  - `synchronized` 锁：分为方法级锁和代码块级锁

  - `ReentrantLock`： 是 `java.util.concurrent.locks` 包中的一个显式锁实现，它实现了与 `synchronized` 相同的功能，但提供了更多的灵活性和控制，可以更更加细粒度的控制锁。

    ```java
    Lock lock = new ReentrantLock();
    lock.lock();  // 获取锁
    try {
        // 线程安全代码
    } finally {
        lock.unlock();  // 释放锁
    }
    ```

	- `ReadWriteLock` ：是一种读写分离的锁，适用于读多写少的场景。它允许多个线程并行读取共享资源，但写操作必须独占访问权限。读锁多个线程可以同时持有，并且并发读取。写锁是线程独占的。

    ```java
    ReadWriteLock rwLock = new ReentrantReadWriteLock();
    Lock readLock = rwLock.readLock();
    Lock writeLock = rwLock.writeLock();
    readLock.lock();// 获取读锁
    try {
        // 线程安全的读取操作
    } finally {
        readLock.unlock();
    }
    writeLock.lock();// 获取写锁
    try {
        // 线程安全的写操作
    } finally {
        writeLock.unlock();
    }
    ```

	- `Semaphore`：信号量也能被认为是一种特殊的锁，是一种控制同时访问共享资源的数量的同步机制。它通过一个计数器来控制访问权限，适用于控制并发访问的场景。

    ```java
    Semaphore semaphore = new Semaphore(3);  // 允许最多3个线程同时访问资源
    semaphore.acquire();  // 获取信号量，减少可用许可
    try {
        // 线程安全代码
    } finally {
        semaphore.release();  // 释放信号量，增加可用许可
    }
    ```


- 乐观锁：

  - 版本号或者时间戳：通过在每个数据对象中维护一个版本号，读线程读取数据时获取版本号，写线程修改时检查版本号是否一致。如果一致则更新数据，并更新版本号，否则重试。

  - CAS：使用 `AtomicInteger`、`AtomicReference` 等类中的原子操作来实现乐观锁。

    ```java
    AtomicInteger count = new AtomicInteger(0);
    count.incrementAndGet();  // 原子操作，避免竞争条件
    ```

## synchronized和reentrantlock区别？

| 特性           | **synchronized**                  | **ReentrantLock**                       |
| -------------- | --------------------------------- | --------------------------------------- |
| **锁类型**     | 内置锁                            | 显式锁                                  |
| **加锁和解锁** | 自动加锁，自动解锁                | 通过 `lock()` 加锁，`unlock()` 解锁     |
| **可中断性**   | 不可中断                          | 可中断，通过 `lockInterruptibly()` 方法 |
| **公平性**     | 非公平锁                          | 可设置为公平锁                          |
| **可重入性**   | 可重入                            | 可重入                                  |
| **条件变量**   | 内建条件变量 `wait()`、`notify()` | 通过 `Condition` 提供灵活的条件变量     |
| **性能**       | 较低，尤其在竞争激烈时            | 更高，特别在高并发和需要灵活控制的场景  |
| **死锁避免**   | 需要开发者小心避免死锁            | 提供 `tryLock()` 方法来避免死锁         |

## 如何理解可重入锁？

可重入锁允许同一线程在已经持有锁的情况下再次获取该锁，背后通过一个计数器来记录线程获取锁的次数。每当线程请求锁时，计数器增加；每次释放锁时，计数器减少，直到计数器为 0 时，锁才会被完全释放。这种机制避免了线程在递归或多层方法调用中因重复加锁导致的死锁问题。

## 死锁产生的必要条件

- 互斥条件：只有对必须互斥使用的资源的争抢才会导致死锁
- 不剥夺条件：进程所获得的资源在未使用完之前，不能由其他进程强行夺走，只能主动释放。
- 请求和保持条件：进程已经保持了至少一个资源，但又提出了新的资源请求，而该资源又被其他进程占有，此时请求进程被阻塞，但又对自己已有的资源保持不放。
- 循环等待条件：存在一种进程资源的循环等待链，链中的每一个进程已获得的资源同时被下一个进程所请求。

死锁的处理策略：破坏其中任意一个即可

# 线程池

## 线程池参数

```java
// 创建线程池
ThreadPoolExecutor threadPool = new ThreadPoolExecutor(CORE_POOL_SIZE,
                                             MAXIMUM_POOL_SIZE,
                                             KEEP_ALIVE,
                                             TimeUnit.SECONDS,
                                             sPoolWorkQueue,
                                             sThreadFactory);
// 向线程池提交任务
threadPool.execute(new Runnable() {
    @Override
    public void run() {
        ... // 线程执行的任务
    }
});
// 关闭线程池
threadPool.shutdown(); // 设置线程池的状态为SHUTDOWN，然后中断所有没有正在执行任务的线程
threadPool.shutdownNow(); // 设置线程池的状态为 STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表
```

- `corePoolSize` 核心线程数（必需)

  默认情况下，核心线程会一直存活，但是当将 `allowCoreThreadTimeout`设置为 true 时，核心线程也会超时回收。

- `maximumPoolSize` 最大线程数（必需)

  线程池所能容纳的最大线程数。当活跃线程数达到该数值后，后续的新任务将会阻塞

- `keepAliveTime` 线程闲置超时时长（必需）

  如果超过该时长，非核心线程就会被回收。如果将`allowCoreThreadTimeout`设置为 true 时，核心线程也会超时回收。

- `unit` 超时时间的单位（必需）

  指定`keepAliveTime`参数的时间单位。常用的有：`TimeUnit.MILLISECONDS`（毫秒）、`TimeUnit.SECONDS`（秒）、`TimeUnit.MINUTES`（分）。

- workQueue（必需）

  任务队列。通过线程池的 execute() 方法提交的 Runnable 对象将存储在该参数中。其采用阻塞队列实现。

- threadFactory（可选）

  线程工厂。用于指定为线程池创建新线程的方式。线程工厂指定创建线程的方式，需要实现 ThreadFactory 接口，并实现 newThread(Runnable r) 方法。该参数可以不用指定，Executors 框架已经为我们实现了一个默认的线程工厂。

- handler（可选）

  拒绝策略。当达到最大线程数时需要执行的饱和策略。

### 任务队列workQueue

任务队列是基于阻塞队列实现的，即采用生产者消费者模式，在 Java 中需要实现BlockingQueue 接口。但 Java 已经为我们提供了 7 种阻塞队列的实现

- ArrayBlockingQueue：一个由数组结构组成的有界阻塞队列（数组结构可配合指针实现一个环形队列）。
- LinkedBlockingQueue： 一个由链表结构组成的有界阻塞队列，在未指明容量时，容量默认为 Integer.MAX_VALUE。
- PriorityBlockingQueue： 一个支持优先级排序的无界阻塞队列，对元素没有要求，可以实现 Comparable 接口也可以提供 Comparator 来对队列中的元素进行比较。跟时间没有任何关系，仅仅是按照优先级取任务。
- DelayQueue： 类似于PriorityBlockingQueue，是二叉堆实现的无界优先级阻塞队列。要求元素都实现 Delayed 接口，通过执行时延从队列中提取任务，时间没到任务取不出来。
- SynchronousQueue： 一个不存储元素的阻塞队列，消费者线程调用 take() 方法的时候就会发生阻塞，直到有一个生产者线程生产了一个元素，消费者线程就可以拿到这个元素并返回；生产者线程调用 put() 方法的时候也会发生阻塞，直到有一个消费者线程消费了一个元素，生产者才会返回。
- LinkedBlockingDeque： 使用双向队列实现的有界双端阻塞队列。双端意味着可以像普通队列一样 FIFO（先进先出），也可以像栈一样 FILO（先进后出）。
- LinkedTransferQueue： 它是ConcurrentLinkedQueue、LinkedBlockingQueue 和 SynchronousQueue 的结合体，但是把它用在 ThreadPoolExecutor 中，和LinkedBlockingQueue 行为一致，但是是无界的阻塞队列。

注意有界队列和无界队列的区别：如果使用有界队列，当队列饱和时并超过最大线程数时就会执行拒绝策略；而如果使用无界队列，因为任务队列永远都可以添加任务，所以设置 maximumPoolSize 没有任何意义。

### 拒绝策略handler

当线程池的线程数达到最大线程数时，需要执行拒绝策略。拒绝策略需要实现 RejectedExecutionHandler 接口，并实现 rejectedExecution(Runnable r, ThreadPoolExecutor executor) 方法。不过 Executors 框架已经为我们实现了 4 种拒绝策略：

- AbortPolicy（默认）：丢弃任务并抛出 RejectedExecutionException 异常。
- CallerRunsPolicy：由调用线程处理该任务。
- DiscardPolicy：丢弃任务，但是不抛出异常。可以配合这种模式进行自定义的处理方式。
- DiscardOldestPolicy：丢弃队列最早的未处理任务，然后重新尝试执行任务。

## 线程池参数经验

- CPU密集型：corePoolSize = CPU核数 + 1（避免过多线程竞争CPU）
- IO密集型：corePoolSize = CPU核数 x 2（或更高，具体看IO等待时间）

场景一：电商场景，特点瞬时高并发、任务处理时间短，线程池的配置可设置如下：

```java
ThreadPoolExecutor threadPool = new ThreadPoolExecutor(
    										 16, // 8核*2
                                             32, // 最大*2
                                             10, // 非核心线程空闲10s回收
                                             TimeUnit.SECONDS,
                                             new SynchronousQueue<>(), //不缓存任务，直接消费
                                             new AbortPolicy() // 直接拒绝
                                 );
```

场景二：后台数据处理服务，特点稳定流量、任务处理时间长（秒级）、允许一定延迟，线程池的配置可设置如下：

```java
ThreadPoolExecutor threadPool = new ThreadPoolExecutor(
    										 8, // 
                                             8, // 禁止扩容，避免消耗资源
                                             0, // 为了稳定不回收
                                             TimeUnit.SECONDS,
                                             new ArrayBlockingQueue<>(1000), // 有界队列，容量1000
                                             new CallerRunsPolicy() // 满了叫人来
                                 );
```

场景三：微服务HTTP请求处理，特点IO密集型、依赖下游服务响应时间，线程池的配置可设置如下：

```
ThreadPoolExecutor threadPool = new ThreadPoolExecutor(
    										 16, // 8核*2
                                             64, // 下游满的话多设置一点
                                             60, // 非核心线程空闲60s回收
                                             TimeUnit.SECONDS,
                                             new ArrayBlockingQueue<>(200), // 有界队列，容量200
                                             new CustomRetryPolicy() // 自定义拒绝策略（重试或降级）
                                 );
```

## 功能线程池

- 定长线程池（FixedThreadPool）：只有核心线程，线程数量固定，执行完立即回收，任务队列为链表结构的有界队列。用于控制线程最大并发数。
- 定时线程池（ScheduledThreadPool ）：核心线程数量固定，非核心线程数量无限，执行完闲置 10ms 后回收，任务队列为延时阻塞队列。用于执行定时或周期性的任务。
- 可缓存线程池（CachedThreadPool）：无核心线程，非核心线程数量无限，执行完闲置 60s 后回收，任务队列为不存储元素的阻塞队列。用于执行大量、耗时少的任务。
- 单线程化线程池（SingleThreadExecutor）：只有 1 个核心线程，无非核心线程，执行完立即回收，任务队列为链表结构的有界队列。不适合并发但可能引起 IO 阻塞性及影响 UI 线程响应的操作，如数据库操作、文件操作等。

但是根据《阿里巴巴 Java 开发手册》中禁止使用Executors类来进行创建线程池，会引发大量的生产事故，所以线程池的使用还是要结合具体的使用场景。

## 线程池中shutdown ()，shutdownNow()这两个方法有什么作用？

- `shutdown ()`：设置线程池的状态为SHUTDOWN，然后中断所有没有正在执行任务的线程
- `shutdownNow()`：设置线程池的状态为 STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表





