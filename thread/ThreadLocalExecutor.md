# 1. ThreadPoolExcutor简述

### 1. 线程池的工作原理：

![threadpoolExecutor][picture]

1.1 线程池刚创建时，里面没有一个线程，任务队列是作为参数传进来的。不过，就算队列里面有任务，线程池也不会马上执行它们。

1.2 当调用 execute() 方法添加一个任务时，线程池会做出如下判断：

   -   如果正在运行的线程数小于 corePoolSize，那么马上创建线程运行这个任务。
   -   如果正在运行的线程大于或等于 corePoolSize，那么将这个任务放入队列。
   -   如果这个时候队列满了，而且正在运行的线程数量小于 maxmumPoolSize，那么还是要创建线程运行这个任务。
   -   如果对列满了，而且正在运行的线程数量大于或等于 maximumPoolSize，那么线程池会抛出异常，告诉调用者“我不能再接受任务了”

1.3 当一个线程完成任务时，它会从队列中取下一个任务来执行。

1.4 当一个线程无事可做，超过一定的时间（keepAliveTime）时，线程池会判断，如果当前运行的线程数大于 corePoolSize 时，那么这个线程会被停用掉，所以线程池的所有任务完成后，它最终会收缩到 corePoolSize 的大小。

### 2.线程池有哪些配置

> 线程池可以使用 java.util.concurrent.ThreadPoolExecutor 来创建，在该类中包含最全参数的构造函数

```java
public ThreadPoolExecutor(int corePoolSize,
                         int maximumPoolSize,
                         int keepAliveTime,
                         TimeUnit unit,
                         BlockingQueue<Runnable> workQueue,
                         ThreadFactory threadFactory,
                         RejectedExecutionHandler handler)
```

**maximumPoolSize:** 线程池最大线程数，表示在线程池中最多能创建多少个线程。如果当线程池中的数量到达这个数字时，新来的任务会抛出异常．

**keepAliveTime:**表示线程没有任务执行时最多能保持多少时间会停止，然后线程池的数目维持在 corePoolSize。

**unit:**参数 keepAliveTime 的时间单位。

**workQueue:** 一个阻塞队列，用来存储等待执行的任务，如果当前对线程的需求超过了 corePoolSize 大小，才会放在这里。

**threadFactory:** 线程工厂，主要用来创建线程，比如指定线程的名字。

**handler:** 如果线程池已满，新的任务处理方式。

### 3. 线程池的阻塞队列包含哪几种选择？

> 1. ArrayBlockingQueue
> 2. LinkedBlockingQueue
> 3. PriorityBlockingQueue
> 4. SynchronousQueue

***ArrayBlockingQueue*** 是一个有边界的阻塞队列，它的内部实现是一个数组。它的容量在初始化时就确定不变。

***LinkedBlockingQueue*** 阻塞队列大小的配置是可选的，其内部实现是一个链表

***PriorityBlockingQueue*** 是一个没有边界的队列，所有插入到 PriorityBlockingQueue 的对象必须实现 java.lang.Comparable 接口，队列优先级的排序就是按照我们对这个接口的实现来定义的。

***SynchronousQueue*** 队列内部仅允许一个元素。当一个线程插入一个元素后会被阻塞，除非这个元素被另一个线程消费。

### 4. 如果线程池已经满了，可是还有新的任务提交怎么办？

线程池已经满的定义，是只运行线程数 == maximumPoolSize，并且 workQueue 是有界队列并且已满。

这时候再提交任务怎么办呢？线程池会将任务传递给给最后一个参数 RejectedExecutionHandler 来处理，比如打印报错日志、抛出异常、存储到 Mysql/redis 用于后续处理等等，线程池默认也提供了几种处理方式。

[picture]: ../img/threadPoolExcutor(1).png

