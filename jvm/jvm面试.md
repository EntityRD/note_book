# JVM 面试总结

## 1. JVM 运行内存的分类

- 程序计数器：当前线程所执行的字节码的行号指示器，用于记录正在执行的虚拟机字节指令地址，线程私有
   注：如果正在执行的是 Native 方法，计数器值则为空
- Java 虚拟栈：存放基本数据类型、对象的引用、方法出口等，线程私有
- Native 方法栈：和虚拟栈相似，只不过它服务于Native方法，线程私有
- Java 堆：java 内存最大的一块，所有对象实例、数组都存放在 java 堆，GC 回收的地方，线程共享
- 方法区：存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码数据等。（即永久带），回收目标主要是常量池的回收和类型的卸载，各线程共享

## 2. Java 内存堆和栈区别

- 栈内存用来存储基本类型的变量和对象的引用变量，堆内存用来存储 Java 中的对象，无论是**成员变量**，局部变量，还是类变量，它们指向的对象都存储在堆内存中
- 栈内存归属于单个线程，每个线程都会有一个栈内存，其存储的变量只能在其所属线程中可见，即栈内存可以理解成线程的私有内存，堆内存中的对象对所有线程可见。堆内存中的对象可以被所有线程访问
- 如果栈内存没有可用的空间存储方法调用和局部变量，JVM 会抛出 java.lang.StackOverFlowError，如果是堆内存没有可用的空间存储生成的对象，JVM 会抛出 java.lang.OutOfMemoryError
- 栈的内存要远远小于堆内存，如果你使用递归的话，那么你的栈很快就会充满，-Xss 选项设置栈内存的大小。-Xms 选项可以设置堆的开始时的大小

## 3. Java四引用

- 强引用（StrongReference）强引用是使用最普遍的引用。如果一个对象具有强引用，那垃圾回收器绝不会回收它。当内存空间不足，Java虚拟机宁愿抛出 OutOfMemoryError 错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题
- 软引用（SoftReference）
   如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中
- 弱引用（WeakReference）
   弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。
   弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中
- 虚引用（PhantomReference）
   虚引用在任何时候都可能被垃圾回收器回收，主要用来**跟踪对象被垃圾回收器回收的活动，被回收时会收到一个系统通知**。虚引用与软引用和弱引用的一个区别在于：虚引用**必须**和引用队列 （ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。

## 4. GC回收机制

- Java 中对象是采用 new 或者反射的方法创建的，这些对象的创建都是在堆（Heap）中分配的，所有对象的回收都是由Java虚拟机通过垃圾回收机制完成的。GC 为了能够正确释放对象，会监控每个对象的运行状况，对他们的申请、引用、被引用、赋值等状况进行监控
- Java 程序员不用担心内存管理，因为垃圾收集器会自动进行管理
- 可以调用下面的方法之一：System.gc() 或 Runtime.getRuntime().gc() ，但 JVM 可以屏蔽掉显示的垃圾回收调用

## 5. GC 标记对象的死活

- 引用计数法：给对象添加一个引用计数器,没当被引用的时候,计数器的值就加一。引用失效的时候减一,当计数器的值为 0 的时候就表示改对象可以被 GC 回收了，弊端:A->B,B->A,那么 AB 将永远不会被回收了。也就是引用有环的情况
- 根搜索算法(可达性算法) GC Roots Tracing：通过一个叫 GC Roots 的对象作为起点,从这些结点开始向下搜索,搜索所走过的路径称为引用链,当一个对象没有与任何的引用链相连的时候则改对象就可以被。 GC 回收回收了Roots 包括：java 虚拟机栈中引用的对象,本地方法栈中引用的对象,方法区中常量引用的对象,方法区中静态属性引用的对象

    - 在 java 语言中，可作为 GC Roots 的对象包括以下几种：

       > 虚拟机栈（栈帧中的本地变量表）中引用的对象  
       > 方法区中的类静态属性引用的对象  
       > 方法区中的常量引用对象  
       > 本地方法栈中 JNI （即一般说的 Native 方法）的引用对象

