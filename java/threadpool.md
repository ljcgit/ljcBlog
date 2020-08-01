## 1.执行流程
 
 一个任务被提交给线程池后，如果正在运行的线程数量少于核心线程池大小的，就通过创建一个新线程来执行提交的任务（无论别的线程是否空闲）。
 如果线程数大于核心线程数，就会将任务先放入队列中，直到队列满，就创建新线程，线程总数不能超过最大线程数。
 当线程数量超过核心线程数后，如果这些线程处于闲置状态，并且闲置了keepAliveTime时间，就会将这些超过核心线程数的线程停止，来减少资源的浪费。
 同时可以通过调用allowCoreThreadTimeOut方法来允许核心线程也只能显示keepAliveTime时间。
 
 
 ## 2.任务队列
 + Direct handoffs：代表是SynchronousQueue，相当于队列大小为1，一个线程想要提交任务必须等待其他线程把任务取走。由于队列大小为1，就会导致创建出许多新线程，这就需要一个没有边界的最大线程数量。
 + Unbounded queues：代表是LinkedBlockingQueue，无边界队列，最大线程数量往往和核心线程数量一样。
 + Bounded queues：代表是ArrayBlockingQueue，固定大小队列，可以根据应用场景，合理的调整线程数的大小和队列大小。
 
 
 ## 3.拒绝策略
 当线程池处于shutdown状态或者线程数量和队列都已达到最大值，就会调用rejectedExecution方法来拒绝新提交的任务。
 
 有四个拒绝策略：
 + 默认是直接抛出RejectedExecutionException异常；
 + 直接通过当前线程执行；
 + 直接抛弃该任务；
 + 抛弃队列第一个任务，重新将新任务添加到队列中。
 
 ## 4.CTL
 ctl是线程池的状态位，包含了工作线程数和线程池状态，工作线程数用低29位表示，该值可能与实际存活的线程数不同，比如说存在线程创建失败的情况，线程池状态用高3位表示。<br/>
 ```java
     private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
    private static final int COUNT_BITS = Integer.SIZE - 3;
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

    // runState is stored in the high-order bits
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;

    // Packing and unpacking ctl
    private static int runStateOf(int c)     { return c & ~CAPACITY; }
    private static int workerCountOf(int c)  { return c & CAPACITY; }
    // 通过或运算符计算线程池状态
    private static int ctlOf(int rs, int wc) { return rs | wc; }
 ```
 
 线程池一共有5种状态：
 + RUNNING：既接受新任务，也能继续处理已提交的任务；
 + SHOTDOWN：不接受新任务，能继续处理已提交的任务；
 + STOP：不接受新任务，不能继续处理已提交的任务，同时中断正在执行的任务；
 + TIDYING：所有任务执行完毕，并执行terminated方法；
 + TERMINATED：terminated方法执行完毕。
 
 
  RUNNING -> SHUTDOWN：调用shutdown()；
  (RUNNING or SHUTDOWN) -> STOP：调用shutdownNow()；
  SHUTDOWN -> TIDYING：队列和线程池都为空；
  STOP -> TIDYING：线程池为空；
  TIDYING -> TERMINATED：terminated方法执行完毕。


## 5.线程池构造方法参数
```java
    public ThreadPoolExecutor(int corePoolSize,       //核心线程数大小
                              int maximumPoolSize,    //最大线程数大小
                              long keepAliveTime,     //空闲线程存活时间
                              TimeUnit unit,          //存活时间单位
                              BlockingQueue<Runnable> workQueue,   //阻塞队列
                              ThreadFactory threadFactory,    //线程工厂
                              RejectedExecutionHandler handler) {    //拒绝策略
    }
```

## 6.execute方法
```java
 public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        int c = ctl.get();
        //判断当前线程数是否小于核心线程数
        if (workerCountOf(c) < corePoolSize) {
            // 新建线程
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }
 ```
执行任务主要分为三种：
1.如果是少于核心线程数的线程在运行，就把提交的任务作为线程的任务，启动一个新线程；
2.将任务成功添加到任务队列后，要进行第二次检查，避免原先运行的线程停止或者线程处于shutdown状态及以上，然后要么将原来的任务从队列中移除，要么当工作线程为0时新建一个线程；
3.如果加入队列失败，就会新添加一个线程，如果添加线程失败，说明线程数达到饱和或者线程池处于shutdown状态及以上。
