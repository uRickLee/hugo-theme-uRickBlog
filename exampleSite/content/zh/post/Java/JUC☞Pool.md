---
title: JUC☞Pool
date: 2020-02-01 23:08:24
image: https://gitee.com/uRick/oss/raw/master/blog/多线程框架体系.png
description: 顾名思义线程池，负责管理线程、分配任务、控制线程数量的容器；好处就是能很好的管理任务资源利用率，通过空间换取时间的一种方式来降低频繁创建线程和销毁带来的开销；提高任务执行响应时间，当存在新的任务时无需创建线程即可复用池子中的线程。
categories:
 - Java
tags:
 - 多线程
 - 线程池
---

> 顾名思义线程池，负责管理线程、分配任务、控制线程数量的容器；好处就是能很好的管理任务资源利用率，通过空间换取时间的一种方式来降低频繁创建线程和销毁带来的开销；提高任务执行响应时间，当存在新的任务时无需创建线程即可复用池子中的线程。

## 1. Java线程池架构
Java线程池内部实现机制是基于Executor框架实现的，内部其实就是采用Queue与Locker实现的，通过Queue管理任务资源，使用Locker实现处理并发；对于Java后端编码人员来说，工作中遇到多任务是必不可少的，所以掌握好线程池是比要的。如图Executor框架体系：
![多线程框架体系](https://gitee.com/uRick/oss/raw/master/blog/多线程框架体系.png)

- **Executor：** 线程池实现顶层接口，  该接口提供了一种将任务提交与如何运行每个任务的机制方法，包括线程使用、调度等细节；
- **ExecutorService：** 它是Executor的拓展接口，主要提供了任务资源关闭、状态检测、异步执行任务的API方法；
- **AbstractExecutorService：** 主要提供**ExecutorService**的默认基础实现；
- **ScheduledExecutorService：** 提供`schedule(...)`各种重载实现，主要用于实现周期任务的调度，如定时器；
- **ThreadPoolExecutor：** 线程池核心实现，包括任务调度、任务排队、回收资源、拒绝策略等等；
- **ScheduledThreadPoolExecutor:** 周期性任务调度、延时任务的核心实现类，该类继承了ThreadPoolExecutor，同时也拥有线程池的特性；

![ScheduledExecutorService方法](https://gitee.com/uRick/oss/raw/master/blog/ScheduledExecutorService方法.png)

- **FutureTask:** 实现`RunnableFuture`接口，而`RunnableFuture`继承了`Future`、`Ruunable`接口，`Future`提供了异步任务计算结果的获取方式以及判断任务是否完成或取消的方法；
- **Callable：** 创建线程的另一种方式，主要用于异步结果计算中，它的方法`call`可以接收返回值；而**RunnableAdapter**是`Runnable`与`Callable`的适配器，它使`Ruunable`具备`Callable`一样的能力；

```java
static final class RunnableAdapter<T> implements Callable<T> {
    final Runnable task;
    final T result;
    RunnableAdapter(Runnable task, T result) {
        this.task = task;
        this.result = result;
    }
    public T call() {
        task.run();
        return result;
    }
}
```
- **Executors：** 一个对线程池创建方式进行封装的工具类，提供了常见线程池创建方法；
    - **newFixedThreadPool：** 该方法返回一个固定数量的线程池，线程数不变，当有一个任务提交时，若线程池中空闲，则立即执行，若没有，则会被暂缓在一个任务队列中，等待有空闲的线程去执行；
    - **newSingleThreadExecutor:** 创建1个线程的线程池，若空闲则执行，若没有空闲线程则暂缓在任务队列中；
    - **newCachedThreadPool：** 返回一个可根据实际情况调整线程个数的线程池，不限制最大线程数量，若用空闲的线程则执行任务，若无任务则不创建线程。并且每一个空闲线程会在60秒后自动回收；
    - **newScheduledThreadPool:** 创建一个可以指定线程的数量的线程池，但是这个线程池还带有延迟和周期性执行任务的功能，类似定时器；
    - **newWorkStealingPool：** 创建ForkJoinPool线程池。

## 2. ThreadPoolExecutor解析

![ThreadPoolExecutor结构图](https://gitee.com/uRick/oss/raw/master/blog/ThreadPoolExecutor结构图.png)

### 2.1. 线程池核心参数
```java
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit,
                          BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler) {
    if (corePoolSize < 0 || maximumPoolSize <= 0 || maximumPoolSize < corePoolSize || keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ? null : AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

如上构造方法参数：

- **corePoolSize：** 核心线程池的大小；
- **maximumPoolSize：** 最大线程池的大小；
- **keepAliveTime：** 超时时间，超出核心线程数量以外的线程空闲存活时间；
- **unit：** 存活时间单位；
- **workQueue：** 用来暂时保存任务的工作队列；
- **threadFactory：** 创建线程的工厂类，`Executors`中有默认工厂类（`DefaultThreadFactory`）实现；
- **handler：** 对无法进行处理的任务，可以自定义实现`RejectedExecutionHandler`，执行对应的拒绝策略。JDK默认实现了4中策略：
![线程拒绝策略](https://gitee.com/uRick/oss/raw/master/blog/线程拒绝策略.png)
    - **CallerRunsPolicy：** 使用调用者线程执行任务；
    - **AbortPolicy：** 直接抛出异常；
    - **DiscardPolicy：** 不做任务处理，丢弃到当前任务；
    - **DiscardOldestPolicy：** 移除队列Head任务，并立即执行当前任务。

线程池中几大工作运行状态：

```java
// 基于int32bit来判断状态
private static final int COUNT_BITS = Integer.SIZE - 3;
// 最大线程容量
private static final int CAPACITY = (1 << COUNT_BITS) - 1;
// 接受新任务，并执行队列中的任务
private static final int RUNNING  = -1 << COUNT_BITS;
// 不接受任务，只会执行队列中的任务
private static final int SHUTDOWN =  0 << COUNT_BITS;
//中断执行中的任务，也不会接受新任务
private static final int STOP  =  1 << COUNT_BITS;
// 准备调用terminated()
private static final int TIDYING  =  2 << COUNT_BITS;
// 该状态表示terminated()执行完了
private static final int TERMINATED =  3 << COUNT_BITS;
```

### 2.2. 线程池任务调度核心流程
![线程池调度流程](https://gitee.com/uRick/oss/raw/master/blog/线程池调度流程.png)
#### 2.2.1. executor方法
`execute(Runnable command)`是执行任务的核心入口，它主要的工作就是创建Worker和添加任务到队列中；

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();
    // 并不是一开始就会创建核心线程Worker的，使用时在创建
    //当工作线程数，小于核心线程池数时，则创建核心工作线程
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // 工作线程大于corePoolSize，则添加任务到队列中
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        // 在看一下，worker是否工作
        if (!isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 队列已满，则创建非核心工作线程
    else if (!addWorker(command, false))
        reject(command);//执行拒绝策略
}
```
#### 2.2.2. addWorker方法
> `addWorker(Runnable firstTask, boolean core)`主要负责创建执行任务的工作线程，根据传入的参数`core`判断创建的是核心线程还是非核心线程；
> 当线程池中线程数小于核心线程时，则创建核心的线程；当线程数大于核心线程且任务队列已满时，再创建非核心线程执行任务；

```java
private boolean addWorker(Runnable firstTask, boolean core) {
// 根据队列、工作线程容量、以及运行状态判断是否需要创建Worker
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);
        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN && ! (rs == SHUTDOWN && firstTask == null && ! workQueue.isEmpty()))
            return false;
        for (;;) {
            int wc = workerCountOf(c);
           //如果工作线程数大于默认容量大小或者大于核心线程数大小，则直接return,不允许添加worker
            if (wc >= CAPACITY || wc >= (core ? corePoolSize : maximumPoolSize)) return false;
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

    boolean workerStarted = false;//线程是否被启动
    boolean workerAdded = false;//worker是否添加成功
    Worker w = null;
    try {
        w = new Worker(firstTask);// 创建worker
        final Thread t = w.thread;// 基于线程工厂创建的线程，这个线程其实就worker,因为worker实现Runnable接口
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                int rs = runStateOf(ctl.get());
                if (rs < SHUTDOWN || (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // 重新检查一下，创建worker是否是活着的
                        throw new IllegalThreadStateException();
                    workers.add(w);// 将worker添加HashSet<Worker>中
                    int s = workers.size();
                    //工作线程数大于以前出现过的最大线程数，则设置新的线程池数
                    //这样做是为了便于监控
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {// worker添加成功了，可以愉快的玩耍了
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        // 若worker启动失败，则移除worker,并做中断处理，具体实现可查看addWorkerFailed(W实现)
        if (!workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

若worker启动失败，会去继续如下处理：

```java
private void addWorkerFailed(Worker w) {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();//获取的重入锁
        try {
            if (w != null)
                workers.remove(w);// 将work从HashSet<Worker>中移除
            decrementWorkerCount();// cas方式减少核心线程数
            tryTerminate();// 检测中断，防止worker未被中断情况
        } finally {
            mainLock.unlock();
        }
    }
```

```java
final void tryTerminate() {
    for (;;) {
        int c = ctl.get();
        // 运行中、TIDYING/SHUTDOWN下并且队列不为空，不做中断处理
        if (isRunning(c) || runStateAtLeast(c, TIDYING) || (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
            return;
        if (workerCountOf(c) != 0) { // Eligible to terminate
            interruptIdleWorkers(ONLY_ONE);// 中断空闲Wroker,也就没有可执行的task的Worker
            return;
        }
        // 执行中断，从TIDYING到TERMINATED的过程
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                try {
                    terminated();
                } finally {
                    ctl.set(ctlOf(TERMINATED, 0));
                    termination.signalAll();
                }
                return;
            }
        } finally {
            mainLock.unlock();
        }
        // else retry on failed CAS
    }
}
```

#### 2.2.3. runWorker方法
> 如上，既然`Worker`实现了`Runnable`，那么当调用start时，肯定定会执行重写`run`的内部代码，也就是`runWorker`,接下来继续看内部实现机制。其中Worker还是实现AQS同步队列，它自定义独占锁的实现，采用独占锁目的是为了防止意外被中断的情况，当前Worker正在工作，不允许被打扰；那么它为什么不是可重入的呢？其中几个主要作用：

1. 如果正在执行任务，则不应该中断线程；
2. 如果该线程现在不是独占锁的状态，也就是空闲的状态，说明它没有在处理任务，这时可以对该线程进行中断；
3. 线程池在执行`shutdown`方法或`tryTerminate`方法时会调用`interruptIdleWorkers`方法来中断空闲的线程，`interruptIdleWorkers`方法会使用`tryLock`方法来判断线程池中的线程是否是空闲状态；
4. 之所以设置为不可重入，是因为我们不希望任务在调用像`setCorePoolSize`这样的线程池控制方法时重新获取锁，这样会中断正在运行的线。

```java
// 设置线程的核心线程数,并覆盖构造函数中设置的核心线程数。
// 如果新值小于当前值，则多余的现有线程将在下一次空闲时终止。
// 如果需要，新线程将开始执行队列任务。
public void setCorePoolSize(int corePoolSize) {
    if (corePoolSize < 0)
        throw new IllegalArgumentException();
    int delta = corePoolSize - this.corePoolSize;
    this.corePoolSize = corePoolSize;
    if (workerCountOf(ctl.get()) > corePoolSize)
        interruptIdleWorkers();
    else if (delta > 0) {//若新设置的核心线程数大于构造函数中设置的核心线程数，判断是否需要创建核心Worker
        int k = Math.min(delta, workQueue.size());
        while (k-- > 0 && addWorker(null, true)) {
            if (workQueue.isEmpty())//队里为空，则终止
                break;
        }
    }
}
```

runWorker内部执行任务逻辑

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;//获取worker对应的任务
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) {
            w.lock();//加锁，防止SHUTDOWN时被中断，如下，当ctl.get()大于STOP就会被中断了
           //如果池停止，则确保线程被中断；
           //如果没有，则确保线程不中断；
           //这需要在第二种情况下进行重新检查，以便在清除中断的同时处理关闭现在的竞争。
            if ((runStateAtLeast(ctl.get(), STOP) || (Thread.interrupted() && runStateAtLeast(ctl.get(), STOP))) && !wt.isInterrupted()) 
            wt.interrupt();
            try {
                beforeExecute(wt, task);//任务执行前做一些事情，默认没有实现，这里可以自己定义
                Throwable thrown = null;
                try {
                    // 执行任务，为什么提交的任务只执行run呢，
                    // 其实就是为了统一管理资源，若调用start()则通过JVM调度，就无法控制了
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    // 最后处理一些收尾工作，同beforeExecute一个道理
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;//完成任务数
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        // 处理worker退出，销毁资源，completedAbruptly标识是否被中断
        processWorkerExit(w, completedAbruptly);
    }
}
```

worker循环获取任务执行结束后，调用`processWorkerExit`来销毁worker

```java
private void processWorkerExit(Worker w, boolean completedAbruptly) {
    if (completedAbruptly) // If abrupt, then workerCount wasn't adjusted
        decrementWorkerCount();

    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        completedTaskCount += w.completedTasks;// 是指当前任务完成数
        workers.remove(w);// 移除worker
    } finally {
        mainLock.unlock();
    }

    tryTerminate();//中断处理

    int c = ctl.get();
    if (runStateLessThan(c, STOP)) {// 判断线程池是否停止
        if (!completedAbruptly) {// worker是否执行结束
            int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
            if (min == 0 && ! workQueue.isEmpty())
                min = 1;
            if (workerCountOf(c) >= min)// 是否会执行后续工作addWorker
                return; // replacement not needed
        }
        addWorker(null, false);
    }
}
```

#### 2.2.4. reject方法
> 当线程池中的线程已经用完了，无法继续为新任务服务；当任务队列满了，装不下任务了，亦或者系统压力太大，无法继续承载工作了，那么`reject`就起作用了。默认情况JDK已经实现了4中机制：

- **AbortPolicy：** 抛出异常`RejectedExecutionException`；
- **CallerRunsPolicy：** 当前线程池为关闭，则直接运行调用者线程（任务）；
- **DiscardOledestPolicy：** 获取队列头中的任务，并提交到到线程池中执行，其实就把资历最老的任务抛弃，然后执行reject的任务；
- **DiscardPolicy：** 不做任务何处，直接抛弃，就是这么横🤣，这种方式太暴力，不太优雅！

*其实除了上面的4中策略，我们还可以自定义实现`RejectedExecutionHandler`，自定义处理策略，然后构造线程池时，传入这个`reject`即可*
```java
public interface RejectedExecutionHandler {
 void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}
```

## 3. FutureTask
> 对于`FutureTask`在本文开头部分，Java线程池架构图中也有体现，它是Java中实现异步计算的基础；其中`ScheduledThreadPoolExecutor`就是基于`FutureTask`与`ThreadPoolExecutor`来实现的。

![内部结构](https://gitee.com/uRick/oss/raw/master/blog/FutureTask内部结构.png)

### 3.1. FutureTask使用实例

```java
public class Tester {
    public static void main(String[] args) {
        MultiTask multiTask1 = new MultiTask(10, 20, 2000L);//任务1
        MultiTask multiTask2 = new MultiTask(20, 40, 4000L);//任务2
        FutureTask<Integer> taskOne = new FutureTask<>(multiTask1);
        FutureTask<Integer> taskTwo = new FutureTask<>(multiTask2);
        ExecutorService executor = Executors.newFixedThreadPool(2);
        executor.execute(taskOne);//执行任务
        executor.execute(taskTwo);
        while (true) {
            try {
                if (!taskOne.isDone()) {//判断任务是否新建状态
                    //获取任务执行结果，一直阻塞直到获取到结果（没有发生中断或取消的情况下）
                    System.out.println("FutureTask1 output=" + taskOne.get());
                }
                if (!taskTwo.isDone()) {
                    System.out.println("Waitng for futureTask2 for completion");
                    System.out.println("FutureTask2 output=" + taskTwo.get());
                }
                if (taskOne.isDone() && taskTwo.isDone()) {
                    System.out.println("Completed both the FutureTasks, shutting down the executors");
                    executor.shutdown();
                    return;
                }
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
        }
    }
    /**
     * 定义实现Callable接口，乘法计算Task
     */
   static class MultiTask implements Callable<Integer> {
        int a;
        int b;
        long sleepTime;
        public MultiTask(int a, int b, long sleepTime) {
            this.a = a;
            this.b = b;
            this.sleepTime = sleepTime;
        }
        @Override
        public Integer call() throws Exception {
            Thread.sleep(sleepTime);
            return a * b;
        }
    }
}
```

### 3.2. FutureTask的几大状态

```java
private static final int NEW          = 0;//创建的新任务，还没开始
private static final int COMPLETING   = 1;//表示任务计算已经完成，但是还有一些后续工作需要处理
private static final int NORMAL       = 2;//表示任务正常结束,执行特别顺畅，没有发生异常或者中断
private static final int EXCEPTIONAL  = 3;//表示任务发生异常
private static final int CANCELLED    = 4;//表示任务已经被取消
private static final int INTERRUPTING = 5;//表示任务中断中，可以理解为正在做中断处理
private static final int INTERRUPTED  = 6;//表示任务已经被中断
```
通常情况下，几种之间存在以下关系：

 * NEW -> COMPLETING -> NORMAL *过五关斩六将，任务执行很顺畅*
 * NEW -> COMPLETING -> EXCEPTIONAL *任务执行过程发生异常*
 * NEW -> CANCELLED *任务被取消*
 * NEW -> INTERRUPTING -> INTERRUPTED *任务被中断*

如下源码，通过它的构造方法，可以看出FutureTask支持Callable与Runnable方式实现的任务，若是实现Runnable的任务，则需要通过`Executors.callable(runnable, result)`方法包装一下，其实内部就是通过RunnableAdapter适配器做一下适配转换为Callable形式，默认新创建的Task状态都为New;

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}

public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}
```

**RunnableAdapter适配器**

```java
static final class RunnableAdapter<T> implements Callable<T> {
    final Runnable task;
    final T result;
    RunnableAdapter(Runnable task, T result) {
        this.task = task;
        this.result = result;
    }
    public T call() {
        task.run();
        return result;
    }
}
```

### 3.3. 内部原理解析

#### 3.3.1. get方法
> get方法主要检查任务状态，为后续获取结果预处理，最后通过`report(s)`方法返回结果；

```java
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);//未发生中断和取消情况，阻塞等待获取结果，不做超时处理
    return report(s);//响应结果，若发生中断和取消则抛出指定的异常信息
}

