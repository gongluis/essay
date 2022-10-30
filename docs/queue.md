#### 一、为什么要使用队列？

保持任务执行的有序性和对任务的管理控制如设置最大任务处理数，设置饱和策略等

```
private static final int QUEUE_CAPACITY = 30;//队列容量定义为30
private static ArrayBlockingQueue<Bean> taskQueue = new ArrayBlockingQueue(QUEUE_CAPACITY);//新建队列

public void doWork(Bean Bean) {
        if (taskQueue.size() == QUEUE_CAPACITY) {//饱和处理---丢弃
            try {
                Bean poll = taskQueue.poll();//丢弃队列中靠前的任务 并回调结果
                LogExtKt.log(TAG, "队列任务超过限度三十，丢弃该任务: " + poll.ID + poll.index);
                Data data = getBaseData(poll);
                data.interceptResult = InterceptPicConfig.RESULT_REMOTE_OVER_TASK;
                poll.picresultListener.onResult(data);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
        taskQueue.offer(interceptTaskBean);//添加进队列
        Log.d(TAG, "添加队列任务: " + TaskBean.ID + " index:" + TaskBean.index
                + " size:" + taskQueue.size());
    }


```

#### 二、队列中取任务如何设计？

单例+线程池+死循环

```
 private ExecutorService service = Executors.newSingleThreadExecutor();
 private void startThread() {
        service.execute(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    if (isWorking) {
                        Log.d(TAG, "task is working: continue");
                        continue;
                    }
                    isWorking=true;
                    doWork();//调用一、中方法
                    isWorking=false;
                }
            }
        });
    }
```

