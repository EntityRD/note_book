# HashMap
## 1. 简述

- HashMap 是由数组和链表组合构成的数据结构，数组里每个地方存了 Key-Value 这样的实例，jdk1.7 叫做 Entry，jdk1.8 叫做 Node。初始化是所有位置都是 null，put 插入的时候会根据 Key 的 hash 函数去计算出一个 index 值，比如 index 是2，就将这个 Key-Value 插入到数组下标为2的位置。

## 2. 为什么需要链表，链表又是什么样子的？

- 由于数组的长度是有限的，在有限的数组长度使用哈希，不同的 key 经过 hash 会 hash 到同一个值上，这样就形成了链表。
```java
/**
 * Basic hash bin node, used for most entries. (See below for TreeNode
 * subclass, and in LinkedHashMap for its Entry subclass.)
 */
 static class Node<K, V> implements Map.Entry<K, V> {
     final int hash;
     final K key;
     V value;
     Node<K, V> next;

     Node(int hash, K key, V value, Node<K, V> next){
         this.hash = hash;
         this.key = key;
         this.value = value;
         this.next = next;
     }
 }
```
看 Node 的源码，可以知道：每一个节点保存了自身的 hash 值，key、value，以及下一个节点的信息。

## 3. 新的节点 Node 在插入时，它的插入方式是什么？

- JDK1.8 之前采用的是头插发，就是说新来的值会取代原有的值，原来的值就会被顺推到链表上去。写这个代码的作者认为后来的值被查询的可能性会大一点，为了提高查询效率。

- JDK1.8 之后，都是用的尾插法。尾插法在扩容时会保持链表元素原来的顺序，不会出现链表成环的问题。java8 链表有红黑树的部分，红黑树的引入巧妙的将原本 O(n) 的时间复杂度降低到了 O（logn）

## 4. 为什么改成尾插法了？

- 因为头插法会改变链表上数据的顺序，可能会出现 Infinite Loop，所以 JDK1.8 之后，都是用的尾插法。
    - 一个容量为 2 的 Entry 数组， 使用**不通的线程**插入A，B，C，假设在 resize 之前打一个断点，那就意味着数据都插入了，但是还没有扩容 resize。结果可能是：在数组同一条 Entry 链上 A->B->C(图1)；  
      ![pic_1]  
    因为 1.7 是头插法，resize 后，在旧数组同一条 Entry 链上的元素，通过计算可能会在不通的位置上，可能的结果如下  
    ![pic_2]  
    一旦所有线程完成，可能出现环形链表，导致 Infinite Loop 如图三  
    ![pic_3]  

## 5. HashMap 的扩容机制

- 由于数组的容量是有限的，数据多次插入的时候，到达一定的容量时，就会进行扩容，也就是 resize。

## 6. 什么时候扩容（resize）？

- 两个因素：
    - Capatity：HashMap 的长度
    - LoadFactoy：负载因子，默认值时 0.75f

    > eg：一个 HashMap 的初始容量是 100，当你存储第 76 个的时候，判断的时候就发现需要 resize 了。

## 7. HashMap 怎么进行扩容？

- 扩容分两步：
    1. 扩容：创建一个新的 Entry 空数组，长度是原来的两倍
    2. ReHash：遍历原来的 Entry 数组，把所有的 Entry 重新 Hash 到新的数组中去。  
   **问题：** 为什么要重新 Hash?  
   **答：** 长度扩大了，Hash 的规则也随之改变。Hash 的公式：index = HashCode (key) & (Length-1)

## 8. Java8 HashMap 是否可以用在多线程上？

- java7 在多线程操作的时候 HashMap 可能出现死循环，原因就是扩容转移后，前后链表顺序倒置，在转移过程中修改了原来链表中节点的引用关系。
- java8 在同样的前提下不会引起死循环，原因时扩容转移后，前后链表顺序不变，保持之前节点的引用关系。
- java8 在同样的前提下不会引起死循环，原因是扩容转移后，前后链表顺序不变，保持之前节点的引用关系。
- java8 即使不会出现死循环，但是通过源码看到 put/get 方法没有加同步锁。多线程情况最容易出现的就是：无法保证上一秒 put 的值，下一秒 get 的时候还是原值，所以线程安全还是无法保证。

## 9. HashMap 的默认初始长度是多少？

> 16

```java
/**
 * The default initial capacity - MUST be a power of two.
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
```
- 之所以是 16，是为了服务将 key 映射到 index 的算法，也是为了保证得到一个均匀分布的 hash

## 10. 为什么重写 equals 方法的时候，需要重写 hashCode?（用 HashMap 举例）

- equals:
    - 对于值对象：== 比较的是两个对象的值
    - 对于引用对象：比较的是两个对象的地址

> 以保证相同的对象返回相同的 hash 值，不同的对象返回不同的 hash 值

[pic_1]: https://img-blog.csdnimg.cn/20200709175448235.png
[pic_2]: https://img-blog.csdnimg.cn/20200709175537966.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MzcxMzIz,size_16,color_FFFFFF,t_70
[pic_3]: https://img-blog.csdnimg.cn/20200709175602836.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MzcxMzIz,size_16,color_FFFFFF,t_70