public V get(long timeout, TimeUnit unit)
    throws InterruptedException, ExecutionException, TimeoutException {
    if (unit == null)
        throw new NullPointerException();
    int s = state;
    if (s <= COMPLETING &&
        (s = awaitDone(true, unit.toNanos(timeout))) <= COMPLETING)
        throw new TimeoutException();
    return report(s);
}

// 适配任务结果，正常结束或抛出异常信息
private V report(int s) throws ExecutionException {
    Object x = outcome;
    if (s == NORMAL)
        return (V)x;
    if (s >= CANCELLED)//任务取消，抛CancellationException异常
        throw new CancellationException();
    throw new ExecutionException((Throwable)x);
}
```

#### 3.3.2. awaitDone方法
> `int awaitDone(boolean timed, long nanos)`方法是等待任务计算处理结果，超时响应、中断、取消的核心处理方法，get方法也调用它处理的。

```java
/**
 * @param timed true if use timed waits
 * @param nanos time to wait, if timed
 * @return state upon completion
 */
private int awaitDone(boolean timed, long nanos)
    throws InterruptedException {
    final long deadline = timed ? System.nanoTime() + nanos : 0L;//计算超时时间
    WaitNode q = null;
    boolean queued = false;//入栈标识
    for (;;) {
        if (Thread.interrupted()) {//发生中断，则移节点q
            removeWaiter(q);
            throw new InterruptedException();
        }

        int s = state;
        if (s > COMPLETING) {//若任务执行完毕，设置了最终状态或者被取消，则返回
            if (q != null)
                q.thread = null;
            return s;
        }
        else if (s == COMPLETING) // 让出资源，并再次参与竞争资源，其实这里就是为能够更快的获得到执行最终结果状态
            Thread.yield();
        else if (q == null)
            q = new WaitNode();//一个简单的链表，记录Treiber栈等待中的线程
        else if (!queued)//CAS入栈操作
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset, q.next = waiters, q);
        else if (timed) {//若设置了超时处理，则进入超时处理逻辑
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {//已经超时，则移除节点，返回状态
                removeWaiter(q);
                return state;
            }
            LockSupport.parkNanos(this, nanos);
        }
        else
            LockSupport.park(this);
    }
}

