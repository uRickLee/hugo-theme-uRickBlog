---
title: JUC☞Queue
date: 2020-02-02 23:08:24
image: https://gitee.com/uRick/oss/raw/master/blog/队列顶层接口UML.png
description: 队列其实就是一种特殊的链表结构，在JUC下，Doug Lea大师为我们提供了很多线程安全的队列，运用了很多数据结构相关算法知识
categories:
 - Java
tags:
 - 多线程
 - 队列
---

## 1. 队列
### 1.1. 队列基础
队列其实就是一种特殊的链表结构，在JUC下，Doug Lea大师为我们提供了很多线程安全的队列，运用了很多数据结构相关算法知识（可参考：[https://cloud.tencent.com/developer/article/1173781](https://cloud.tencent.com/developer/article/1173781)），其中抽象层UML结构如下：

![队列顶层接口UML](https://gitee.com/uRick/oss/raw/master/blog/队列顶层接口UML.png)

#### 1.1.1. Queue
> **Queue** 是所有队列的一个抽象接口，它实现Collection接口，它提供了最顶层的几个队列操作方法，如下：

```java
public interface Queue<E> extends Collection<E> {
    // 在队列允许的情况下，向队列中添加数据，若元素无法插入到队列，
    // 也就是达到队列容量时会抛出IllegalStateException
    boolean add(E e);
    // 在队列允许的情况下，向队列中添加数据，若元素无法插入到队列,则返回false，
    boolean offer(E e);
    // 移除队列头元素，当队列为空时抛出异常NoSuchElementException
    E remove();
    // 移除队列头元素，区别于remove()，当队列头没有元素时不会抛出异常
    E poll();
    // 检索队列头部元素，但不删除,区别于peek()，它在队列为空时抛出异常NoSuchElementException。
    E element();
    // 检索队列头部元素，但不删除,当队列头没有元素时不会抛出异常。
    E peek();
}
```

#### 1.1.2. BlockingQueue
> **BlockingQueue** 阻塞队列抽象接口，定义了阻塞队列中常用的API

```java
public interface BlockingQueue<E> extends Queue<E> {
    boolean add(E e);
    boolean offer(E e);
    // 提供阻塞的方式向队列插入元素
    void put(E e) throws InterruptedException;
    // 指定等待时间想队列插入元素 
    boolean offer(E e, long timeout, TimeUnit unit)
    // 提供阻塞的方式获取元素
    E take() throws InterruptedException;
    // 指定等待时间，从队列中获取元素
    E poll(long timeout, TimeUnit unit)throws InterruptedException;
    // 获取队列剩余容量，官方建议：不能用它检测元素是否插入成功操作。
    int remainingCapacity();
    boolean remove(Object o);
    // 检测队列是否包括某个元素
    public boolean contains(Object o);
    // 将队列中的数据移动到另一个集合中，并清空当前队列
    int drainTo(Collection<? super E> c);
    // 队列中的数据移动到另一个集合中，maxElements是指订的最大传输元素个数，同时队列会移除指定maxElements的元素
    int drainTo(Collection<? super E> c, int maxElements);
}
```

阻塞队列中常用方法归纳概括：

| **方法/处理方式** |   **抛异常**    | **返回特殊值** | **阻塞** |              **超时退出**              |
| :----------------: | :--------------: | :-------------: | :------: | :-------------------------------------: |
|   **插入方法**    |     add(E e)     |   offer(E e)    | put(E e) | offer(E e, long timeout, TimeUnit unit) |
|   **移除方法**    | remove(Object o) |     poll()      |  take()  |    poll(long timeout, TimeUnit unit)    |
|   **检查方法**    |    element()     |     peek()      |    -     |                    -                    |


#### 1.1.3. BlockingDeque
> BlockingDeque 阻塞双端队列抽象接口，所谓双端队列就是一个支持队列两端操作元素出队和入队，定义了阻塞队列中常用的API。
> BlockingDeque 继承BlockingQueue接口，BlockingDeque在BlockingQueue的基础上新增双端阻塞队列的操作方法；主要新增了以first结尾和last结尾的方法，用于操作队列两端的元素。

```java
public interface BlockingDeque<E> extends BlockingQueue<E>, Deque<E> {
    void addFirst(E e);
    void addLast(E e);
    boolean offerFirst(E e);
    boolean offerLast(E e);
    void putFirst(E e) throws InterruptedException;
    void putLast(E e) throws InterruptedException;
    boolean offerFirst(E e, long timeout, TimeUnit unit)throws InterruptedException;
    boolean offerLast(E e, long timeout, TimeUnit unit)throws InterruptedException;
    E takeLast() throws InterruptedException;
    E pollFirst(long timeout, TimeUnit unit)throws InterruptedException;
    E pollLast(long timeout, TimeUnit unit)throws InterruptedException;
    boolean removeFirstOccurrence(Object o);
    boolean removeLastOccurrence(Object o);
    boolean add(E e);
    boolean offer(E e);
    void put(E e) throws InterruptedException;
    boolean offer(E e, long timeout, TimeUnit unit)throws InterruptedException;
    E remove();
    E poll();
    E take() throws InterruptedException;
    E poll(long timeout, TimeUnit unit)throws InterruptedException;
    E element();
    E peek();
    boolean remove(Object o);
    public boolean contains(Object o);
    public int size();
    Iterator<E> iterator();
    void push(E e);
}
```

双端队列中常用方法归纳概括：

- 添加元素队头相关方法

| **方法/处理方式** |  **抛异常**  | **返回特殊值** |  **阻塞**  |       **超时退出**       |
| :----------------: | :-----------: | :-------------: | :---------: | ------------------------- |
|   **插入方法**    |  addFirst(e)  |  offerFirst(e)  | putFirst(e) | offerFirst(e, time, unit) |
|   **移除方法**    | removeFirst() |   pollFirst()   | takeFirst() | pollFirst(time, unit)     |
|   **检测方法**    |  getFirst()   |   peekFirst()   |      -      | -                         |

- 添加元素到队尾相关方法

| **方法/处理方式** | **抛异常**  | **返回特殊值** | **阻塞**  |      **超时退出**       |
| :----------------: | :----------: | :-------------: | :--------: | ------------------------ |
|   **插入方法**    |  addLast(e)  |  offerLast(e)   | putLast(e) | offerLast(e, time, unit) |
|   **移除方法**    | removeLast() |   pollLast()    | takeLast() | pollLast(time, unit)     |
|   **检测方法**    |  getLast()   |   peekLast()    |     -      | -                        |



### 1.2. 常用队列
|          **队列**           |                                             **说明**                                             | **版本** |
| :--------------------------: | ------------------------------------------------------------------------------------------------- | :-------: |
| **ConcurrentLinkedQueue** | 基于链接节点的无界线程安全非阻塞队列，它采用先进先出的规则对节点进行排序，使用CAS算法实现元素操作 |  JDK1.5   |
| **ConcurrentLinkedDeque** | 基于链接节点的无界线程安全双端队列，它采用先进先出的规则对节点进行排序，使用CAS算法实现元素操作   |  JDK1.7   |
|  **ArrayBlockingQueue**   | 基于数组实现的有界阻塞队列，按照FIFO的原则对元素进行排序                                          |  JDK1.5   |
|  **LinkedBlockingQueue**  | 基于链表实现的有界阻塞队列，默认容量和最大长度为Integer.MAX_VALUE，按照FIFO的原则对元素进行排序   |  JDK1.5   |
|   **SynchronousQueue**    | 不存储元素的阻塞队列，每一个put操作必须等待一个take操作，否则不能继续添加元素                     |  JDK1.5   |
|  **LinkedTransferQueue**  | 由链表结构组成的无界阻塞TransferQueue队列，与其他队列的主要区别在于多了tryTransfer和transfer方法 |  JDK1.7   |
| **PriorityBlockingQueue** | 基于数组实现的有界阻塞优先级队列，默认按照自然顺序排序，也通过比较器进行排序                      |  JDK1.5   |
|     **PriorityQueue**      | 基于数组实现的有界非阻塞优先级队列，默认按照自然顺序排序，也通过比较器进行排序                    |  JDK1.5   |
|       **DelayQueue**       | 使用优先级队列PriorityQueue来实现，是一个支持延时获取元素的无界阻塞队列                           |  JDK1.5   |
|  **LinkedBlockingDeque**  | 由链表结构组成的双向阻塞队列                                                                      |  JDK1.6   |


#### 1.2.1. ConcurrentLinkedQueue
>  一个基于链接节点的无界（可以一直向队列插入数据）程安全的非阻塞的队列，基于CAS自旋方式实现。采用FIFO的规则对节点进行排序，元素入队时添加到队列尾部，出队时从队列头部取出元素。

> ConcurrentLinkedQueue是由一个节点头head和一个尾节点tail组成，缺省情况下，head等于tail,队列存储一个空节点。而节点是由节点项item和节点关系next组成的，如下源码片段：

```java
// 队列组成
private transient volatile Node<E> head;
private transient volatile Node<E> tail;

// 节点
private static class Node<E> {
        volatile E item;
        volatile Node<E> next;
}
```

JUC中所有队列都实现了接口QueueAPI，通用API如下：

- public E peek() 获取头部元素，但是不会出队（删除）
- public E poll() 元素出队，删除头元素，让下一个元素指向head头
- boolean add(E e) 元素入队，调用offer方法入队
- boolean offer(E e); 元素入队
- boolean isEmpty(E e); 检测队列元素是否为空

#### 1.2.2. ArrayBlockingQueue
一个基于数组实现的有界阻塞队列，默认情况下多线程访问队列是非公平的（多线程竞争访问队列资源）；如下源码片段中构造方法表明ArrayBlockingQueue对象创建需要设置一个容量，默认ReentrantLock重入锁来实现队列访问控制的，使用一个数组items维护队列元素，定义两个监控等待队列notEmpty和notFull成员变量。

```java
// 指定固定容量创建队列
public ArrayBlockingQueue(int capacity) {
    this(capacity, false);
}

// 指定容量创建队列，同时定义线程访问队列的方式：公平或非公平
public ArrayBlockingQueue(int capacity, boolean fair) {
    if (capacity <= 0)
        throw new IllegalArgumentException();
    this.items = new Object[capacity];
    lock = new ReentrantLock(fair);
    notEmpty = lock.newCondition();
    notFull =  lock.newCondition();
}

// 指定的集合初始化队列，容量需定义指定集合大小，否则抛出异常IllegalArgumentException
 public ArrayBlockingQueue(int capacity, boolean fair, Collection<? extends E> c) {
        this(capacity, fair);
        final ReentrantLock lock = this.lock;
        lock.lock(); // Lock only for visibility, not mutual exclusion
        try {
            int i = 0;
            try {
                for (E e : c) {//遍历集合c，存入队列中
                    checkNotNull(e);
                    items[i++] = e;
                }
            } catch (ArrayIndexOutOfBoundsException ex) {
                throw new IllegalArgumentException();
            }
            count = i;
            putIndex = (i == capacity) ? 0 : i;
        } finally {
            lock.unlock();
        }
    }
```

#### 1.2.3. LinkedBlockingQueue
LinkedBlockingQueues是基于链表Node节点来实现的一个有界阻塞队列，队列默认容量和最大容量都为 ***Integer.MAX_VALUE*** ，也是通过重入锁实现多线程访问控制的，它通过两个锁来维护元素的出队和入队，一个putLock锁控制元素入队操作，一个takeLock锁控制出队操作。

#### 1.2.4. PriorityBlockingQueue
它是一个基于数组实现自定义排序的无界阻塞的优先级队列，队列初始容量为11，当元素存储超过指定容量或者初始容量时，扩增为原容量的50%；  
队列中的元素时按照自然顺序排序的，可以根据需求自定排序方式，同时注意同一级别的元素时不能保证优先级的。它提供了四种构造方式：

```java
// 默认创建队列，容量为11
public PriorityBlockingQueue() {
    this(DEFAULT_INITIAL_CAPACITY, null);
}

// 指定容量创建队列，比较器使用默认的排序规则（自然排序），可查看排序算法siftUpComparable
public PriorityBlockingQueue(int initialCapacity) {
    this(initialCapacity, null);
}

// 指定容量，和比较器创建队列
public PriorityBlockingQueue(int initialCapacity,
                             Comparator<? super E> comparator) {
    if (initialCapacity < 1)
        throw new IllegalArgumentException();
    this.lock = new ReentrantLock();// 获取重入锁
    this.notEmpty = lock.newCondition();// 构建监控队列
    this.comparator = comparator;// 指定比较器
    this.queue = new Object[initialCapacity];// 初始化数组
}

// 通过给定集合创建队列
public PriorityBlockingQueue(Collection<? extends E> c)
```

#### 1.2.5. SynchronousQueue
SynchronousQueue 是一个同步阻塞队列，也可以说是数据传输队列，它是不存储任何元素；每个put入队操作都必须等待一个take出队操作，可以将SynchronousQueue理解最简易的中间件，消息生产者负责将消息存储队列中，消费者需要从队列中取出消息，当消费者没有消费消息，则生产者会一直等待消费者消费，当消费者没有可消费的消息，会一直等待生产者生产消息；  SynchronousQueue 内部分别定义了TransferQueue类和TransferStack类来实现公平访问和非公平访问策略,可以通过以下两种方式创建队列。

```java
// 默认创建非公平策略的队列
public SynchronousQueue() {
        this(false);
    }

// 自定义同步策略
public SynchronousQueue(boolean fair) {
    transferer = fair ? new TransferQueue<E>() : new TransferStack<E>();
}
```

#### 1.2.6. DelayQueue
基于PriorityQueued队列实现的无界阻塞延时队列，队列中的元素都必须要实现Delayed接口，创建元素时可以指定多久才能从队列中获取当前元素。只有在延迟期满时才能从队列中提取元素，可用于做缓存或定时任务。

#### 1.2.7. LinkedTransferQueue
LinkedTransferQueue是一个基于链表结构的即时传输无界阻塞队列，实现了TransferQueue接口，相对于以上介绍的队列，它新增了两种方法transfer和trytransfer  
对于transfer（）类方法，如果当前有消费者正在等待接收元素，transfer()可以把生产者传入的元素立刻transfer（传输）给消费者。如果没有消费者在等待接收元素，transfer方法会将元素存放在队列的tail节点，并等到该元素被消费者消费了才返回。
trytransfer()与transfer()相反,它会试探性的传输数据给消费者，若没有消费者消费信息，则直接返回false,若存在直接传输，返回true。