## 1.并行和并发有什么区别？
并发是多个事件在同一时间段执行，而并行是多个事件在同一个时间点执行。

## 2.线程和进程的区别？
+ 进程是资源分配的最小单位，线程是程序执行的最小单位。
+ 进程有自己的独立空间，每启动一个进程，系统就会为它分配地址空间，建立数据表来维护代码段，堆栈段和数据段，这种操作非常昂贵。而线程是共享进程中的数据的，使用相同的地址空间，因此CPU切换一个线程的花费远比进程要小得多，同时创建一个线程的开销也比进程要小很多。
+ 线程之间的通信更方便，同一进程下的线程共享全局变量，静态变量等数据，而进程之间的通信需要以通信的方式（IPC）进行。不过如何处理好同步与互斥是编写多线程程序的难点。
+ 但是多进程程序更健壮，多线程程序只要有一个程序死掉，整个进程也死掉了，而一个进程死掉并不会对另一个进程造成影响，因为进程有自己独立的地址空间。


**进程与线程的选择取决以下几点：**
1、需要频繁创建销毁的优先使用线程；因为对进程来说创建和销毁一个进程代价是很大的。
2、线程的切换速度快，所以在需要大量计算，切换频繁时用线程，还有耗时的操作使用线程可提高应用程序的响应
3、因为对CPU系统的效率使用上线程更占优，所以可能要发展到多机分布的用进程，多核分布用线程；
4、并行操作时使用线程，如C/S[架构](http://lib.csdn.net/base/architecture "大型网站架构知识库")的服务器端并发线程响应用户的请求；
5、需要更稳定安全时，适合选择进程；需要速度时，选择线程更好。

## 3.守护线程是什么？
守护线程（daemon thread），是个服务线程，准确地来说就是服务其他的线程，类似垃圾回收线程。

## 4.创建线程有哪几种方式？
+ 继承Thread类创建线程类。
+ 通过Runnable接口类创建线程类。
+ 通过Callable和Future创建线程。
> 采用实现Runnable、Callable接口的方式创见多线程时，优势是：
线程类只是实现了Runnable接口或Callable接口，还可以继承其他类。
在这种方式下，多个线程可以共享同一个target对象，所以非常适合多个相同线程来处理同一份资源的情况，从而可以将CPU、代码和数据分开，形成清晰的模型，较好地体现了面向对象的思想。
劣势是：
编程稍微复杂，如果要访问当前线程，则必须使用Thread.currentThread()方法。
使用继承Thread类的方式创建多线程时优势是：
编写简单，如果需要访问当前线程，则无需使用Thread.currentThread()方法，直接使用this即可获得当前线程。
劣势是：
线程类已经继承了Thread类，所以不能再继承其他父类。

## 5.说一下 runnable 和 callable 有什么区别？
1）Runnable提供run方法，不会抛出异常，只能在run方法内部处理异常。Callable提供call方法，直接抛出Exception异常，也就是你不会因为call方法内部出现检查型异常而不知所措，完全可以抛出即可。
2）Runnable的run方法无返回值，Callable的call方法提供返回值用来表示任务运行的结果
3）Runnable可以作为Thread构造器的参数，通过开启新的线程来执行，也可以通过线程池来执行。而Callable只能通过线程池执行。

## 6.线程有哪些状态？
+ NEW： 新建状态，线程对象已经创建，但尚未启动
+ RUNNABLE:就绪状态，可运行状态，调用了线程的start方法，已经在java虚拟机中执行，等待获取操作系统资源如CPU，操作系统调度运行。
+ BLOCKED:堵塞状态。线程等待锁的状态，等待获取锁进入同步块/方法或调用wait后重新进入需要竞争锁
+ WAITING：等待状态。等待另一个线程以执行特定的操作。调用以下方法进入等待状态。 Object.wait(), Thread.join(),LockSupport.park
+ TIMED_WAITING: 线程等待一段时间。调用带参数的Thread.sleep, objct.wait,Thread.join，LockSupport.parkNanos,LockSupport.parkUntil
+ TERMINATED:进程结束状态。

## 7.sleep() 和 wait() 有什么区别？
+ 每个对象都有一个锁来控制同步访问，Synchronized关键字可以和对象的锁交互，来实现同步方法或同步块。sleep()方法正在执行的线程主动让出CPU，在sleep指定时间后CPU再回到该线程继续往下执行**（注意：sleep方法只让出了CPU，而并不会释放同步资源锁！！！)**;wait()方法则是指当前线程让自己暂时退让出同步资源锁，以便其他正在等待该资源的线程得到该资源进而运行，只有调用了notify()方法，之前调用wait()的线程才会解除wait状态，可以去参与竞争同步资源锁，进而得到执行。**（注意：notify的作用相当于叫醒睡着的人，而并不会给他分配任务，就是说notify只是让之前调用wait的线程有权利重新参与线程的调度）**；
+ sleep()方法可以在任何地方使用；wait()方法则只能在同步方法或同步块中使用；
+ sleep()是线程线程类（Thread）的方法，调用会暂停此线程指定的时间，但监控依然保持，不会释放对象锁，到时间自动恢复；wait()是Object的方法，调用会放弃对象锁，进入等待队列，待调用notify()/notifyAll()唤醒指定的线程或者所有线程，才会进入锁池，不再次获得对象锁才会进入运行状态；
+ sleep()方法必须捕获异常，而wait()、notify（）、notifyAll（）不需要捕获异常。