private void removeWaiter(WaitNode node) {
    if (node != null) {
        node.thread = null;//设置node为null,作为下面判断的标识
        retry://goto
        for (;;) {
            for (WaitNode pred = null, q = waiters, s; q != null; q = s) {
                s = q.next;
                if (q.thread != null)// q.thread != null说明该q节点不需要移除
                    pred = q;
                else if (pred != null) {//q.thread为null，pred != null则移除
                    pred.next = s;
                    if (pred.thread == null) // pred.thread为null，在其他地方修改了，则需要重试
                        continue retry;
                }
                // 若上述条件不满足，则
                // 设置WaitNode头节点为S，设置失败发起重试
                else if (!UNSAFE.compareAndSwapObject(this, waitersOffset, q, s))
                    continue retry;
            }
            break;
        }
    }
}
```

#### 3.3.3. run方法

> FutureTask实现Runnable接口，run方法是任务执行核心方法，通过线程池调度任务执行，并通过Callable处理异步结果响应；

```java
public void run() {
    /*
     * 首先判断状态，若不是New，则任务可能被执行或取消；
     * runner是FutureTask的一个属性（通过Unsafe获取），用于保存执行任务的线程，
     * 如果不为空则表示已经有线程正在执行，这里用CAS来设置，失败则返回。
     */
    if (state != NEW || !UNSAFE.compareAndSwapObject(this, runnerOffset, null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;//需要执行的任务
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                result = c.call();//执行任务
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                //任务执行发生异常，设置任务为中断状态，并唤醒WaitNode
                setException(ex);
            }
            if (ran)
                set(result);//任务正常执行结束，设置调度结果
        }
    } finally {
        runner = null;
        int s = state;//重新读取状态，避免泄漏
        if (s >= INTERRUPTING)//若任务发生中断，则做中断处理
            handlePossibleCancellationInterrupt(s);//若任务状态为INTERRUPTING，保证任务状态为INTERRUPTED，彻底处理掉
    }
}

