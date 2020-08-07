#### 含义
装线程的池子，目的是管理线程，避免大量创建线程，增加开销，影响响应速度。

```
//五个参数的构造函数
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue)

//六个参数的构造函数-1
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory)

//六个参数的构造函数-2
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          RejectedExecutionHandler handler)

//七个参数的构造函数
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) 
```


```
corePoolSize -> 该线程池中核心线程数最大值

核心线程：在创建完线程池之后，核心线程先不创建，在接到任务之后创建核心线程。并且会一直存在于线程池中（即使这个线程啥都不干），有任务要执行时，如果核心线程没有被占用，会优先用核心线程执行任务。数量一般情况下设置为CPU核数的二倍即可。  


maximumPoolSize -> 该线程池中线程总数最大值

线程总数=核心线程数+非核心线程数。

非核心线程：简单理解，即核心线程都被占用，但还有任务要做，就创建非核心线程。

keepAliveTime -> 非核心线程闲置超时时长

这个参数可以理解为，任务少，但池中线程多，非核心线程不能白养着，超过这个时间不工作的就会被干掉，但是核心线程会保留。
  
  
TimeUnit -> keepAliveTime的单位
这个参数是个枚举类型
    * DAYS
    * HOURS
    * MINUTES
    * SECONDS
    * MILLISECONDS
    * MICROSECONDS
    * NANOSECONDS微毫秒
    
    
BlockingQueue workQueue -> 线程池中的任务队列

默认情况下，任务进来之后先分配给核心线程执行，核心线程如果都被占用，并不会立刻开启非核心线程执行任务，而是将任务插入任务队列等待执行，核心线程会从任务队列取任务来执行，任务队列可以设置最大值，一旦插入的任务足够多，达到最大值，才会创建非核心线程执行任务。

常见的workQueue有四种：

SynchronousQueue：这个队列接收到任务的时候，会直接提交给线程处理，而不保留它，如果所有线程都在工作怎么办？那就新建一个线程来处理这个任务！所以为了保证不出现<线程数达到了maximumPoolSize而不能新建线程>的错误，使用这个类型队列的时候，maximumPoolSize一般指定成Integer.MAX_VALUE，即无限大。

LinkedBlockingQueue：这个队列接收到任务的时候，如果当前已经创建的核心线程数小于线程池的核心线程数上限，则新建线程(核心线程)处理任务；如果当前已经创建的核心线程数等于核心线程数上限，则进入队列等待。由于这个队列没有最大值限制，即所有超过核心线程数的任务都将被添加到队列中，这也就导致了maximumPoolSize的设定失效，因为总线程数永远不会超过corePoolSize

ArrayBlockingQueue：可以限定队列的长度，接收到任务的时候，如果没有达到corePoolSize的值，则新建线程(核心线程)执行任务，如果达到了，则入队等候，如果队列已满，则新建线程(非核心线程)执行任务，又如果总线程数到了maximumPoolSize，并且队列也满了，则发生错误，或是执行实现定义好的饱和策略。

DelayQueue：队列内元素必须实现Delayed接口，这就意味着你传进去的任务必须先实现Delayed接口。这个队列接收到任务时，首先先入队，只有达到了指定的延时时间，才会执行任务。  

ThreadFactory threadFactory -> 创建线程的工厂

可以用线程工厂给每个创建出来的线程设置名字。一般情况下无须设置该参数。  

RejectedExecutionHandler handler -> 饱和策略

这是当任务队列和线程池都满了时所采取的应对策略，默认是AbordPolicy， 表示无法处理新任务，并抛出 RejectedExecutionException 异常。此外还有3种策略，它们分别如下。

CallerRunsPolicy：用调用者所在的线程来处理任务。此策略提供简单的反馈控制机制，能够减缓新任务的提交速度。
DiscardPolicy：不能执行的任务，并将该任务删除。
DiscardOldestPolicy：丢弃队列最近的任务，并执行当前的任务。

```  

#### 如何使用线程池  
以上是原理，线面说用法，java给我们提供了四种线程池  
* FixedThreadPool
* CachedThreadPool
* SingleThreadExecutor
* ScheduledThreadPool  
* 

1. FixedThreadPool  
```
可重用固定线程数的线程池，超出的线程会在队列中等待，在Executors类中我们可以找到创建方式

public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
    
    
FixedThreadPool的corePoolSize和maximumPoolSize都设置为参数nThreads，也就是只有固定数量的核心线程，不存在非核心线程。keepAliveTime为0L表示多余的线程立刻终止，因为不会产生多余的线程，所以这个参数是无效的。FixedThreadPool的任务队列采用的是LinkedBlockingQueue。

创建线程池的方法，在我们的程序中只需要，后面其他种类的同理：  

public static void main(String[] args) {
        // 参数是要线程池的线程最大值
        ExecutorService executorService = Executors.newFixedThreadPool(10);
}
```  
2. CachedThreadPool  

```
CachedThreadPool是一个根据需要创建线程的线程池。  

public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }  
    
    
CachedThreadPool的corePoolSize是0，maximumPoolSize是Int的最大值，也就是说CachedThreadPool没有核心线程，全部都是非核心线程，并且没有上限。keepAliveTime是60秒，就是说空闲线程等待新任务60秒，超时则销毁。此处用到的队列是阻塞队列SynchronousQueue,这个队列没有缓冲区，所以其中最多只能存在一个元素,有新的任务则阻塞等待。

```
3. SingleThreadExecutor
```
SingleThreadExecutor是使用单个线程工作的线程池。其创建源码如下

 public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
    
    
我们可以看到总线程数和核心线程数都是1，所以就只有一个核心线程。该线程池才用链表阻塞队列LinkedBlockingQueue，先进先出原则，所以保证了任务的按顺序逐一进行。  
```  

4. ScheduledThreadPool  
```
ScheduledThreadPool是一个能实现定时和周期性任务的线程池，它的创建源码如下：  

 public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }  
    
可以看出corePoolSize是传进来的固定值，maximumPoolSize无限大，因为采用的队列DelayedWorkQueue是无解的，所以maximumPoolSize参数无效。该线程池执行如下：  
当执行scheduleAtFixedRate或者scheduleWithFixedDelay方法时，会向DelayedWorkQueue添加一个实现RunnableScheduledFuture接口的ScheduledFutureTask(任务的包装类)，并会检查运行的线程是否达到corePoolSize。如果没有则新建线程并启动ScheduledFutureTask，然后去执行任务。

如果运行的线程达到了corePoolSize时，则将任务添加到DelayedWorkQueue中。DelayedWorkQueue会将任务进行排序，先要执行的任务会放在队列的前面。在跟此前介绍的线程池不同的是，当执行完任务后，会将ScheduledFutureTask中的time变量改为下次要执行的时间并放回到DelayedWorkQueue中。

```  