## 6. GC回收算法

- 标记-清除法：标记出没有用的对象，然后一个一个回收掉

    - 缺点：标记和清除两个过程效率不高，产生内存碎片导致需要分配较大对象时无法找到足够的连续内存而需要触发一次GC操作

- 复制算法: 按照容量划分二个大小相等的内存区域，当一块用完的时候将活着的对象复制到另一块上，然后再把已使用的内存空间一次清理掉

    - 缺点：将内存缩小为了原来的一半

- 标记-整理法：标记出没有用的对象，让所有存活的对象都向一端移动，然后直接清除掉端边界以外的内

    - 优点：解决了标记- 清除算法导致的内存碎片问题和在存活率较高时复制算法效率低的问题。

- 分代回收：根据对象存活周期的不同将内存划分为几块，一般是新生代和老年代，新生代基本采用复制算法，老年代采用标记整理算法

## 7. MinorGC&FullGC

- Minor GC 通常发生在新生代的 Eden 区，在这个区的对象生存期短，往往发生 GC 的频率较高，回收速度比较快，一般采用复制-回收算法
- Full GC/Major GC 发生在老年代，一般情况下，触发老年代GC的时候不会触发 Minor GC，所采用的是标记-清除算法

## 8. 内存分配与回收策略

- 结构（堆大小 = 新生代 + 老年代 ）：

    - 新生代(1/3)(初始对象，生命周期短)：Eden 区、survivior 0、survivior 1（ 8 : 1 : 1）
    - 老年代(2/3)(长时间存在的对象)

- 一般小型的对象都会在 Eden 区上分配，如果Eden区无法分配，那么尝试把活着的对象放到 survivor0 中去（Minor GC）

    - 如果 survivor0 可以放入，那么放入之后清除Eden区
    - 如果 survivor0 不可以放入，那么尝试把 Eden 和 survivor0 的存活对象放到 survivor1 中

        - 如果survivor1可以放入，那么放入 survivor1 之后清除 Eden 和 survivor0，之后再把 survivor1 中的对象复制到 survivor0 中，保持 survivor1 一直为空。
        - 如果 survivor1 不可以放入，那么直接把它们放入到老年代中，并清除 Eden 和 survivor0，这个过程也称为分配担保（Full GC）

- 大对象、长期存活的对象则直接进入老年代
- 动态对象年龄判定
- 空间分配担保，Full GC...

## 9. GC垃圾收集器

- Serial New 收集器是针对新生代的收集器，采用的是复制算法
- Parallel New（并行）收集器，新生代采用复制算法，老年代采用标记整理
- Parallel Scavenge（并行）收集器，针对新生代，采用复制收集算法
- Serial Old（串行）收集器，新生代采用复制，老年代采用标记清理
- Parallel Old（并行）收集器，针对老年代，标记整理
- CMS 收集器，基于标记清理
- G1 收集器(JDK)：整体上是基于标记清理，局部采用复制
- 综上：新生代基本采用复制算法，老年代采用标记整理算法。cms采用标记清理

## 10. Java类加载机制

- 概念：
    - 虚拟机把描述类的数据文件（字节码）加载到内存，并对数据进行验证、准备、解析以及类初始化，最终形成可以被虚拟机直接使用的 java 类型（java.lang.Class对象）

- 类的生命周期：
    - 加载过程：通过一个类的全限定名来获取定义此类的二进制字节流，将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。在内存中(方法区)生成一个代表这个类的 java.lang.Class 对象，作为方法区这个类的各种数据的访问入口；
    - 验证过程：为了确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，文件格式验证、元数据验证、字节码验证、符号引用验证
    - 准备过程：正式为类属性分配内存并设置类属性初始值的阶段，这些内存都将在方法区中进行分配
    - 解析阶段：虚拟机将常量池内的符号引用替换为直接引用的过程
    - 初始化阶段：类初始化阶段是类加载过程的最后一步。初始化阶段就是执行类构造器<clint>()方法的过程
    - 使用阶段：
    - 卸载阶段：

