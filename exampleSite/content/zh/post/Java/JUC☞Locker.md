---
title: JUC☞Locker
date: 2020-01-12 23:08:24
image: https://gitee.com/uRick/oss/raw/master/blog/AQS内部关系.png
description: 出世于JDK5，相对于synchronized实现锁，Lock更加的灵活更加的强大,显示的方式获取锁资源和释放所资源，synchronized提供隐式获取锁和释放锁；Lock需要手动处理的获取锁和释放锁，为了安全性，使用者必须能够清晰的掌握原理。  
categories:
 - Java
tags:
 - 多线程
 - 锁
---

## 1. Locker是什么？
出世于JDK5，相对于synchronized实现锁，Lock更加的灵活更加的强大,显示的方式获取锁资源和释放所资源，synchronized提供隐式获取锁和释放锁；Lock需要手动处理的获取锁和释放锁，为了安全性，使用者必须能够清晰的掌握原理。  

JUC中使用的锁通常有java.util.concurrent.locks.Lock、java.util.concurrent.locks.ReadWriteLock、java.util.concurrent.locks.ReentrantLock、java.util.concurrent.locks.ReentrantReadWriteLock；都是基于Lock接口实现的，而Lock锁的实现是依赖于底层队列的。

![java.util.concurrent.locks包下类](https://gitee.com/uRick/oss/raw/master/blog/java.util.concurrent.locks包下类.png)

 如图，白色虚线表示依赖关系，绿色虚线表示实现关系，蓝色实线表示继承；从UML类图中可以看出整个锁的实现机制，所有的锁实现都是基于队列实现的。  

**Condition**  接口是一个锁的监控接口，该接口提供类似与Object中的wait()、wait(long timeout)、notify()以及notifyAll()方法类似，可以实现线程之间的通信；  
**Lock** 接口是一个抽象锁的定义；  
**ReentantLock** 是基于Lock实现的一个重入锁，支持公平锁与非公平锁；  
**ReentantLockReadWriteLock** 顾名思义是实现读写锁的可重入锁，它还支持锁的降级处理。实现了ReadWriteLock接口，ReadWriteLock接口定义了两个方法readLock()/writeLock();    
**StampedLock** 是JDK1.8新增加的锁，可以说是读写锁的一个改进版，实现一种新的读写策略；  
**LockSupport** 类是一个线程操作静态工具类，它提供一些操作线程的静态方法；  
**AbstractOwnableSynchronizer** 是同步器的抽象父类； AbstractQueuedSynchronizer 是用来构建锁或者其他同步组件的基础框架，它使用了一个int成员变量表示同步状态，通过内置的FIFO队列来完成线程的排队工作。ReentantLock、ReentantLockReadWriteLock都定义一个实现AQS的同步器Sync。  
**AbstractQueuedLongSynchronizer** 是一个长状态的同步器，与AQS在结构上没有区别，只是状态类型变为long类型了。 

## 2. AQS（AbstractQueuedSynchronizer）
> 如上，JUC中Locker的核心就是AQS,下面探讨AQS；在Lock 所有锁的实现都是基于AQS实现的，主要通过AQS继承的方式来管理线程的状态，通过一个内置的FIFO队列来完成取线程的排队工作。对于锁来说面向使用者的，它定义了使用者与锁交互的接口（比如可以允许两个线程并行访问），隐藏了实现细节；而AQS是锁的实现者，它简化了锁的实现方式，屏蔽了同步状态管理、线程的排队、等待与唤醒等底层操作。

1. AQS内部关系
![AbstractOwnableSynchronizer](https://gitee.com/uRick/oss/raw/master/blog/AQS内部关系.png)


2. 队列的内部结构
![队列的内部结构](https://gitee.com/uRick/oss/raw/master/blog/队列的内部结构.png)

如图，AQS通过节点Node 的方式来实现一个队列排队，以及线程排队状态的管理，遵守FIFO原则分配和管理资源；为了描述线在队列中排队获取资源的不同状态，引入同步状态（waitStatus）
AQS中的同步状态（waitStatus）分一下几种：

   1. **INITIAL**  值为0，初始状态
   2. **CANCELLED** 值为1，同步队列中的等待的线程等待超时或者中断，需要从同步队列中取消等待，节点进入该状态将不会变化
   3. **SIGNAL** 值为-1，后继节点的线程处于等待状态，而当前节点的线程如果释放了同步状态或者被取消，将通知后继节点，使后继几点的线程运行
   4. **CONDITION** 值为-2,节点在等待队列中，节点线程等待在Condition上，当其他线程对Condition调用了signal()方法，该节点将会从等待队列中移动到同步队列中，加入到获取锁的的队列中去 
   5. **PROPAGATE** 值为-3，表示下一次共享式同步状态获取将会无条件的被传播下去 

如图：AQS中包含两种节点类型，一个是头节点，一个是尾节点。当线程获取资源时，获取队列同步状态失败，AQS会构造一个节点存储线程信息并将节点放入到同步队列的尾部（尾节点），而为了保证线程的安全性，尾节点的设置是需要通过CAS方法设置的；当获取到同步状态后，再从队列中移除，将当前移除节点的下一个节点设置为头节点。在AQS中有两种方式获取同步状态（锁资源），一种是独占式获取锁资源，独占式就是同一时刻只能有一个线程获取到队列同步状态；另一个是共享式获取锁资源，同一时刻可以有多个线程获取资源。

## 3. 电子眼（Condition）
通常我们创建的每一个对象都有一组监控方法，而这些方法设置在Object（`notify()/notifyAll/wait()/wait(long timeout, int nanos)`）中的；这些方法配合线程使用时，就打通了多个线程间沟通隔阂，实现线程通信，它们都是基于JVM层面来实现的，所有的相关操作都由JVM操控。
同理，Condition也具备一组多线程通信的监控方法，它更灵活更强大一些，它的实现在AQS内部ConditionObject中，也是基于队列实现的；为什么会定义在AQS中呢，那是因为JUC下所有的Locker都是基于AQS实现的。
![Condition](https://gitee.com/uRick/oss/raw/master/blog/Condition.png)

怎获得Condition呢？
> 通常Locker内部都会实现AQS，而AQS中定义了Condition的实现，所有相关的Locker可以获得ConditionObject，具体使用方法可查看JUC下相关队列实现（后续介绍），可参考获取方法如下：

```java
  final ConditionObject newCondition() { return new ConditionObject(); }
```
![Condtion结构](https://gitee.com/uRick/oss/raw/master/blog/Condtion结构.png)

如图：Condition基于FIFO队列实现，当调用await方法，把当前线程封装为一个节点加入到Condition队尾中，当线程被唤醒时，从Condition队列头部移除加入到AQS队列尾部，参与竞争资源。其实内部实现并不是特别复杂，可以查看源码理解。

**Condition**核心方法

| 方法                                | 功能                                                    |
| :---------------------------------- | :----------------------------------------------------- |
| await() throws InterruptedException | 表示当前线程进入等待状态，直到被`signal`通知，或者被中断 |
| awaitUninterruptibly()              | 表示当前进入等待状态， 直到被唤醒，不支持中断           |
| awaitNanos(long nanosTimeout)       | 当前线程进入等待状态直到被通知、中断或者超时            |
| awaitUntil(Date deadline)           | 当前线程进入等待状态直到被通知、中断或等待到某个时间    |
| await(long time, TimeUnit unit)     | 同await()， unit指定超时单位                            |
| signal()                            | 唤醒当前Condition队列上Head等待中线程                   |
| signalAll()                         | 唤醒当前Condition队列上所有等待中线程                   |

**洞悉内部细节↓**

1. *await*

```java
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    Node node = addConditionWaiter();//构造Node加入到Condition队列上
    int savedState = fullyRelease(node);//释放Node当前的锁，得到锁的状态state，并唤醒AQS队列中的一个线程
    int interruptMode = 0;// 中断标识
    //判断Node是在AQS队列上，若不在就执行park等待
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
        //判断当前node是否中断，若没有中断则继续执行while判断node是否在AQS队列上，
        //没在队列中或node状态还是CONDITION状态，则继续循环
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    // node被唤醒后去获取锁资源，若没获取成功，且interruptMode != THROW_IE时，则设置interruptMode = REINTERRUPT;
    // 没获取成功的node会加入到AQS队列队尾排队
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // 清理Condition队列资源，其实就是移动节点，将被唤醒的node移除
        unlinkCancelledWaiters();
    if (interruptMode != 0)//若发生中断，则进行中断操作处理
        reportInterruptAfterWait(interruptMode);
}
```

2. *signal*

```java
public final void signal() {
    if (!isHeldExclusively())//判断当前线程是否持有锁，从这里看出线程必须持有锁进行相关操作
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignal(first);
}

//将调用await的当前线程唤醒，并加入到AQS队列中
private void doSignal(Node first) {
do {
    if ( (firstWaiter = first.nextWaiter) == null)
        lastWaiter = null;
    first.nextWaiter = null;
} while (!transferForSignal(first) && (first = firstWaiter) != null);
}

final boolean transferForSignal(Node node) {
/*
 * 若CAS状态失败，则说明已经被取消
 */
if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
    return false;

// 将node加入到AQS队中获取锁资源
Node p = enq(node);
int ws = p.waitStatus;
// ws>0表示p节点已经被取消了，则会去唤醒下一个节点的线程；waitStatus状态可查看上文AQS小节介绍
if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
    LockSupport.unpark(node.thread);
return true;
}
```

*其实Condition的await就是从AQS出队到Condition队列入队的过程，反之就是signal；上述简单解析了等待阻塞与唤醒操作的内部细节，其他变形方法原理大同小异*

## 4. 回头草（ReentrantLock）
重入锁，是指任意线程在获取到锁之后能够再次获取该锁而不会被锁所阻塞，当线程调用lock()方法获取到同步锁后，可以再次调lock()获取到锁而不会别阻塞；需要注意的是线程获取多少次锁，就需要释放多少次锁。

```java
public class ReentrantTest {
    private static Lock lock = new ReentrantLock();
    private static int i = 0;
    public static void main(String[] args) throws InterruptedException {
        for (int j = 0; j < 5; j++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    // 获取锁资源建议不要放到try代码块中，以防止出现异常，释放锁
                    lock.lock();
                    System.out.println("^");
                    try {
                        TimeUnit.SECONDS.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }finally {
                        lock.unlock();
                        System.out.println("v");
                    }
                    i++;
                }
            },"ReentrantLock").start();
        }
        TimeUnit.SECONDS.sleep(10);
        System.out.println(i);
    }
}
```

![ReentrantLock内部实现结构](https://gitee.com/uRick/oss/raw/master/blog/ReentrantLock内部实现结构.png)

如图ReentrantLock内部自定义Sysnc继承AQS实现个性化定制锁，通过自定义实现AQS屏蔽了获取锁和释放锁方式；它通过sync与NonfiarSync实现公平锁和非公平锁，默认ReentrantLock是实现的非公平锁，决定使用公平锁还是非公平锁是根据构造函数参数决定的，所谓公平锁是指等待时间长的线程有最先获取所资源的权限，其实它就是严格遵守FIFO原则的，而非公平锁则是采用抢占的策略的，不管AQS队列中是否存在等待的线程，它也先CAS参与竞争锁。

```java
public ReentrantLock() {
        // 默认创建非公平同步器
        sync = new NonfairSync();
    }

// 指定参数构造，fair=true 创建公平同步器
public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

**接下来探讨一下加锁与释放锁的流程**

![加锁与释放锁的时序图](https://gitee.com/uRick/oss/raw/master/blog/加锁与释放锁的时序图.png)

1. FairSync加锁

```java
final void lock() {
    acquire(1);//这里加锁调用AQS方法acquire，而AQS会调用FairSync内部实现方法tryAcquire
}

// 这里是AQS获取锁通用处理逻辑，而具体获取锁的方式和细节交由具体的实现内实现
// 尝试获取锁，若失败则添加到AQS队列队尾
public final void acquire(int arg) {
//addWaiter将节点添加AQS队尾，添加成功后通过方法acquireQueued自旋的方式获取锁资源
//若被中断了，则这些中断处理
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();//获取队列状态
    if (c == 0) {//状态0表示无锁
    // 判断当前线程是不是队列头节点，若是则设置状态获得锁资源
        if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);//设置线程独占锁
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {//若是统一个线程，直接累加状态即可，不再需要抢占资源，这里便是重入
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

2. FairSync释放锁
对于释放锁走资源，公平与非公平锁都是一样的，主要区别在于加锁方式。

```java
public void unlock() {
    sync.release(1);
}

public final boolean release(int arg) {
    if (tryRelease(arg)) {//释放锁资源
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);//若释放锁资源成功则，唤醒head后续节点
        return true;
    }
    return false;
}

//释放锁资源，getState状态减去指定的状态值
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())//若持有锁的线程不是当前线程，则抛出异常IllegalMonitorStateException
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {//若c=0则清除持锁独占的线程
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);//设置状态，释放成功
    return free;
}
```

3. 公平锁与非公平锁
从上述流程图和构造方法中可以看出，公平锁与非公平锁的区别主要在于加锁指定的同步器不同，非公平锁实现在获取锁资源的时候，首去抢占资源，而公平锁则很老实的遵守FIFO原则获取锁资源；下面对比一下两者的加锁方式：

-  **NonfairSync**

```java
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = 7316153563782823691L;
    final void lock() {
    //加锁时优先CAS获取锁资源，若获取锁成功，则设置独占线程，否则与FairSync获取锁逻辑一样
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }

    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
```

- **FairSync**

```java
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;
    final void lock() {
        acquire(1);
    }
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();//获取队列状态
    if (c == 0) {//状态0表示无锁
    // 判断当前线程是不是队列头节点，若是则设置状态获得锁资源
        if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);//设置线程独占锁
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {//若是统一个线程，直接累加状态即可，不再需要抢占资源，这里便是重入
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```


## 5. 多面手（ReentrantReadWriteLock）
读写锁包含两个锁，一个读锁，读锁是多线程共享的，而另一个是写锁，写锁是独占的，排他的；通过以下案例运行输出结果可以看出同一时间多个线程不能同时获取到写锁的，而读锁是可以的；

> 在执行写操作是，线程必须要获取写锁，当已经有线程持有写锁的情况下，当前线程会被阻塞，只有当写锁释放以后，其他读写操作才能继续执行。使用读写锁提升读操作的并发性，也保证每次写操作对所有的读写操作的可见性。*读锁与读锁可以共享/读锁与写锁不可以共享/写锁与写锁不可以共享*

```java
public class ReadWriteLockDemo {
    private static final Map<String, Object> DATA = new HashMap<>();
    private static ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    private static Lock readLock = readWriteLock.readLock();
    private static Lock writeLock = readWriteLock.writeLock();

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 20; i++) {
            WriteThread writeThread = new WriteThread("Key-" + i, "Data-" + i);
            writeThread.start();
            TimeUnit.SECONDS.sleep(1);
        }

        for (int i = 0; i < 20; i++) {
            ReadThread readThread = new ReadThread("Key-" + i);
            readThread.start();
        }

    }

    static class ReadThread extends Thread {
        private String key;

        public ReadThread(String key) {
            this.key = key;
        }

        @Override
        public void run() {
            get(key);
        }
    }

    static class WriteThread extends Thread {
        private String key;
        private Object data;

        public WriteThread(String key, Object data) {
            this.key = key;
            this.data = data;
        }

        @Override
        public void run() {
            set(key, data);
        }
    }

    public static void set(String key, Object data) {
        try {
            writeLock.lock();
            DATA.put(key, data);
            System.out.println("----->> 写入数据：" + data);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            writeLock.unlock();
        }
    }

    public static Object get(String key) {
        try {
            readLock.lock();
            Object data = DATA.get(key);
            System.out.println("------->> 获取数据：" + data);
            return data;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readLock.unlock();
        }
        return null;
    }
}
```

在读写锁也是支持公平和非公平两种方式的锁的；也支持锁的重入，以读写为例：读线程获取到了读锁之后，能够再次获取读锁，而写线程获取了写锁之后能够再次获取读写锁，同时也可以获取读锁(锁降级)。

*接下来通过如下结构图，进一步深入探讨它的内部结构*

![ReentrantReadWriteLock结构](https://gitee.com/uRick/oss/raw/master/blog/ReentrantReadWriteLock结构.png)

如图，ReentrantReadWriteLock定义基于Lock接口实现读写锁ReadLock/WriteLock，通过定制Sync实现读写能力，下面通过读写时序图理解参悟它的原理：

![ReentrantReadWriteLock读写锁内部方法调用关系](https://gitee.com/uRick/oss/raw/master/blog/ReentrantReadWriteLock读写锁内部方法调用关系.png)


**ReadLock**

- 获取读锁

```java
// 1.java.util.concurrent.locks.ReentrantReadWriteLock.ReadLock#lock
public void lock() {
    sync.acquire(1);
}
// 2.java.util.concurrent.locks.AbstractQueuedSynchronizer#acquireShared
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        //若没能拿到锁资源，则进行入队操作，并轮询拿锁，除非被中断，否则一直等待
        doAcquireShared(arg);
}

// 3.java.util.concurrent.locks.ReentrantReadWriteLock.Sync#tryAcquireShared
protected final int tryAcquireShared(int unused) {
    Thread current = Thread.currentThread();
    int c = getState();
    // 判断是否有线程持有排他锁，且持有锁的线程不是当前线程，则获取锁失败
    if (exclusiveCount(c) != 0 && getExclusiveOwnerThread() != current) return -1;
    int r = sharedCount(c);//持有共享锁的线程数
    //判断获取读锁是否需要阻塞，且共享锁持有线程数是否小于MAX_COUNT(65535),则更新锁状态
    if (!readerShouldBlock() && r < MAX_COUNT && compareAndSetState(c, c + SHARED_UNIT)) {
        if (r == 0) {//若r为0，则设置first线程为当前线程，计数器设为1
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            firstReaderHoldCount++;//若持有first线程为当前线程，则计数器累加（重入）
        } else {//若以上条件都不满足，则根据本地缓存的计数器更新读锁线程持有数量
            HoldCounter rh = cachedHoldCounter;//通过UnSafe类获取线程TID来保证唯一性，设置计数
            if (rh == null || rh.tid != getThreadId(current)) cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    // 若以上条件都不满足则通过fullTryAcquireShared尝试获取，内部判断逻辑同上几乎没有多大区别
    return fullTryAcquireShared(current);
}
```

- 释放读锁

```java
// 1.java.util.concurrent.locks.ReentrantReadWriteLock.ReadLock#unlock
public void unlock() {
    sync.releaseShared(1);
}

// 2.java.util.concurrent.locks.AbstractQueuedSynchronizer#releaseShared
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {//调用3中方法释放锁资源
        doReleaseShared();//获取锁资源，唤醒后续节点
        return true;
    }
    return false;
}

// 3.java.util.concurrent.locks.ReentrantReadWriteLock.Sync#tryReleaseShared
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    if (firstReader == current) {//firstReader是否是当前线程
        // assert firstReaderHoldCount > 0;
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();
        int count = rh.count;
        //若线程持锁数小于等于1，则移除从ThreadLocalHoldCounter中移除这个计数器
        //因为这个线程释放读锁后，不会在持有这个计数器了
        if (count <= 1) {
            readHolds.remove();
            if (count <= 0)//释放读锁资源后，计数器小于等于零说明线程需要释放的锁不匹配当前线程
                throw unmatchedUnlockException();
        }
        --rh.count;
    }

    // 循环更新持锁状态，直到成功
    for (;;) {
        int c = getState();
        int nextc = c - SHARED_UNIT;
          // cas持锁状态，释放读锁不会影响到其他的线程持有的读锁
          // 若存在持有写锁的线程，则需要等待写锁处理结束才能获取到读锁
          // 两者是排他的
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}

// 4.java.util.concurrent.locks.AbstractQueuedSynchronizer#doReleaseShared
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {//若为SIGNAL状态，则需要被唤醒，若唤醒成功，则继续唤醒后续节点
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            // 若ws为0,则说明线程是可运行状态，则设置状态为PROPAGATE，这个状态就是实现共享锁持有的一个状态
            // 若失败则继续自旋
            else if (ws == 0 && !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        // 若head没改变，则退出自旋
        if (h == head)                   // loop if head changed
            break;
    }
}
```

*写锁内部实现原理与ReentrantLock一样，这里不再阐述，另外ReentrantReadWriteLock也是支持公平锁与非公平锁的，*

## 6. 高级货（StampedLock）
StampedLock也是出自大师Doug Lea之手，它没有遵循AQS实现，而是自己内部实现的一套机制，ReentrantReadWriteLock中读写锁都是一种悲观锁，而后者引入了乐观锁的实现，他通过一个stamp（版本号）来实现。乐观锁是指认为没有其他相关线程在修改数据，而是事先去读取数据，然后检查版本号，若版本号发生变化则读取失败，通常用于读多写少的应用场景中。

如下是源码中示例：

> 通过如下示例，我们可以看出StampedLock支持锁的升级，乐观锁配合悲观锁使用性能会得到很大的提升，较ReentrantReadWriteLock用于读多写少的场景，性能会更好一些。

```java
class Point {
   private double x, y;
   private final StampedLock sl = new StampedLock();

   void move(double deltaX, double deltaY) { // an exclusively locked method
     long stamp = sl.writeLock();//版本号
     try {
       x += deltaX;
       y += deltaY;
     } finally {
       sl.unlockWrite(stamp);
     }
   }

   double distanceFromOrigin() { // A read-only method
     long stamp = sl.tryOptimisticRead();
     double currentX = x, currentY = y;//先读取到数据
     if (!sl.validate(stamp)) {//检测版本号是否发生变化，若发生变化就转换为悲观锁
        stamp = sl.readLock();//悲观锁
        try {
          currentX = x;
          currentY = y;
        } finally {
           sl.unlockRead(stamp);
        }
     }
     return Math.sqrt(currentX * currentX + currentY * currentY);
   }

   // 锁升级（读-->写）
   void moveIfAtOrigin(double newX, double newY) { // upgrade
     // Could instead start with optimistic, not read mode
     long stamp = sl.readLock();//悲观读锁
     try {
       while (x == 0.0 && y == 0.0) {
         long ws = sl.tryConvertToWriteLock(stamp);//由读锁转换为写锁
         if (ws != 0L) {
           stamp = ws;
           x = newX;
           y = newY;
           break;
         }
         else {
           sl.unlockRead(stamp);//转换写锁失败
           stamp = sl.writeLock();//获取悲观写锁
         }
       }
     } finally {
       sl.unlock(stamp);//释放获得锁
     }
   }
}
```

![内部组成结构](https://gitee.com/uRick/oss/raw/master/blog/StampedLock内部组成结构.png)

StampedLock并没有实现AQS，而是内部定义WNode封装线程信息，通过WriteLockView与ReadLockView提供读写能力，ReadWriteLockView实现ReadWriteLock接口，用于提供规范的读写锁对象。

通过对*ReentrantReadWriteLock*与*ReentrantLock*解析，再理解StampedLock并不难，这里暂不对StampedLock进行详细的阐述，可参考
[https://www.cnblogs.com/tong-yuan/p/StampedLock.html](https://www.cnblogs.com/tong-yuan/p/StampedLock.html)

## 7. 工具类
### 7.1. LockSupport
>该工具主要提供实现对线程的阻塞和唤醒操作，在AQS中的Condition内部实现就使用该工具类，它提供两类方法： 

   - **park()** 阻塞当前线程，只有调用unpark才会从当前park唤醒
   - **park(Object blocker) /parkNanos(Object blocker, long nanos)** 阻塞当前线程，时间不能超过nanos,超时就直接返回，blocker是用来标识当前线程在等待的对象
   - **parkUntil(Object blocker, long deadline)** 阻塞当前线程，直到1970年到deadline时间的毫秒数，blocker是用来标识当前线程在等待的对象
   - **parkNanos(long nanos)** 阻塞当前线程，时间不能超过nanos,超时就直接返回
   - **parkUntil(long deadline)** 阻塞当前线程，直到1970年到deadline时间的毫秒数
   - **unpark(Thread thread)** 唤醒阻塞的线程
   - **getBlocker(Thread t)**  获取阻塞对象blocker,若阻塞，则返回阻塞对象，若没有则返回Null,线程可能已经被唤醒或者中断了，这里返回的只是一个快照

### 7.2. CyclicBarrier
顾名思义，可重复（ Cyclic）使用的栅栏（Barrier），其实就是控制多个获取资源的线程达到一个屏障点，再释放全部到达屏障点的线程去执行任务；当前屏障点是预先设置好的，若不满足，屏障这道门是不会被打开的；

**使用示例**

```java
public class CyclicBarrierCase {
    private static final CyclicBarrier BARRIER = new CyclicBarrier(5);
    private static ExecutorService executor = Executors.newFixedThreadPool(20);

    public static void main(String[] args) {
        for (int i = 0; i < 20; i++) {
            executor.submit(new Wc(BARRIER, "" + i));
        }
        executor.shutdown();
    }

    static class Wc implements Runnable {
        private CyclicBarrier barrier;
        private String no;//坑编号

        public Wc(CyclicBarrier barrier, String no) {
            this.barrier = barrier;
            this.no = no;
        }

        @Override
        public void run() {
            try {
                barrier.await();//达到屏障点，大家去抢吧
                System.err.println(this.no + "-" + "获得机会，当前正在使用中");
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.err.println(this.no + "-" + "使用结束了");
        }
    }
}
```

**核心方法`dowait(boolean timed, long nanos)`**

```java
private int dowait(boolean timed, long nanos)
    throws InterruptedException, BrokenBarrierException, TimeoutException {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        final Generation g = generation;//标识broken信息
        if (g.broken) throw new BrokenBarrierException();//是否被破坏
        if (Thread.interrupted()) {//若中断，则唤醒所有等待线程,并抛出异常
            breakBarrier();
            throw new InterruptedException();
        }

        int index = --count;//每调用一次dowait，计数器count减一
        if (index == 0) {  // 达到屏障点，释放任务
            boolean ranAction = false;
            try {
                final Runnable command = barrierCommand;
                if (command != null)
                    command.run();//执行任务
                ranAction = true;
                nextGeneration();//重新生成broken信息，唤醒所有等待中的线程
                return 0;
            } finally {
                if (!ranAction) breakBarrier();//执行失败
            }
        }
        //循环处理，直到线程tripped, broken, interrupted,  timed out
        for (;;) {
            try {
                if (!timed) trip.await();
                else if (nanos > 0L) nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                if (g == generation && ! g.broken) {
                    breakBarrier();
                    throw ie;
                } else {
                    Thread.currentThread().interrupt();
                }
            }
            if (g.broken) throw new BrokenBarrierException();
            if (g != generation) return index;
            if (timed && nanos <= 0L) {
                breakBarrier(); throw new TimeoutException();
            }
        }
    } finally {
        lock.unlock();
    }
}
```

### 7.3. CountDownLatch
 用于一个或者多个线程等待其他线程完成，内部也是实现AQS队列来操作完成，构造函数接收一个int的参数，也就是线程的状态state，需要等待n个操作完成就参数n；当每个操作结束时调用countDown方法完成N从大到小递减，当n递减到0时，当线程被唤醒。

```java
// count 必须大于0 等0时，调用await方法不会阻塞
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```

**使用示例**

```java
public class CountDownLatchCase {
   private static CountDownLatch latch = new CountDownLatch(2);
    public static void main(String[] args) throws InterruptedException {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(1);
                // 调用countDown 则会递减N值，
                latch.countDown();
                System.out.println(2);
                latch.countDown();
            }
        }).start();
        latch.await();
        System.out.println(3);
    }
}
```

### 7.4. Semaphore
Semaphore是一款限流工具，基于AQS同步队列共享锁实现访问资源控制，所有获取线程的资源共享一个锁池，这个锁池中的锁数量是有限的，当获取到锁凭证的线程持锁数量达到锁池指定的数量后，其他需要获取锁的线程需要等待锁池中的锁被释放，才能拿到锁，内部结构实现很简单，使用实例:

**使用示例**

```java
public class SemphoreCase {
private static final Semaphore SEMAPHORE = new Semaphore(10);
private static ExecutorService executor = Executors.newFixedThreadPool(20);

public static void main(String[] args) {
    for (int i = 0; i < 20; i++) {
        executor.submit(new Wc(SEMAPHORE, "" + i));
    }
    executor.shutdown();
}

static class Wc implements Runnable {
    private Semaphore semaphore;
    private String no;//如坑编号

    public Wc(Semaphore semaphore, String no) {
        this.semaphore = semaphore;
        this.no = no;
    }

    public void acquireToilet() {
        try {
             semaphore.acquire();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void releaseToilet() {
        semaphore.release();
    }

    @Override
    public void run() {
        this.acquireToilet();
        System.err.println(this.no + "-" + "获得机会，当前正在使用中");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        this.releaseToilet();
        System.err.println(this.no + "-" + "使用结束了");
    }
}
```