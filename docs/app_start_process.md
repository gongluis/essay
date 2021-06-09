#### 基本概念
1. launcher进程
2. SystemServer进程
```
是zygotefork出来的第一个进程，systemServer和zygote是android framework最重要的两个进程，系统里面最重要的服务都是在这两个进程中开启的，如:ActivityManageService PackageManagerService WindowManagerService
```
3. zygote进程
```
android所有进程都是zygite进程fork出来的
Android 开机流程
```
4. app进程

#### 流程
1. 启动进程，点击图标发生在launch进程，桌面实际上是一个launchactivity,点击图标相当于startactivity,通过binder进程通信机制发消息给system_server进程，在该进程中经过一系列操作amst通过socket通信告知zygote进程fork子进程（app进程）

2. 开启主线程app进程启动后，首先实例化activityThead,并执行其main函数，创建ApplicationThread，及handler体系

3. 创建并初始化Application和activity,始于activiyThread的main方法，通过binder通信告诉system-server进程中的ams我需要一个application,在ams的attachApplication中依次初始化appliaction和activity,分别有两个关键函数，thread#bindApplication()通知出现成handler创建application对象、绑定Context、执行Application的oncreat声明周期然后通过主线程handler消息通知创建activity对象，然后调用oncreat方法

#### 其它
1.每个进程都运行一个单独的Dvm,每个dvm占用一个Linux进程，独立的进程可防止虚拟机崩的时候所有程序都关闭。
2. 安卓是基于Linux系统，在linux中所有进程都是init进程直接或者间接fork出来的，fork进程比创建进程的效率高，在android中所有进程都是zygote进程fork出来的。

#### 开机流程
1. 安卓开机linux内核启动后，启动init进程
    * 开机动画
    * 文件系统的创建和挂在
    * 其他复杂工作
    * 最主要的是启动zygote进程，servicemanager(binder服务管理器，管理所有安卓系统服务)
2. zygote进程会fork出system server进程
    * 启动binder线程池
    * 创建system manager对象，会启动ams等
    * 启动桌面进程
3. zygote进程运行后，fork APP进程