- Java 类加载器：

    - 类加载器负责加载所有的类，同一个类(一个类用其全限定类名(包名加类名)标志)只会被加载一次
    - Bootstrap ClassLoader:根类加载器，负责加载 java 的核心类，它不是 java.lang.ClassLoader 的子类，而是由 JVM 自身实现
    - Extension ClassLoader:扩展类加载器，扩展类加载器的加载路径是JDK目录下 jre/lib/ext ,扩展类的 getParent() 方法返回 null，实际上扩展类加载器的父类加载器是根加载器，只是根加载器并不是 Java 实现的
    - System ClassLoader:系统(应用)类加载器，它负责在JVM启动时加载来自java命令的 -classpath 选项、java.class.path 系统属性或 CLASSPATH 环境变量所指定的 jar 包和类路径。程序可以通过getSystemClassLoader() 来获取系统类加载器。系统加载器的加载路径是程序运行的当前路径

- 双亲委派模型的工作过程：

    - 首先会先查找当前 ClassLoader 是否加载过此类，有就返回；
    - 如果没有，查询父 ClassLoader 是否已经加载过此类，如果已经加载过,就直接返回 Parent 加载的类；
    - 如果整个类加载器体系上的 ClassLoader 都没有加载过，才由当前 ClassLoader 加载(调用 findClass )，整个过程类似循环链表一样。

- 双亲委托机制的作用：

    - 共享功能：可以避免重复加载，当父亲已经加载了该类的时候，子类不需要再次加载，一些 Framework 层级的类一旦被顶层的 ClassLoader 加载过就缓存在内存里面，以后任何地方用到都不需要重新加载。
    - 隔离功能：因为 String 已经在启动时被加载，所以用户自定义类是无法加载一个自定义的类装载器，保证 java/Android 核心类库的纯净和安全，防止恶意加载。

- 如何打破双亲委派模型？

    - 双亲委派模型的逻辑都在 loadClass() 中，重写 loaderClass() ，一般是重写 findClass() 的
    - 系统自带的三个类加载器都加载特定目录下的类，如果我们自己的类加载器放在一个特殊的目录，那么系统的加载器就无法加载，也就是最终还是由我们自己的加载器加载

- 自定义ClassLoader：

    - loadClass(String name,boolean resolve)：根据指定的二进制名称加载类
    - findClass(String name)： 根据二进制名称来查找类
    - 直接使用或继承已有的ClassLoader实现：java.net.URLClassLoader、java.security.SecureClassLoader、 java.rmi.server.RMIClassLoader
    - 在调用 loadClass()，会先根据委派模型在父加载器中加载，如果加载失败，则会调用自己的 findClass 方法来完成加载

## 11. 引起类加载操作的五个行为

- 遇到new、getstatic、putstatic或invokestatic这四条字节码指令
- 反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化
- 子类初始化的时候，如果其父类还没初始化，则需先触发其父类的初始化
- 虚拟机执行主类的时候(有 main(string[] args))
- JDK1.7 动态语言支持

## 12. Java对象创建时机

- 使用new关键字创建对象
- 使用Class类的newInstance方法(反射机制)
- 使用Constructor类的newInstance方法(反射机制)
- 使用Clone方法创建对象
- 使用(反)序列化机制创建对象

# JVM 故障排查和调优

## 1. JDK 监控和故障处理工具总结

### 1.1 JDK 命令行工具