protected void setException(Throwable t) {
    //设置state为COMPLETING，再设置EXCEPTIONAL
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        outcome = t;//将异常对象传递给outcome，一遍后续get()获取结果的时候抛出对应的异常信息
        UNSAFE.putOrderedInt(this, stateOffset, EXCEPTIONAL); // final state
        finishCompletion();
    }
}
```

#### 3.3.4. runAndReset方法
> runAndReset方法与run方法本质上没多大区别，区别在于runAndReset是为执行重复性任务而设计的，任务执行结束后不调用set方法设置任务状态state，而是直接根据任务状态state判断返回当前任务是否可以重复下一次执行，该方法的使用可以查看`ScheduledThreadPoolExecutor`线程池中的使用，后续会讲解。

```java
prote  cted boolean runAndReset() {
    /*
     * 首先判断状态，若不是New，则任务可能被执行或取消；
     * runner是FutureTask的一个属性（通过Unsafe获取），用于保存执行任务的线程，
     * 如果不为空则表示已经有线程正在执行，这里用CAS来设置，失败则返回。
     */
    if (state != NEW || !UNSAFE.compareAndSwapObject(this, runnerOffset, null, Thread.currentThread()))
        return false;
    boolean ran = false;
    int s = state;
    try {
        Callable<V> c = callable;
        if (c != null && s == NEW) {
            try {
                c.call(); // don't set result
                ran = true;
            } catch (Throwable ex) {
                //任务执行发生异常，设置任务为中断状态，并唤醒WaitNode
                setException(ex);
            }
        }
    } finally {
        runner = null;
        s = state;//重新读取状态，避免泄漏
        if (s >= INTERRUPTING))//若任务发生中断，则做中断处理
            handlePossibleCancellationInterrupt(s);
    }
    //任务执行成功，且state为NEW返回true
    //防止任务被取消或中断（若中断或取消，不在执行后续重复性任务）
    return ran && s == NEW;
}
```

#### 3.3.5. finishCompletion方法
>  任务执行完成或发生异常中断或者取消后，调用finishCompletion清理资源，移除当前任务，唤醒所有等待中的线程；

```java
private void finishCompletion() {
// assert state > COMPLETING;
for (WaitNode q; (q = waiters) != null;) {//轮询waiters并唤醒
    // 置空WiterNode栈顶
    if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
        for (;;) {
            Thread t = q.thread;
            if (t != null) {
                q.thread = null;
                LockSupport.unpark(t);//唤醒线程
            }
            WaitNode next = q.next;
            if (next == null)//已经没有waiter了直接跳出
                break;
            q.next = null; // unlink to help gc
            q = next;
        }
        break;
    }
}

