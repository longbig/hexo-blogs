---
title: java8的5种线程池
top: false
cover: false
toc: true
mathjax: true
date: 2021-05-08 23:29:02
password:
summary:
tags:
categories:
---
# JAVA8的5种线程池

[toc]

## 介绍

JAVA1.8版本加了一种newWorkStealingPool，现在共有5种线程池：newSingleThreadPool、newFixedThreadPool、newCachedThreadPool、newScheduledThreadPool、newWorkStealingPool。

参考：https://vvcat.blog.csdn.net/article/details/91409942

**线程池的创建步骤：**

```java
//创建单核心的线程池
ExecutorService executorService = Executors.newSingleThreadExecutor();
//创建固定核心数的线程池，这里核心数 = 2
ExecutorService executorService = Executors.newFixedThreadPool(2);
//创建一个按照计划规定执行的线程池，这里核心数 = 2
ExecutorService executorService = Executors.newScheduledThreadPool(2);
//创建一个自动增长的线程池
ExecutorService executorService = Executors.newCachedThreadPool();
//创建一个具有抢占式操作的线程池
ExecutorService executorService = Executors.newWorkStealingPool();
```

**线程池的参数：**

* **corePoolSize：核心线程数**，当初始化线程池时，会创建核心线程进入等待状态，即使它是空闲的，核心线程也不会被摧毁，从而降低了任务一来时要创建新线程的时间和性能开销；
* **maximumPoolSize：最大线程数**，核心线程数都被用完了，那只能重新创建新的线程来执行任务，但是前提是不能超过最大线程数量；
* **keepAliveTime：线程存活时间**，除核心线程外被创建的线程可以存活多久，即这些线程完成任务，后面处于空闲状态，一定时间后被销毁；
* unit：存活时间单位；
* **workQueue：任务的阻塞队列**，未被执行的任务需要进入队列中排队，队列是FIFO的，等到线程空闲时，再取出里面的任务；

## newSingleThreadPool

单核心线程池，最大线程数只会有一个，时间为0表示无限的生命，不会被销毁；

```java
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```



## newFixedThreadPool

核心线程数固定，需要在创建时传入，并且核心线程数就是最大线程数，存活时间无限，不会销毁；

```java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```



## newCachedThreadPool

可进行缓存的线程池，常用的线程池。它的线程数是无限的，核心线程数为0，默认是存活时间是60s，即空闲60s后被销毁。有2种创建方法：

```java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

效果：当线程池中没有空闲线程时，这时有任务过来了，那么会新建线程执行任务。有空闲线程则会复用；

## newScheduledThreadPool

有计划性的线程池，可设置延迟时间，在给定的延迟时间之后运行，或者周期性的运行。与Timer定时器类似。构造函数：

```java
    /**
     * Creates a new {@code ScheduledThreadPoolExecutor} with the
     * given core pool size.
     *
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
     * @throws IllegalArgumentException if {@code corePoolSize < 0}
     */
    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue());
    }
```

内部用的延时阻塞队列维护任务进行，线程池并不常用。

## newWorkStealingPool

抢占式线程池，与其他4种不同，使用的是ForkJoinPool类，构造函数：

```java
    /**
     * Creates a thread pool that maintains enough threads to support
     * the given parallelism level, and may use multiple queues to
     * reduce contention. The parallelism level corresponds to the
     * maximum number of threads actively engaged in, or available to
     * engage in, task processing. The actual number of threads may
     * grow and shrink dynamically. A work-stealing pool makes no
     * guarantees about the order in which submitted tasks are
     * executed.
     *
     * @param parallelism the targeted parallelism level
     * @return the newly created thread pool
     * @throws IllegalArgumentException if {@code parallelism <= 0}
     * @since 1.8
     */
    public static ExecutorService newWorkStealingPool(int parallelism) {
        return new ForkJoinPool
            (parallelism,
             ForkJoinPool.defaultForkJoinWorkerThreadFactory,
             null, true);
    }
```

它是一个并行的线程池，入参是线程并发的数量，这个线程不会保证任务的顺序执行，哪个线程抢到任务，就由它执行
