> **jps（JVM Process Status）**：类似 UNIX 的 *ps* 命令。用户查看所有 Java 进程的启动类、传入参数和 Java 虚拟机参数等信息；
> **jstat（JVM Statistics Monitoring Tool）：** 用于收集 HotSpot 虚拟机各方面的运行数据；
> **jinfo（Configuration Info for Java）：** Configuration Info fo Java，显示虚拟机配置信息；
> **jmap（Memory Map for Java）：** 生成堆转存储快照；
> **jhat（JVM Heap Dump Browser）：** 用于分析 heapdump 文件，它会建立一个 HTTP/HTML 服务器，让用户可以在浏览器上查看分析结果；
> **jstack（Stack Trace for Java）：** 生成虚拟机当前时刻的线程快照，线程快照就是当前虚拟机内每一条线程正在执行的方法栈的集合。

#### 1.1.1 *jps:* 查看所有 Java 进程

> *jps(JVM Process Status)* 命令类似 UNIX 的 *ps* 命令。  
> *jps:* 显示虚拟机执行主类名称以及这些进程的本地虚拟机唯一 ID（Local Virtual Machine Identified，LVMID）。*jsp -q*：只输出进程的本地虚拟机唯一 ID。   

![jps]

- *jps -l:* 输出主类的全名，如果进程执行的是 Jar 包，输出 Jar 路径。

![jps_-l]
       
- *jps -v:* 输出虚拟机进程启动时

![jps_v]

- *jps -m:* 输出传递给 Java 进程 main() 函数的参数。

#### 1.1.2 *jstat:* 监视虚拟机各种运行状态信息

> jstat(JVM Statistics Monitoring Tool) 使用于监视虚拟机各种运行状态信息的命令行工具。它可以显示本地或者远程（需要远程主机提供 RMI 支持）虚拟机进程中的类信息、内存、垃圾收集、JIT 编译等运行数据，在没有 GUI，只提供了纯文本控制台环境的服务器上，它将是运行期间定位虚拟机性能问题的首选工具。

- jstat 命令使用格式：

    `jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]`  

- 比如 jstat -gc -h3 31736 1000 10表示分析进程 id 为 31736 的 gc 情况，每隔 1000ms 打印一次记录，打印 10 次停止，每 3 行后打印指标头部。

- 常见的 option 如下：

    - jstat -class vmid ：显示 ClassLoader 的相关信息；
    - jstat -compiler vmid ：显示 JIT 编译的相关信息；
    - jstat -gc vmid ：显示与 GC 相关的堆信息；
    - jstat -gccapacity vmid ：显示各个代的容量及使用情况；
    - jstat -gcnew vmid ：显示新生代信息；
    - jstat -gcnewcapcacity vmid ：显示新生代大小与使用情况；
    - jstat -gcold vmid ：显示老年代和永久代的行为统计，从jdk1.8开始,该选项仅表示老年代，因为永久代被移除了；
    - jstat -gcoldcapacity vmid ：显示老年代的大小；
    - jstat -gcpermcapacity vmid ：显示永久代大小，从jdk1.8开始,该选项不存在了，因为永久代被移除了；
    - jstat -gcutil vmid ：显示垃圾收集信息；

- 另外，加上 -t参数可以在输出信息上加一个 Timestamp 列，显示程序的运行时间。

#### 1.1.3 *jinfo:* 实时地查看和调整虚拟机各项参数

- **jinfo vmid：** 输出当前 jvm 进程的全部参数和系统属性 (第一部分是系统的属性，第二部分是 JVM 的参数)。

- **jinfo -flag name vmid：** 输出对应名称的参数的具体值。比如输出 MaxHeapSize、查看当前 jvm 进程是否开启打印 GC 日志 ( -XX:PrintGCDetails :详细 GC 日志模式，这两个都是默认关闭的)。

```vim
C:\Users\SnailClimb>jinfo  -flag MaxHeapSize 17340
-XX:MaxHeapSize=2124414976
C:\Users\SnailClimb>jinfo  -flag PrintGC 17340
-XX:-PrintGC
```