done();//模板方法，子类实现，任务执行结束，可自定义处理相关资源

callable = null;
}
```

*📌拓展：*  在上文中提到Treiber栈，这是一种Treiber算法[^1]实现的，是一个可扩展的无锁栈，利用细粒度的并发原语CAS来实现的[^2]。

## 4. ScheduledThreadPoolExecutor解析

![ScheduledThreadPoolExecutor UML结构图](https://gitee.com/uRick/oss/raw/master/blog/ScheduledThreadPoolExecutorUML结构图.png)

ScheduledThreadPoolExecutor是一个基于线程池ThreadPoolExecutor实现的一套执行周期性任务的框架，它继承了ThreadPoolExecutor类，同时实现ScheduledExecutorService接口，ScheduledExecutorService接口定义任务调度的策略方法，也是开发过程中常用的API方法。


*ScheduledExecutorService方法：*

> 注意这里的`scheduleAtFixedRate`与`scheduleWithFixedDelay`的区别，前者是指周期性任务，不会因为前一次的任务执行而影响下一个任务，而是每次都按照计划周期（period）执行任务；后者执行任务时，前一个任务执行会响应到后者的执行时间，其实就delay延迟时间就是在前一个任务执行完成后，再延迟（delay）时间后执行下一个任务。

```java
public interface ScheduledExecutorService extends ExecutorService {
    //执行给定延迟时间delay的任务
    public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit);
    //执行给定延迟时间delay的任务，同schedule一样，只不过时基于Callable接口实现
    public <V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit);
    //执行给定延迟时间initialDelay执行任务（首次执行），并按照周期（period）执行后续任务
    public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit);
    //执行给定延迟时间initialDelay执行任务（首次执行），后续任务在每次任务执行结束与下一次开始执行之间延迟（delay）
    public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit);
}
```

### 4.1. ScheduledThreadPoolExecutor使用示例

```java
public class BeeperControl {
    private final ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
    private final String DATE_FORMAT = "yyyy-MM-dd HH:mm:ss:SSS";

