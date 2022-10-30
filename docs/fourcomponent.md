[TOC]

## 四大组件及生命周期  

### 一、Activity  
1. 生命周期  
```
经典题目：A->B 问A和B的生命周期变化？然后从B返回A，问A和B的生命周期变化？
A(onPause)
B(onCreat)
B(onStart)
B(onResume)
A(onStop)
从B返回A
B(onPause)
A(onRestart)
A(onStart)
A(onResume)
B(onStop)
B(onDestroy)
```
主要知识点：掌握两组对应关系  
```
onStart-onResume
onPause-onStop
```
2. 启动模式  
```
activity四种启动模式：
1. standard 每次启动都会生成一个新的activity实例进栈，允许存在多个activity实例，默认启动模式
2. singleTop 当目标activity位于栈顶，再次启动时回调用目标activity的onNewintent方法，栈顶可复用
3. singleTask 如果没有设置新的任务栈属性，会在已有的任务栈种将位于该实例上面的activity全结束掉。
3. singleInstance 全局复用，一个独立的栈中，该栈中只有一个独立的activity,有坑

任务栈简述：app启动的时候系统会为application创建一个任务栈，activity的打开和关闭即入栈和出栈，先进后出

使用场景：singleTop,打开的activity位于栈顶，点击通知栏打开栈顶的activity,适用于childActivity
singletask:MainActivity,如何快速关闭100个activity,给跳转的activity设置singletask属性


```
ActivityA,ActivityB(singleInstance),ActivityC
A中启动B,B中启动C
此时按理在任务栈中的顺序从上到下是CBA
按理在C中按返回键应该回到B中，此时却回到了A中。
方法在B中定义全局变量returnActivityB,然后在A的onstart方法中判断全局变量returnActivityB，如果为true则跳转到B

3. 数据传递
```
1. intent  
2. startActivityForResult

```
4. 事件分发  
### 二、Service  
> service中能否进行耗时操作及如何 和activity通信 
1. 生命周期   
```
1. startService  
    * call to startService
    * onCreat
    * onStartCommand()
    * Service running
    * onDestroy
    * Service Shut Down
2. bindService
    * call to bindService
    * onCreat
    * onBind
    * Client are bind to Service
    * onUnbind
    * onDestroy
    * Service shut down
```
2. 启动模式  
```
1. startService
2. bindService 
两者区别：startService只是启动了service,启动它的组件（activity）没有关联,只有当调用stopSrewvice服务才会终止。bindService与其他组件进行了绑定，当启动方销毁，service会自行进行unbind操作，然后才会销毁service.
```
3. 使用方法
    * 第一步在Manifest中注册  
    * 第二步启动startService/bindService
    * 第三步解绑unBindService   
    * 第四步暂停stopService

4. 耗时任务

    Service不可以做耗时任务，onCreat方法是在主线程中调用的，耗时操作会阻塞ui,如果要采用耗时操作可在线程和handler方式，还有一种intentService,内部开了子县城并且任务执行完毕后会自动停止任务，而且多个耗时任务要执行的时候会按照顺序执行，原理就是内置的handler关联了任务队列

### 三、BroadcastReceiver

定义：你的应用可以对外部事件进行监听电话网络这些，也可以自定义发送广播事件。

使用：

 1. 注册，静态注册和动态注册，区别就是静态注册是一只在监听，动态activity关闭后即失效，需要自己控制。

    

### 四、ContentProvider

定义：使一个应用程序的指定数据集提供给其它应用，只有在多个应用程序间共享数据才需要内容提供者



