[Toc]
#### 定义的理解
> 主线程、工作线程、线程通信
1. 主线程
应用启动时，系统会为该应用创建一个主线程，该线程很重要，所有的ui操作必须在该线程执行，并且该线程不能阻塞，5s会发生anr。
2. 工作线程
主线程外用来执行任务的线程，执行完任务后必须通过主线程更新界面。
3. 工作线程和主线程通信
    * Activity.runOnUiThread(Runnable)
    * View.post(Runnable)
```
public void onClick(View v) {
    new Thread(new Runnable() {
        public void run() {
            // a potentially time consuming task
            final Bitmap bitmap =
                    processBitMap("image.png");
            imageView.post(new Runnable() {
                public void run() {
                    imageView.setImageBitmap(bitmap);
                }
            });
        }
    }).start();
}
```
3. Handler  
处理复杂的线程间通信
4. AsyncTask api30不推荐了，concurrent代替
```
1. 创建AsyncTask的子类并实现doInBackground,onProgressUpdate,onpostExcute
private class DownloadFilesTask extends AsyncTask<URL, Integer, Long> {
     protected Long doInBackground(URL... urls) {
         int count = urls.length;
         long totalSize = 0;
         for (int i = 0; i < count; i++) {
             totalSize += Downloader.downloadFile(urls[i]);
             publishProgress((int) ((i / (float) count) * 100));
             // Escape early if cancel() is called
             if (isCancelled()) break;
         }
         return totalSize;
     }

     protected void onProgressUpdate(Integer... progress) {
         setProgressPercent(progress[0]);
     }

     protected void onPostExecute(Long result) {
         showDialog("Downloaded " + result + " bytes");
     }
 }
 2.开始该任务 
 new DownloadFilesTask().execute(url1, url2, url3);

```

#### Thread类的常用方法  
1. start()方法：开始执行该线程
3. stop()方法：强制结束该线程
4. join()方法 ：等待该线程结束
5. sleep()方法：该线程进入等待
6. run()方法 :直接执行该线程的run方法（线程调用start()也会执行run方法，区别是一个是由线程调度运行run 方法，一个是直接调用线程中的run方法）
>注意：wait()和notify()是object中的方法，分别表示线程挂起和线程恢复
wait()与sleep()的区别：wait()会释放对象锁，sleep()不会释放对象锁


#### 线程的5大状态与转换
1. 新建状态：新建线程对象，并没有调用start之前；
2. 就绪状态：调用start方法之后就进入就绪状态，另外线程在睡眠和挂起中恢复的时候也会进入就绪状态；
3. 运行状态：线程被设置为当前线程开始执行run方法；
4. 阻塞状态：线程被暂停，比如调用sleep方法后；
5. 死亡状态：线程执行结束。
> 注意，由阻塞状态不可以直接回到运行状态，要先经历就绪状态然后到运行状态

#### 锁的类型  
1. 可重入锁：在执行对象中所有同步方法不用再次获得锁
```
例子，排队打水，一家中任何人先到了，这家的其他人来了都可以不用排队直接打水。
实现细节：
A线程正在执行任务，B线程也来排队
A线程（state:1 队列头指针：null 队列尾指针：B线程节点）
这个时候A线程又来请求锁，只是把状态值state改成了2,如果A线程释放了一个锁就-1
就是一个线程获取到了锁后再次去获取同一个锁，仅仅把状态值进行累加，

非公平锁模型：当线程A执行完之后，要唤醒线程B是需要时间的，而且线程B醒来后还要再次竞争锁，所以如果在切换过程当中，来了一个线程C，那么线程C是有可能获取到锁的，如果C获取到了锁，B就只能继续乖乖休眠了。
```
2.可中断锁：在等待获取锁过程中可中断
3.公平锁：按等待获取锁的线程的等待时间进行获取，等待时间长的具有优先获取锁的权力
4. 读写锁：对资源读取和写入的时候拆分为2部分处理，读的时候可以多线程一起读，写的时候必须同步的写

#### synchronized和lock的区别和使用  
synchronized  
存在层次：Java的关键字，在jvm层面上；  

锁的释放：1.以获取锁的线程执行同步代码，释放锁；
2.线程执行发生异常，jvm会让线程释放锁；  

锁的获取：假设A线程获得锁，B线程等待，如果A线程阻塞，B线程会一直等待；  

锁状态：无法判断；  

锁类型：可重入，不可中断，非公平；
性能：少量同步

不可中断锁


lock  
存在层次：是一个类  

锁的释放：在finally中必须释放锁，不然容易造成线程死锁  
锁的获取：分情况而定，Lock有多个锁获取的方式，可以尝试获得锁  

锁状态：可以判断  

锁类型：可重入，可判断，可公平（两者皆可）  

性能：大量同步
##### 设置线程优先级  
* Thread.setPrioriy()
* Process.setThreadPriority()  
* 
第一种是java原生方法，区间是1~10  
第二种是android，范围-19~19推荐  

原因;java设计的1~10不能对应每一个平台的等级比如linuxAndroid，就是-5~4
#### 锁方法和锁静态方法区别？锁方法和锁方法块区别？

静态方法是类锁，方法是对象锁
锁方法对象锁，所代码块，锁对象不一样

#### 参考博客  
https://www.cnblogs.com/yulinfeng/p/11020576.html

#### 并发编程
##### CountDownLatch用法  
CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；
> 利用它可以实现类似计数器的功能。比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。