    public void beepForAnHour() throws ExecutionException, InterruptedException {
        scheduler.schedule(new Runnable() {
            @Override
            public void run() {
                SimpleDateFormat sdf = new SimpleDateFormat(DATE_FORMAT);
                String dateStr = sdf.format(new Date());
                System.out.println(dateStr + "-->10s一次性schedule");
            }
        }, 10, TimeUnit.SECONDS);
        // Callable Task
        FutureTask<String> futureTask = new FutureTask<>(new Callable<String>() {
            @Override
            public String call() throws Exception {
                SimpleDateFormat sdf = new SimpleDateFormat(DATE_FORMAT);
                String dateStr = sdf.format(new Date());
                return dateStr + "-->10s一次性Callable";
            }
        });

        scheduler.schedule(futureTask, 10, TimeUnit.SECONDS);

        if (!futureTask.isDone()) {//获取结果
            System.out.println(futureTask.get()+"FutureTask Result");
        }

        scheduler.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                SimpleDateFormat sdf = new SimpleDateFormat(DATE_FORMAT);
                String dateStr = sdf.format(new Date());
                System.out.println(dateStr + "-scheduleAtFixedRate-->周期性任务->30*60");
                try {
                    TimeUnit.SECONDS.sleep(60);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }
        }, 30, 60, TimeUnit.SECONDS);
        scheduler.scheduleWithFixedDelay(new Runnable() {
            @Override
            public void run() {
                SimpleDateFormat sdf = new SimpleDateFormat(DATE_FORMAT);
                String dateStr = sdf.format(new Date());
                System.out.println(dateStr + "-scheduleWithFixedDelay-->周期性任务->30*60");
                try {
                    TimeUnit.SECONDS.sleep(60);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, 30, 60, TimeUnit.SECONDS);
    }
}
```

### 4.2. 内部原理解析

#### 4.2.1. 构造方法
> 它使用内部类DelayedWorkQueue作为任务调度存储队列，任务延时周期性执行就是通过延时队列实现的，它的maximumPoolSize为`Integer.MAX_VALUE`，keepAliveTime为0，通过调用父类构造方法创建ScheduledThreadPoolExecutor实例。

```java
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS, new DelayedWorkQueue());
}
public ScheduledThreadPoolExecutor(int corePoolSize, ThreadFactory threadFactory) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS, new DelayedWorkQueue(), threadFactory);
}

public ScheduledThreadPoolExecutor(int corePoolSize, RejectedExecutionHandler handler) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS, new DelayedWorkQueue(), handler);
}