- 使用 jinfo 可以在不重启虚拟机的情况下，可以动态的修改 jvm 的参数。尤其在线上的环境特别有用,请看下面的例子：

- jinfo -flag [+|-]name vmid 开启或者关闭对应名称的参数。

```vim
C:\Users\SnailClimb>jinfo  -flag  PrintGC 17340
-XX:-PrintGC

C:\Users\SnailClimb>jinfo  -flag  +PrintGC 17340

C:\Users\SnailClimb>jinfo  -flag  PrintGC 17340
-XX:+PrintGC
```

#### 1.1.4 jmap:生成堆转储快照

- jmap（Memory Map for Java）命令用于生成堆转储快照。 如果不使用 jmap 命令，要想获取 Java 堆转储，可以使用 “-XX:+HeapDumpOnOutOfMemoryError” 参数，可以让虚拟机在 OOM 异常出现之后自动生成 dump 文件，Linux 命令下可以通过 kill -3 发送进程退出信号也能拿到 dump 文件。

- jmap 的作用并不仅仅是为了获取 dump 文件，它还可以查询 finalizer 执行队列、Java 堆和永久代的详细信息，如空间使用率、当前使用的是哪种收集器等。和 jinfo 一样，jmap有不少功能在 Windows 平台下也是受限制的。

> 示例：将指定应用程序的堆快照输出到桌面。后面，可以通过 jhat、Visual VM 等工具分析该堆文件。

```vim
C:\Users\SnailClimb>jmap -dump:format=b,file=C:\Users\SnailClimb\Desktop\heap.hprof 17340
Dumping heap to C:\Users\SnailClimb\Desktop\heap.hprof ...
Heap dump file created
```

#### 1.1.5 jhat: 分析 heapdump 文件

- jhat 用于分析 heapdump 文件，它会建立一个 HTTP/HTML 服务器，让用户可以在浏览器上查看分析结果。

```vim
C:\Users\SnailClimb>jhat C:\Users\SnailClimb\Desktop\heap.hprof
Reading from C:\Users\SnailClimb\Desktop\heap.hprof...
Dump file created Sat May 04 12:30:31 CST 2019
Snapshot read, resolving...
Resolving 131419 objects...
Chasing references, expect 26 dots..........................
Eliminating duplicate references..........................
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```

#### 1.1.6 jstack :生成虚拟机当前时刻的线程快照

- jstack（Stack Trace for Java）命令用于生成虚拟机当前时刻的线程快照。线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合.

- 生成线程快照的目的主要是定位线程长时间出现停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等都是导致线程长时间停顿的原因。线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做些什么事情，或者在等待些什么资源。

- 下面是一个线程死锁的代码。我们下面会通过 jstack 命令进行死锁检查，输出死锁信息，找到发生死锁的线程。

```java
public class DeadLockDemo {
    private static Object resource1 = new Object();//资源 1
    private static Object resource2 = new Object();//资源 2

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + "get resource1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                }
            }
        }, "线程 1").start();

        new Thread(() -> {
            synchronized (resource2) {
                System.out.println(Thread.currentThread() + "get resource2");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource1");
                synchronized (resource1) {
                    System.out.println(Thread.currentThread() + "get resource1");
                }
            }
        }, "线程 2").start();
    }
}
```

Output
```vim
Thread[线程 1,5,main]get resource1
Thread[线程 2,5,main]get resource2
Thread[线程 1,5,main]waiting get resource2
Thread[线程 2,5,main]waiting get resource1
```

- 线程 A 通过 synchronized (resource1) 获得 resource1 的监视器锁，然后通过Thread.sleep(1000);让线程 A 休眠 1s 为的是让线程 B 得到执行然后获取到 resource2 的监视器锁。线程 A 和线程 B 休眠结束了都开始企图请求获取对方的资源，然后这两个线程就会陷入互相等待的状态，这也就产生了死锁。

- 通过 jstack 命令分析：
    1. 通过 jps -l 找到相应的启动项目 ID
    2. 通过 jstack ID 查看

