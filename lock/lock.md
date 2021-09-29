# Lock（锁）面试

## 1. 请简述下 synchronized 与 java.util.concurrent.locks.Lock 的相同之处和不同之处

- **主要相同点：** Lock 能完成 synchronized 所实现的所有功能
- **主要不同点：** Lock 有比 synchronized 更精确的线程语义和更好的性能。synchronized 会自动释放锁，而 Lock 一定要手动释放，并且必须在 finally 从句中释放。

## 2. Java 中如何确保 N 个线程可以访问 N 个资源，但同时又不导致死锁？

**参考回答：**  
使用多线程的时候，一种非常简单的避免死锁的方式就是：指定获取锁定的顺序，并强制线程按照指定的顺序获取锁。因此，如果所有的线程都是以同样的顺序加锁和释放锁，就不会出现死锁了。  
  
- 预防死锁，预先破坏产生死锁的四个条件。互斥不可能破坏，所以有如下三种方法:
    1. 破坏请求和保持条件，进程必须等所有要请求的资源都空闲时才能申请资源，这种方法会使资源浪费严重（有些资源可能仅在运行初期或结束时才使用，甚至根本不使用）。允许进程获取初期所需资源后，便开始运行，运行过程中再逐步释放自己占有的资源，比如有一个进程的任务是把数据复制到磁盘中再打印，前期只需获得磁盘资源而不需要获得打印机资源，待复制完毕后再释放掉磁盘资源。这种方法比第一种方法好，会使资源利用率上升。  
      
    2. 破坏不可抢占条件，这种方法代价大，实现复杂。  
      
    3. 破坏循环等待条件，对各进程请求资源的顺序做一个规定，避免相互等待。这种方法对资源的利用比率前两种都高，但是前期要为设备指定序号，新设备加入会有一个问题

## 3. 请问什么是死锁（deadlock）？

**参考回答：**

- 两个线程或两个以上线程都在等待对方执行完毕才能继续往下执行的时候就发生了死锁。结果就是这些线程都陷入了无限的等待中。

> 例如，如果线程 1 锁住了 A，然后尝试对 B 进行加锁，同时线程 2 已经锁住了 B，接着尝试对 A 进行加锁，这时死锁就发生了。线程 1 永远得不到 B，线程 2 也永远得不到 A，并且它们永远也不会知道了这样的事情。为了得到彼此的对象（A 和 B），它们将永远阻塞下去。这种情况就是一个死锁。

## 4. synchronized作用于静态方法和非静态方法的区别
- 非静态方法：  
给对象加锁(可以理解为给这个对象的内存上锁,注意 只是这块内存,其他同类对象都会有各自的内存锁),这时候在其他一个以上线程中执行该对象的这个同步方法(注意:是该对象)就会产生互斥
- 静态方法:   
相当于在类上加锁(*.class
位于代码区,静态方法位于静态区域,这个类产生的对象公用这个静态方法,所以这块内存，N个对象来竞争),
这时候,只要是这个类产生的对象,在调用这个静态方法时都会产生互斥。即该类所有的对象都共享一把锁。

## 5. 当一个线程进入一个对象的synchronized方法A之后，其它线程是否可进入此对象的synchronized方法B？

不能。其它线程只能访问该对象的非同步方法，同步方法则不能进入。因为非静态方法上的synchronized修饰符要求执行方法时要获得对象的锁，如果已经进入A方法说明对象锁已经被取走，那么试图进入B方法的线程就只能在等锁池（注意不是等待池哦）中等待对象的锁

## 6. 线程同步的几种方式

1. synchronized修饰
2. volatile实现同步（只能保证可见性，不能保证原子性）
3. 使用局部变量ThreadLocal
4. 使用原子类（AtomicInteger、AtomicBoolean……）
5. 使用Lock
6. 使用容器类（BlockingQueue、ConcurrentHashMap）

## 7. 乐观锁和悲观锁的理解及如何实现，有哪些实现方式？

- 乐观锁，每次操作时不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止
- 悲观锁是会导致其它所有需要锁的线程挂起，等待持有锁的线程释放锁。
- 乐观锁可以使用volatile+CAS原语实现，带参数版本来避免ABA问题，在读取和替换的时候进行判定版本是否一致
- 悲观锁可以使用synchronize的以及Lock

## 