public ScheduledThreadPoolExecutor(int corePoolSize, ThreadFactory threadFactory, RejectedExecutionHandler handler) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS, new DelayedWorkQueue(), threadFactory, handler);
}
```

#### 4.2.2. 任务调度方法
> ScheduledThreadPoolExecutor有四个调度方法，就是上文提到ScheduledExecutorService接口定义的，其实四个方法内部实现逻辑大同小异，下面以`scheduleAtFixedRate`方法为例探究内部实现逻辑。

```java
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit) {
    if (command == null || unit == null)// 任务与时间单位不能为空
        throw new NullPointerException();
    if (period <= 0)//若period小于等于零，参数非法
        throw new IllegalArgumentException();
    // 创建ScheduledFutureTask异步任务，若任务是Void类型的，triggerTime(initialDelay, unit)方法用于计算任务触发时间
    ScheduledFutureTask<Void> sft = new ScheduledFutureTask<Void>(command, null, triggerTime(initialDelay, unit), unit.toNanos(period)); 
    // decorateTask是一个任务装潢的模板方法，提供给子类实现，这里直接return的command
    RunnableScheduledFuture<Void> t = decorateTask(command, sft);
    sft.outerTask = t;
    // 延时执行
    delayedExecute(t);
    return t;
}
```

```java
private void delayedExecute(RunnableScheduledFuture<?> task) {
    if (isShutdown())//检查线程池是否关闭，若关闭则执行线程池拒绝策略
        reject(task);
    else {
        super.getQueue().add(task);// 任务入队
        // 在判断线程池是否关闭，若关闭则移除task
        if (isShutdown() && !canRunInCurrentRunState(task.isPeriodic()) && remove(task))
            task.cancel(false);//取消执行任务，移除队列
        else//比较当前工作线程数与线程池定义时的配置，添加对应的worker，确保task任务能能够被执行
            ensurePrestart();
    }
}
```

#### 4.2.3. 内部类RunnableScheduledFuture
从源码中可知，线程池中几个调度任务方法中，都使用内部类ScheduledFutureTask对提交的任务封装了一层，定义内部通用属性，它继承了FutureTask类和实现RunnableScheduledFuture接口；同时也定义了任务在延时队列DelayedWorkQueue中的排序逻辑和任务执行逻辑，下面看一下它的内部结构：

```java
private class ScheduledFutureTask<V> extends FutureTask<V> implements RunnableScheduledFuture<V> {
    private final long sequenceNumber;//序列号，保证FIFO队列原则
    private long time;//任务触发时间
    //任务执行周期，period等于零,表示非周期性任务，period大于零表示计划型（fixed-rate）周期任务，peroid小于零表示延迟型（fixed-delay）周期任务
    private final long period;
    //  将重新执行的目标任务重新排队
    RunnableScheduledFuture<V> outerTask = this;
    int heapIndex;//延时队列索引

    ScheduledFutureTask(Runnable r, V result, long ns) {
        super(r, result);
        this.time = ns;
        this.period = 0;
        this.sequenceNumber = sequencer.getAndIncrement();
    }

    ScheduledFutureTask(Runnable r, V result, long ns, long period) {
        super(r, result);
        this.time = ns;
        this.period = period;
        this.sequenceNumber = sequencer.getAndIncrement();
    }

    ScheduledFutureTask(Callable<V> callable, long ns) {
        super(callable);
        this.time = ns;
        this.period = 0;
        this.sequenceNumber = sequencer.getAndIncrement();
    }

    //计算延时时间
    public long getDelay(TimeUnit unit) {
        return unit.convert(time - now(), NANOSECONDS);
    }

    //任务在延时队列中的排序规则
    public int compareTo(Delayed other) {
        if (other == this) //同一个任务
            return 0;
        if (other instanceof ScheduledFutureTask) {//ScheduledFutureTask任务
            ScheduledFutureTask<?> x = (ScheduledFutureTask<?>)other;
            long diff = time - x.time;
            if (diff < 0)//执行时间小的排在前面
                return -1;
            else if (diff > 0)
                return 1;
            else if (sequenceNumber < x.sequenceNumber)//当上述条件不满足，执行时间相等时，根据队列序列号排序，序列号小的排前面
                return -1;
            else
                return 1;
        }
        //非ScheduledFutureTask任务，排序逻辑同上一样
        long diff = getDelay(NANOSECONDS) - other.getDelay(NANOSECONDS);
        return (diff < 0) ? -1 : (diff > 0) ? 1 : 0;
    }

    //是否是周期性任务
    public boolean isPeriodic() { return period != 0; }

    // 计算下一次执行时间
    private void setNextRunTime() {
        long p = period;
        if (p > 0) time += p;
        else time = triggerTime(-p);
    }

    //取消任务
    public boolean cancel(boolean mayInterruptIfRunning) {
        boolean cancelled = super.cancel(mayInterruptIfRunning);
        if (cancelled && removeOnCancel && heapIndex >= 0) remove(this);
        return cancelled;
    }