```vim
Found one Java-level deadlock:
=============================
"线程 2":
  waiting to lock monitor 0x000000000333e668 (object 0x00000000d5efe1c0, a java.lang.Object),
  which is held by "线程 1"
"线程 1":
  waiting to lock monitor 0x000000000333be88 (object 0x00000000d5efe1d0, a java.lang.Object),
  which is held by "线程 2"

Java stack information for the threads listed above:
===================================================
"线程 2":
        at DeadLockDemo.lambda$main$1(DeadLockDemo.java:31)
        - waiting to lock <0x00000000d5efe1c0> (a java.lang.Object)
        - locked <0x00000000d5efe1d0> (a java.lang.Object)
        at DeadLockDemo$$Lambda$2/1078694789.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
"线程 1":
        at DeadLockDemo.lambda$main$0(DeadLockDemo.java:16)
        - waiting to lock <0x00000000d5efe1d0> (a java.lang.Object)
        - locked <0x00000000d5efe1c0> (a java.lang.Object)
        at DeadLockDemo$$Lambda$1/1324119927.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```

> 可以看到 jstack 命令已经帮我们找到发生死锁的线程的具体信息。

### 1.2 JDK 可视化分析工具

#### 1.2.1 JConsole:Java 监视与管理控制台

> JConsole 是基于 JMX 的可视化监视、管理工具。可以很方便的监视本地及远程服务器的 java 进程的内存使用情况。你可以在控制台输出 *console* 命令启动或者在 JDK 目录找到 *jconsole.exe* 然后双击启动。

#### 1.2.2 连接 Jconsole

- 如果需要使用 Jconsole 连接远程进程，可以在远程 Java 程序启动时加上下面这些参数：

```text
-Djava.rmi.server.hostname=外网访问 ip 地址 
-Dcom.sun.management.jmxremote.port=60001   //监控的端口号
-Dcom.sun.management.jmxremote.authenticate=false   //关闭认证
-Dcom.sun.management.jmxremote.ssl=false
```

在使用 JConsole 连接时，远程进程地址如下：

`外网访问 ip 地址:60001`

#### 1.2.3 内存监控

JConsole 可以显示当前内存的详细信息。不仅包括堆内存/非堆内存的整理信息，还可以细化到 eden 区、survivor 区等的使用情况。

> **新生代 GC (Minor GC):** 指发生新生代的垃圾收集动作，Minor GC 非常频繁，回收速度一般也比较快。  
> **老年代 GC（Major GC/Full GC）:** 指发生在老年代的 GC，出现了 Major GC 经常会伴随至少一次的 Minor GC(并非绝对)，Major GC 的速度一般会比 Minor GC 的慢 10 倍以上。

#### 1.2.4 线程监控

类似 **jstack** 命令，不过这个是可视化的。

最下面有一个“检测死锁（D）”按钮，点击这个按钮可以自动为你找到发生死锁的线程以及它们的详细信息。

#### 1.2.5 Visual VM:多合一故障处理工具