* CountDownLatch类只提供了一个构造器：
```
public CountDownLatch(int count) {  };  //参数count为计数值
```
* 然后下面这3个方法是CountDownLatch类中最重要的方法：
```
public void await() throws InterruptedException { };   //调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
public boolean await(long timeout, TimeUnit unit) throws InterruptedException { };  //和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
public void countDown() { };  //将count值减1
```
* 例子
```
public class Test {
     public static void main(String[] args) {   
         final CountDownLatch latch = new CountDownLatch(2);
          
         new Thread(){
             public void run() {
                 try {
                     System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
                    Thread.sleep(3000);
                    System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
                    latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
             };
         }.start();
          
         new Thread(){
             public void run() {
                 try {
                     System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
                     Thread.sleep(3000);
                     System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
                     latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
             };
         }.start();
          
         try {
             System.out.println("等待2个子线程执行完毕...");
            latch.await();
            System.out.println("2个子线程已经执行完毕");
            System.out.println("继续执行主线程");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
     }
}
```
执行结果：
```
线程Thread-0正在执行
线程Thread-1正在执行
等待2个子线程执行完毕...
线程Thread-0执行完毕
线程Thread-1执行完毕
2个子线程已经执行完毕
继续执行主线程
```
##### Semaphore用法  
Semaphore其实和锁有点类似，它一般用于控制对某组资源的访问权限。
> 　Semaphore翻译成字面意思为 信号量，Semaphore可以控同时访问的线程个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可

* Semaphore类位于java.util.concurrent包下，它提供了2个构造器：
```
public Semaphore(int permits) {          //参数permits表示许可数目，即同时可以允许多少线程进行访问
    sync = new NonfairSync(permits);
}
public Semaphore(int permits, boolean fair) {    //这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可
    sync = (fair)? new FairSync(permits) : new NonfairSync(permits);
}
```

* 下面说一下Semaphore类中比较重要的几个方法，首先是acquire()、release()方法：
```
public void acquire() throws InterruptedException {  }     //获取一个许可
public void acquire(int permits) throws InterruptedException { }    //获取permits个许可
public void release() { }          //释放一个许可
public void release(int permits) { }    //释放permits个许可
```
acquire()用来获取一个许可，若无许可能够获得，则会一直等待，直到获得许可。

　　release()用来释放许可。注意，在释放许可之前，必须先获获得许可。

　　这4个方法都会被阻塞，如果想立即得到执行结果，可以使用下面几个方法：
　　
```
public boolean tryAcquire() { };    //尝试获取一个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException { };  //尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
public boolean tryAcquire(int permits) { }; //尝试获取permits个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException { }; //尝试获取permits个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
```
另外还可以通过availablePermits()方法得到可用的许可数目。

* 例子：假若一个工厂有5台机器，但是有8个工人，一台机器同时只能被一个工人使用，只有使用完了，其他工人才能继续使用。那么我们就可以通过Semaphore来实现：
```
public class Test {
    public static void main(String[] args) {
        int N = 8;            //工人数
        Semaphore semaphore = new Semaphore(5); //机器数目
        for(int i=0;i<N;i++)
            new Worker(i,semaphore).start();
    }
     
    static class Worker extends Thread{
        private int num;
        private Semaphore semaphore;
        public Worker(int num,Semaphore semaphore){
            this.num = num;
            this.semaphore = semaphore;
        }
         
        @Override
        public void run() {
            try {
                semaphore.acquire();
                System.out.println("工人"+this.num+"占用一个机器在生产...");
                Thread.sleep(2000);
                System.out.println("工人"+this.num+"释放出机器");
                semaphore.release();           
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
执行结果：  
```
工人0占用一个机器在生产...
工人1占用一个机器在生产...
工人2占用一个机器在生产...
工人4占用一个机器在生产...
工人5占用一个机器在生产...
工人0释放出机器
工人2释放出机器
工人3占用一个机器在生产...
工人7占用一个机器在生产...
工人4释放出机器
工人5释放出机器
工人1释放出机器
工人6占用一个机器在生产...
工人3释放出机器
工人7释放出机器
工人6释放出机器
```
##### reentrantLock 
> concurrent包下高级并发工具 须先获取到锁，再进入try {...}代码块，最后使用finally保证释放锁,可以使用tryLock()尝试获取锁。

Java语言直接提供了synchronized关键字用于加锁，但这种锁一是很重，二是获取时必须一直等待，没有额外的尝试机制。

```
public class Counter {
    private int count;

    public void add(int n) {
        synchronized(this) {
            count += n;
        }
    }
}
```
如果用ReentrantLock替代，可以把代码改造为：  
```
public class Counter {
    private final Lock lock = new ReentrantLock();
    private int count;

    public void add(int n) {
        lock.lock();
        try {
            count += n;
        } finally {
            lock.unlock();
        }
    }
}
```
* 高级用法  
ReentrantLock是可重入锁，它和synchronized一样，一个线程可以多次获取同一个锁。

和synchronized不同的是，ReentrantLock可以尝试获取锁：

```
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try {
        ...
    } finally {
        lock.unlock();
    }
}
```  
上述代码在尝试获取锁的时候，最多等待1秒。如果1秒后仍未获取到锁，tryLock()返回false，程序就可以做一些额外处理，而不是无限等待下去。

所以，使用ReentrantLock比直接使用synchronized更安全，线程在tryLock()失败的时候不会导致死锁。