    public void run() {
        boolean periodic = isPeriodic();
        if (!canRunInCurrentRunState(periodic))//判断是否需要取消任务
            cancel(false);
        else if (!periodic)//若是一次性任务，则调用父类FutureTask的run方法执行任务
            ScheduledFutureTask.super.run();
        else if (ScheduledFutureTask.super.runAndReset()) {//执行任务并判断是否有资格重复下一次执行
            setNextRunTime();//设置下一次任务执行时间
            reExecutePeriodic(outerTask);//对任务进行重新入队操作，同delayedExecute方法
        }
    }
}
```

#### 4.2.4. 怎么实现任务延时？
> 在文章开头分析ThreadPoolExecutor线程池源码时，得知线程池获取任务是通过调用队列`take()`方法与`poll(long timeout, TimeUnit unit)`方法的，下面来分析一下队列DelayedWorkQueue的`take()`方法是如何获取任务的，怎么达到延时效果的？

```java
public RunnableScheduledFuture<?> take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();//获取锁
    try {
        for (;;) {
            RunnableScheduledFuture<?> first = queue[0];
            if (first == null)//队列header任务为null，表示队列中没有数据，则放入condition队列等待被唤醒.
                available.await();
            else {
                long delay = first.getDelay(NANOSECONDS);//根据task任务计算delay时间
                if (delay <= 0)//任务已超时，则task出队
                    return finishPoll(first);
                first = null; // don't retain ref while waiting
                if (leader != null)//说明在执行else逻辑，线程thisThread还在阻塞中
                    available.await();
                else {
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        available.awaitNanos(delay);//等待delay时间超时
                    } finally {
                        if (leader == thisThread)
                            leader = null;//清空标识
                    }
                }
            }
        }
    } finally {
        //说明没有等待超时处理的线程，且队列中有任务，则唤醒condition队列里的任务并释放锁
        if (leader == null && queue[0] != null)
            available.signal();
        lock.unlock();
    }
}
```

## 5. 写在最后
### 5.1. 如何合理配置线程池的大小
> 对于线程池大小设置，一致是一个困惑的问题，怎么去设置呢？有没有一个计算方式呢？工作中总感觉是在揣测，下面给一个计算方式：

- 需要分析线程池执行的任务的特性是`CPU密集型`还是`IO密集型`；
- 每个任务执行的平均时长大概是多少，这个任务的执行时长可能还跟任务处理逻辑是否涉，以及网络传输和底层系统资源依赖有关系；
- 若是`CPU密集型`， 主要是执行计算任务，响应时间很快， CPU一直在运行，这种任务CPU的利用率很高，那么线程数的配置应该根据CPU核心数来决定，CPU核心数=最大同时执行线程数，假设CPU 核心数为4，那么服务器最多能同时执行4个线程；过多的线程会导致上下文切换反而使得效率降低。那线程池的最大线程数可以配置为CPU核心数+1；
- 若是`IO密集型`， 主要是进行IO操作，执行IO操作的时间较长，这时CPU出于空闲状态，导致CPU的利用率不高，这种情况下可以增加线程池的大小；可以结合线程的等待时长来做判断，等待时间越高，那么线程数也相对越多，一般可以配置CPU核心数的2倍。

> 线程池设定最佳线程数目= （（线程池设定的线程等待时间+线程CPU时间）/ 线程CPU时间 ）* CPU数目
> CPU时间为单个线程执行时间

### 5.2. 线程池中的线程初始化
> 默认情况下，创建线程池之后线程池中是没有线程的，需要提交任务之后才会创建线程。在实际中如果需要线程池创建之后立即创建线程；

可以通过两个方法实现：

- `prestartCoreThread()`：初始化一个核心线程；
- `prestartAllCoreThreads()`：初始化所有核心线程。

### 5.3. 如何优雅的关闭线程池
ThreadPoolExecutor提供了两个方法 ，用于线程池的关闭， 分别是`shutdown()`和`shutdownNow()`，其中：
- `shutdown()`：不会立即终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务；
- `shutdownNow()`：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务。

### 5.4. 线程池容量的动态调整

ThreadPoolExecutor提供了动态调整线程池容量大小的方法：`setCorePoolSize()`和`setMaximumPoolSize()`，其中：

- `setCorePoolSize()`：设置核心池大小
- `setMaximumPoolSiz()`：设置线程池最大能创建的线程数目大小。

### 5.5. 线程池中常见属性含义

|          属性          |                             含义                             |       备注        |
| :--------------------: | :----------------------------------------------------------: | :---------------: |
|      corePoolSize      |                          核心池大小                          |                   |
|    maximumPoolSize     |                       最大线程池的大小                       |                   |
|     keepAliveTime      |      超时时间，超出核心线程数量以外的线程空闲存活时间；      |                   |
| allowCoreThreadTimeOut |                  是否允许线程池核心线程超时                  | `keepAliveTime>0` |
|    **生命周期状态**    |                           **描述**                           |     **备注**      |
|        RUNNING         |       线程池可以处理新任务，也可以处理阻塞队列中的人物       |                   |
|        SHUTDOWN        | 线程池已关闭，不再接受新任务，只会处理阻塞队列中的任务，处理结束后将关闭 |   `shutdown()`    |
|          STOP          | 不处理新任务、阻塞队列中的任务，并且会直接暴力中断正在执行中的任务 |  `shutdownNow()`  |
|        TIDYING         |               所有任务都已经终止，线程池为空的               |                   |
|       TERMINATED       |                  执行`terminated()`后的状态                  |                   |



[^1]: [https://en.wikipedia.org/wiki/Treiber_stack#cite_note-4](https://en.wikipedia.org/wiki/Treiber_stack#cite_note-4)
[^2]: [https://segmentfault.com/a/1190000012463330](https://segmentfault.com/a/1190000012463330)