## 8.线程的 run()和 start()有什么区别？
run()相当于线程的任务处理逻辑的入口方法，它由Java虚拟机在运行相应线程时直接调用，而不是由应用代码进行调用。
而start()的作用是启动相应的线程。启动一个线程实际是请求Java虚拟机运行相应的线程，而这个线程何时能够运行是由线程调度器决定的。start()调用结束并不表示相应线程已经开始运行，这个线程可能稍后运行，也可能永远也不会运行。
```java
public class test1 {
        public static void  main(String[] args) {
            Thread t = new Thread(){
                public void run(){
                    world();
                }
            };

            t.start();
            //t.run();
            System.out.print(" Hello ");
        }

        static void world(){
            System.out.print(" world ");
        }
}
```
> 用start()输出： Hello  world 
   用run()输出： world  Hello 

1.start（）方法来启动线程，真正实现了多线程运行。这时无需等待run方法体代码执行完毕，可以直接继续执行下面的代码；通过调用Thread类的start()方法来启动一个线程， 这时此线程是处于就绪状态， 并没有运行。 然后通过此Thread类调用方法run()来完成其运行操作的， 这里方法run()称为线程体，它包含了要执行的这个线程的内容， Run方法运行结束， 此线程终止。然后CPU再调度其它线程。
2.run（）方法当作普通方法的方式调用。程序还是要顺序执行，要等待run方法体执行完毕后，才可继续执行下面的代码； 程序中只有主线程——这一个线程， 其程序执行路径还是只有一条， 这样就没有达到写线程的目的。

## 9.线程池都有哪些状态？
1、RUNNING
(1) 状态说明：线程池处在RUNNING状态时，能够接收新任务，以及对已添加的任务进行处理。 
(2) 状态切换：线程池的初始化状态是RUNNING。换句话说，线程池被一旦被创建，就处于RUNNING状态，并且线程池中的任务数为0！

2、 SHUTDOWN
(1) 状态说明：线程池处在SHUTDOWN状态时，不接收新任务，但能处理已添加的任务。 
(2) 状态切换：调用线程池的shutdown()接口时，线程池由RUNNING -> SHUTDOWN。

3、STOP
(1) 状态说明：线程池处在STOP状态时，不接收新任务，不处理已添加的任务，并且会中断正在处理的任务。 
(2) 状态切换：调用线程池的shutdownNow()接口时，线程池由(RUNNING or SHUTDOWN ) -> STOP。

