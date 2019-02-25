# 1 ThreadPoolExecutor
该线程池是比较常用的线程池。参数如下：  
参数名 | 解释   
------------ | ------------     
corePoolSize | 核心线程池大小   
maximumPoolSize | 最大线程池大小  
keepAliveTime | 线程池中超过corePoolSize数目的空闲进程的最大存活时间；可以allowCoreThreadTimeOut(true)使得核心线程有效时间   
TimeUnit | keepAliveTime时间单位   
workQueue | 阻塞任务队列   
threadFactory | 新建线程工厂   
RejectedExecutionHandler | 当提交任务数超过maxmumPoolSize+workQueue之和时，任务会交给RejectedExecutionHandler来处理   

1.当线程池小于corePoolSize时，新提交任务将创建一个新线程执行任务，即使此时线程池中存在空闲线程。   
2.当线程池达到corePoolSize时，新提交任务将被放入workQueue中，等待线程池中任务调度执行   
3.当workQueue已满，且maximumPoolSize>corePoolSize时，新提交任务会创建新线程执行任务   
4.当提交任务数超过maximumPoolSize+workQueue时，新提交任务由RejectedExecutionHandler处理   
5.当线程池中超过corePoolSize线程，空闲时间达到keepAliveTime时，关闭空闲线程   
6.当设置allowCoreThreadTimeOut(true)时，线程池中corePoolSize线程空闲时间达到keepAliveTime也将关闭   

##  1.1 newFixedThreadPool
```java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```
创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
>创建线程池的时候默认将corePoolSize和maximumPoolSize设置成相同值，表示不会创建出更多线程。
```java
    public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>(),
                                      threadFactory);
    }
```
## 1.2 newSingleThreadExecutor
```java
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```
创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。
>这里就是同时将corePoolSize和maximumPoolSize设置成1。
```java
    public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory));
    }
```


## 1.3 newCachedThreadPool
```java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```
创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
+ 工作线程的创建数量几乎没有限制(其实也有限制的,数目为Interger. MAX_VALUE), 这样可灵活的往线程池中添加线程。
+ 如果长时间没有往线程池中提交任务，即如果工作线程空闲了指定的时间(默认为1分钟)，则该工作线程将自动终止。终止后，如果你又提交了新的任务，则线程池重新创建一个工作线程。
+ 在使用CachedThreadPool时，一定要注意控制任务的数量，否则，由于大量线程同时运行，很有会造成系统瘫痪。
```java
    public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>(),
                                      threadFactory);
    }
```
# 2 ForkJoinPool
ForkJoinPool原理类似分治法的思想，先把大的任务分成若干个小任务并计算，最后把所有小任务的计算结果合并起来。

## 2.1 newWorkStealingPool
```java
    public static ExecutorService newWorkStealingPool(int parallelism) {
        return new ForkJoinPool
            (parallelism,
             ForkJoinPool.defaultForkJoinWorkerThreadFactory,
             null, true);
    }
```
工作窃取线程池，默认所有线程都存在一个自己的任务队列，当自己线程所有任务执行完毕时，可以从别的线程的任务队列中获取到未执行的任务放入本线程执行，这就是工作窃取，使用该线程池可以有效提高CPU利用率。
每一个工作线程简单的通过以下两条原则进行活动：
+ 若队列非空，则代表自己线程的Task还没执行完毕，取出Task并执行。
+ 若队列为空，则随机选取一个其他的工作线程的Task并执行。
```java
    public static ExecutorService newWorkStealingPool() {
        return new ForkJoinPool
            (Runtime.getRuntime().availableProcessors(),
             ForkJoinPool.defaultForkJoinWorkerThreadFactory,
             null, true);
    }
```

# 3 ScheduledThreadPoolExecutor
该线程池继承自ThreadPoolExecutor。
## 3.1 newScheduledThreadPool
```java
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
```
```java
    public static ScheduledExecutorService newScheduledThreadPool(
            int corePoolSize, ThreadFactory threadFactory) {
        return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
    }
```
初始化的线程池可以在指定的时间内周期性的执行所提交的任务，在实际的业务场景中可以使用该线程池定期的同步数据。

未完，待续。。。