VisualVM 提供在 Java 虚拟机（Java Virutal Machine, JVM）上运行的 Java 应用程序的详细信息。在 VisualVM 的图形用户界面中，可以方便、快捷地查看多个 Java 应用程序的相关信息。Visual VM 官网：[https://visualvm.github.io/]。Visual VM 文档：[https://visualvm.github.io/documentation.html]

> VisualVM（All-in-One Java Troubleshooting Tool）是到目前为止随 JDK 发布的功能最强大的运行监视和故障处理程序，官方在 VisualVM 的软件说明中写上了“All-in-One”的描述字样，预示着他除了运行监视、故障处理外，还提供了很多其他方面的功能，如性能分析（Profiling）。VisualVM 的性能分析功能甚至比起 JProfiler、YourKit 等专业且收费的 Profiling 工具都不会逊色多少，而且 VisualVM 还有一个很大的优点：不需要被监视的程序基于特殊 Agent 运行，因此他对应用程序的实际性能的影响很小，使得他可以直接应用在生产环境中。这个优点是 JProfiler、YourKit 等工具无法与之媲美的。

VisualVM 基于 NetBeans 平台开发，因此他一开始就具备了插件扩展功能的特性，通过插件扩展支持，VisualVM 可以做到：

- 显示虚拟机进程以及进程的配置、环境信息（jps、jinfo）。
- 监视应用程序的 CPU、GC、堆、方法区以及线程的信息（jstat、jstack）。
- dump 以及分析堆转储快照（jmap、jhat）。
- 方法级的程序运行性能分析，找到被调用最多、运行时间最长的方法。
- 离线程序快照：收集程序的运行时配置、线程 dump、内存 dump 等信息建立一个快照，可以将快照发送开发者处进行 Bug 反馈。

## 2. JVM 调优

### 2.1 JVM 配置常用参数

#### 2.1.1 Java 内存区域常见配置参数概览

![java_params]

#### 2.1.2 堆参数

|参数|描述|
|:----|:----|
|-Xms|设置 JVM 启动时堆内存的初始化大小|
|-Xmx|设置堆内存最大值|
|-Xmn|设置年轻代的空间大小，剩下的为老年代的空间大小|
|-XX:PermGen|设置永久代内存的初始化大小（JDK 1.8 开始废弃了永久代）|
|-XX:MaxPermGen|设置永久代的最大值|
|-XX:SurvivorRatio|设置 Eden 区和 Survivor 区的空间比例：Eden/S0 = Eden/S1 默认为 8|
|-XX:NewRatio|设置年老代和年轻代的比例大小，默认值为2|
  
#### 2.1.2 回收器参数

|参数|描述|
|:----|:----|
|-XX:+UseSerialGC|串行，Young 区和 Old 区都使用串行，使用复制算法回收，逻辑简单高效，无线程切换开销|
|-XX:+UseParallelGC|并行，Young 区：使用 Parallel scavenge 回收算法，会产生多个线程并回收。通过-XX:ParallelGCThreads=n 参数指定有线程数，默认时是 CPU 核数；Old 区：单线程|
|-XX:+UseParallelOldGC|并行，和 UseParallelGC 一样，Young 区和 Old 区的垃圾回收时都使用多线程收集|
|-XX:+UseConcMarkSweepGC|并发，短暂停顿的并发的收集。Young 区：可以使用普通的或者 parallel 垃圾回收算法，由参数 -XX:+UseParNewGC 来控制；Old 区：只能使用 Concurrent Mark Sweep|
|-XX:+UseG1GC|并行的，并发的和增量式压缩短暂停顿的垃圾收集器。不区分 Young 区和 Old 区空间。它把堆空间划分为多个大小相等的区域。当进行垃圾收集时，它会优先收集存活对象较少的区域，因此叫“Garbage First”|

如上表所示，目前主要有串行、并行和并发三种，对于大内存的应用而言，串行的性能太低，因此使用到的主要是并行和并发两种。并行和并发 GC 的策略通过 UseParallelGC和UseConcMarkSweepGC 来指定，还有一些细节的配置参数用来配置策略的执行方式。例如：XX:ParallelGCThreads， XX:CMSInitiatingOccupancyFraction 等。 通常：Young 区对象回收只可选择并行（耗时间），Old 区选择并发（耗 CPU）。

#### 2.1.3 项目中常用配置

> **备注：** 在 Java8 中永久代的参数 -XX:PermSize 和 -XX:MaxPermSize 已经失效

|参数设置|描述|
|:---|:---|
|-Xms4800m|初始化堆空间大小|
|-Xmx4800m|最大堆空间大小|
|-Xmn1800m|年轻代的空间大小|
|-Xss512k|设置线程栈空间大小|
|-XX:PermSize=256m|永久区空间大小（jdk1.8 开始废弃了永久代）|
|-XX:MaxPermSize=256m|最大永久区空间大小|
|-XX:+UseStringCache|默认开启，启动缓存常用的字符串|
|-XX:+UseConcMarkSweepGC|老年代使用 CMS 收集器|
|-XX:+UseParNewGC|新生代使用并行收集器|
|-XX:ParallelGCThreads=4|并行线程数量4|
|-XX:+CMSClassUnloadingEnabled|允许对类的元数据清理|
|-XX:+DisableExplicitGC|禁止显示的GC|
|-XX:+UseCMSInitiatingOccupancyFraction=68|设置CMS在来年代回收的阈值为68%|
|-verbose:gc|输出虚拟机GC详情|
|-XX:+PrintGCDetails|打印 GC 详情日志|
|-XX:+PrintGCDateStamps|打印 GC 的耗时|
|-XX:+PrintTenuringDistribution|打印 Tenuring 年龄信息|
|-XX:+HeapDumpOnOutOfMemoryError|当抛出 OOM 时进行 HeapDump|
|-XX:HeapDumpPath=/home/admin/logs|指定 HeapDump 的文件路径或目录|

### 2.2 常用 GC 调优策略

#### 2.2.1 GC 调优原则

> 多数的 Java 应用不需要在服务器上进行 GC 优化； 多数导致 GC 问题的 Java 应用，都不是因为我们参数设置错误，而是代码问题； 在应用上线之前，先考虑将机器的 JVM 参数设置到最优（最适合）； 减少创建对象的数量； 减少使用全局变量和大对象； GC 优化是到最后不得已才采用的手段； 在实际使用中，分析 GC 情况优化代码比优化 GC 参数要多得多。

#### 2.2.2 GC 调优目的

将转移到老年代的对象数量降低到最小； 减少 GC 的执行时间。

#### 2.2.3 GC 调优策略

**策略1：** 新对象预留在新生代，由于 Full GC 的成本远高于 Minor GC，因此尽可能将对象分配在新生代是明智的做法，实际项目中根据 GC 日志分析新生代空间大小分配是否合理，适当通过“-Xmn”命令调节新生代大小，最大限度降低新对象直接进入老年代的情况。

**策略2：** 大对象进入老年代，虽然大部分情况下，将对象分配在新生代是合理的。但是对于大对象这种做法却值得商榷，大对象如果首次在新生代分配可能会出现空间不足导致很多年龄不够的小对象被分配的老年代，破坏新生代的对象结构，可能会出现频繁的 full gc。因此，对于大对象，可以设置直接进入老年代（当然短命的大对象对于垃圾回收来说简直就是噩梦）。-XX:PretenureSizeThreshold 可以设置直接进入老年代的对象大小。

**策略3：** 合理设置进入老年代对象的年龄，-XX:MaxTenuringThreshold 设置对象进入老年代的年龄大小，减少老年代的内存占用，降低 full gc 发生的频率。

**策略4：** 设置稳定的堆大小，堆大小设置有两个参数：-Xms 初始化堆大小，-Xmx 最大堆大小。

**策略5：** 注意： 如果满足下面的指标，则一般不需要进行 GC 优化：

> MinorGC 执行时间不到50ms； Minor GC 执行不频繁，约10秒一次； Full GC 执行时间不到1s； Full GC 执行频率不算频繁，不低于10分钟1次。这里是一般的情况 但是实际上还是根据业务来定

[jsp]:..\img\jps.png
[jps_-l]: ..\img\jps_-l.png
[jps_v]: ..\img\jps-v.png
[https://visualvm.github.io/]: https://visualvm.github.io/
[https://visualvm.github.io/documentation.html]: https://visualvm.github.io/documentation.html
[java_params]: https://img-blog.csdnimg.cn/img_convert/bddba4fa822162e06b9a3572e5c9c07c.png