4、TIDYING
(1) 状态说明：当所有的任务已终止，ctl记录的”任务数量”为0，线程池会变为TIDYING状态。当线程池变为TIDYING状态时，会执行钩子函数terminated()。terminated()在ThreadPoolExecutor类中是空的，若用户想在线程池变为TIDYING时，进行相应的处理；可以通过重载terminated()函数来实现。 
(2) 状态切换：当线程池在SHUTDOWN状态下，阻塞队列为空并且线程池中执行的任务也为空时，就会由 SHUTDOWN -> TIDYING。 
当线程池在STOP状态下，线程池中执行的任务为空时，就会由STOP -> TIDYING。

5、 TERMINATED
(1) 状态说明：线程池彻底终止，就变成TERMINATED状态。 
(2) 状态切换：线程池处在TIDYING状态时，执行完terminated()之后，就会由 TIDYING -> TERMINATED。


## 10.多线程锁的升级原理是什么？
![锁升级](https://upload-images.jianshu.io/upload_images/16503287-5b1ba7469ed94a10.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 锁降级确实是会发生的，当JVM进入安全点（SafePoint）的时候，会检查是否有闲置的Monitor，然后试图进行降级。

**[锁的升级参考博客](https://blog.csdn.net/zqz_zqz/article/details/70233767)**


## 11.什么是死锁？
  线程死锁是指由于两个或者多个线程互相持有对方所需要的资源，导致这些线程处于等待状态，无法前往执行。当线程进入对象的synchronized代码块时，便占有了资源，直到它退出该代码块或者调用wait方法，才释放资源，在此期间，其他线程将不能进入该代码块。当线程互相持有对方所需要的资源时，会互相等待对方释放资源，如果线程都不主动释放所占有的资源，将产生死锁。

> 当然死锁的产生是必须要满足一些特定条件的： 
1.互斥条件：进程对于所分配到的资源具有排它性，即一个资源只能被一个进程占用，直到被该进程释放 
2.请求和保持条件：一个进程因请求被占用资源而发生阻塞时，对已获得的资源保持不放。 
3.不剥夺条件：任何一个资源在没被该进程释放之前，任何其他进程都无法对他剥夺占用 
4.循环等待条件：当发生死锁时，所等待的进程必定会形成一个环路（类似于死循环），造成永久阻塞。

## 12.怎么防止死锁？
+ 设置加锁顺序
+ 设置加锁时限
+ 死锁检测

## 13.多线程间的通信方式？
+ 同步
+ while循环
+ wait()/notify()
+ Lock+Condition
+ 管道

## 14.ThreadLocal 是什么？有哪些使用场景？
> + protected T initialValue()
> + public T get()
> + public void set(T value)
> + public void remove()

ThreadLocal适用于每个线程需要自己独立的实例且该实例需要在多个方法中使用，也即变量在线程间隔离而在方法或类间共享的场景。

## 15.ABA 问题?
如果另一个线程修改了V值，假设原来V值是A，先修改成B，再修改回成A，当前线程CAS操作无法分辨当前V值是否发生了变化。


## 16.synchronized 和 ReentrantLock 区别是什么？
ReentrantLock在加锁和内存上提供的语义与内置锁相同，此外它还提供了一些其他功能，包括定时的锁等待，可中断的锁等待，公平性，以及实现非块结构的加锁。ReentrantLock在性能上似乎优于内置锁。
ReebtrantLock的危险性比同步机制高，因为它不能自动释放锁，必须在finally块中手动释放。
> 在一些内置锁无法满足需求的情况下，ReentrantLock可以作为一种高级工具。当需要一些高级功能时才应该使用ReentrantLock，这些功能包括：可定时的，可轮询的与可中断的锁获取操作，公平队列，以及非块结构的锁。否则，还是应该优先使用synchronized。

## 17.CountDownLatch、CyclicBarrier和Semaphore常见用法？
#### CountDownLatch:
类似一种门闩，首先需要设定该头门上面需要加几个门闩，当某个线程被该latch阻塞时，需要做的就是等待其他线程执行chutdown来取下门闩，只有latch上的门闩数量为0，该扇门被打开，线程继续执行。它关注的单位是事件，同时无法重复使用。
> 常用方法：
> + void await()   
> + boolean await(long timeout,TimeUnit unit)
> + void countDown()
> + long getCount()
```java
public class demo4 {

    private volatile List<Integer> list=new ArrayList<>();
    private void add(int val){
        list.add(val);
    }
    private int get(){
        return list.size();
    }

    public static void main(String[] args) {
        demo4 d=new demo4();
        CountDownLatch cl=new CountDownLatch(1);

        new Thread(()->{
                if(d.get()!=5) {
                    try {
                        cl.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println("t2结束");
        },"t2").start();

        new Thread(()->{
                for (int i = 1; i <= 10; i++) {
                    d.add(i);
                    System.out.println("add:" + i);
                    if(i==5)  {
                        cl.countDown();
                    }
                    try {
                        TimeUnit.SECONDS.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
        },"t1").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }


    }
}
```
#### CyclicBarrier:
回环栅栏。需要让一组线程等待至某个状态后再同时继续执行，同时还支持当线程都到达状态时执行一个指定的方法。CyclicBarrier可以被重用。

> 常用构造函数和方法：
> + CyclicBarrier(int parties)  指定需要到达状态的线程数
> + CyclicBarrier(int parties, Runnable barrierAction) 指定需要达到状态的线程数，同时指定会执行的事件
> + int await()
> + int await(long timeout, TimeUnit unit)
> + int getNumberWaiting()  返回已到达屏障的线程数目。
> + int getParties()  返回还需到达屏障的线程数目。
> + void reset()        
```java
public class MyCyclicBarrier {

    public static void main(String[] args) throws InterruptedException {
        int N = 5;
        CyclicBarrier cb = new CyclicBarrier(N,() -> {
            System.out.println("人员已经坐满了"+Thread.currentThread().getName()+"开车了!");
        });
        for(int i=0;i<N*2; i++){
            TimeUnit.MILLISECONDS.sleep(2000);
            new Thread(new Passenger(cb)).start();
        }
    }


    static class Passenger implements Runnable{
        CyclicBarrier cb ;
        public Passenger(CyclicBarrier c){
            cb = c;
        }

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName()+"乘客开始等车！等待其他乘客上车！");
            try {
                cb.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println("开车了！");
        }
    }
}
```

#### Semaphore:
信号量。通过acquire()来获取一个或多个许可，通过release()来释放一个或多个许可。
> 常用构造函数和方法：
> + Semaphore(int permits)
> + Semaphore(int permits,boolean fair)
> + void acquire()
> + void acquire(int permits)
> + void acquireUninterruptibly() 从信号量获取许可证，阻塞直到可用为止
> + void acquireUninterruptibly(int permits)
> + void release()
> + void release(int permits)
> + boolean tryAcquire()
> + boolean tryAcquire(int permits)
> + boolean tryAcquire(int permits,long timeout,TimeUnit unit)
> + boolean tryAcquire(long timeout,TimeUnit unit)
```java
public class MySemaphore {

    public static void main(String[] args) {

        int N = 5;
        Semaphore s = new Semaphore(5);
        for(int i=0;i<10;i++){
            new Thread(new Worker(i,s)).start();
        }
    }

    static class  Worker implements Runnable{

        public int No;
        public Semaphore s;

        public Worker(int No,Semaphore s){
            this.No = No;
            this.s = s;
        }

        @Override
        public void run() {
            try {
                s.acquire();
                System.out.println("工人"+No+"开始获得了一台机器，开始工作了！");
                TimeUnit.MILLISECONDS.sleep(2000);
                System.out.println("工人"+No+"工作结束释放了一台机器！");
                s.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

