
## 性能优化
[TOC]
#### 内存优化  
内存泄漏是常客  
##### 什么是内存泄漏？  
```
内存不在GC的掌控范围内 
对一些对象有有限的周期声明，希望要做的事完成了，它会被垃圾回收器回收掉，但是有一些对这个对象的引用还存在，导致无法被回收，占用内存，持续累加，内存很快被耗尽OOM.
如一个activity的onDestroy被调用，但是如果有一个后台线程持有这个activity的引用，该activity占用的内存就不能被回收。
```  
##### 原因  
1. 错误使用单例-context(ApplicationContext)
2. handler  
```
在handler中设置ui元素
解决方案：
1. activity关闭的时候移除所有的消息和runable
2. 采用静态内部类+弱引用

```
3. thread
```
非静态内部类或者匿名类创建线程会隐式引用activity,线程run方法不执行完，线程是不会销毁的，所引用的activity也不会销毁。
解决方案：
线程内部使用弱引用/加个tag,activity销毁时修改tag值，不继续执行线程中的操作
```
4. AsyncTask
```
在AsyncTask中进行耗时操作，完成后，更新ui，有时会在耗时操作还没结束，activity就退出了，但是引用还被AsyncTask持有，无法释放导致内存泄漏。
解决方法：在ondestroy的时候，调用AsyncTask.cancel()
5. 资源未关闭BroadcastReceiver、注册后记得取消注册。
6. 动画资源未释放，无限循环动画
```

##### 影响？  
oom
##### 工具-leakcanary

2. java的GC内存回收机制是什么？  
答：对象在没有任何引用的时候才会被回收
3. GC回收机制的原理是什么？/可以作为GC Root引用点是啥？  
答：1.java栈中引用的对象2.方法静态引用的对象3.方法常量引用的对象4.Native中JNI引用的对象5.Thread-活着的线程。  

4. 内存泄漏原因？  
答:代码存在泄漏，内存无法释放/图片加载消耗大量内存，系统给应用分配内存即为应用能占用内存的最大值。
```
Android Profiler 测量应用性能  
```    
#### 代码优化  
##### 使用lint检查取出无效代码
1. 检测未使用的资源  
```
点击菜单栏 Analyze -> Run Inspection by Name -> unused resources -> Moudule ‘app’ -> OK
```
2. 检测无效代码  
```
步骤：点击菜单栏 Analyze -> Run Inspection by Name -> unused declaration -> Moudule ‘app’ -> OK
```
##### 代码规范优化
1. 避免创建不必要的对象，越少的对象意味着越少的GC操作  
```
1. 字符串拼接优先使用stringBuilder或stringBuffer,而不是+号连接，+会创建多余的对象，拼接的字符串越长，+号连接符的性能越低
2. 当一个方法返回值为string通常要判断string的作用，如果明确知道该方法返回的string再进行拼接可以考虑返回一个stringBuuffer对象来替代，将一个对象的引用返回，而返回string则是创建了一个短生命周期的临时对象。
```
2. 静态优于抽象
```
如果只是想调用来完成某项功能，可设置成静态的，不用专门调用这个方法而创建对象
```
3. 对常量使用static final修饰
4. 去除淡黄色警告
5. 节制的使用service,可考虑使用intentServie代替service  
6. 不适用静态变量保存核心数据  
```
android的进程并不是安全的，被kill掉后，app不会重新开始启动，android系统会创建一个新的Apllication对象然后启动上次用户离开时的activity,造成这个app从来没有退出的假象，静态变量数据由于进程被杀死而被初始化。
```
7. 轮询操作优化  
```
大概思路：当用户打开这个页面的时候初始化TimerTask对象，每个一分钟请求一次服务器拉取订单信息并更新UI，当用户离开页面的时候清除TimerTask对象，即取消轮训请求操作
```

#### UI优化（布局优化和绘制优化）   
> 每秒60帧，相当于每帧需要在大概16毫秒内完成渲染，否则就会卡顿

1. ui线程做耗时操作
2. Layout过于复杂，无法在16ms内完成渲染
3. 同一时间动画执行的次数过多，CPU GPU负荷过重
4. View频繁的触发measure layout
5. 内存频繁出发gc,stoptheword
    * onDraw方法不需要创建新的对象
    * onDraw方法不需要执行耗时操作循环
6. 无用的资源和逻辑导致加载和执行缓慢）  
    * Analyze->Run Inspection by Name… –>unused resource 
    * Analyze->Inspect Code
7. anr  

通过瞬时测试和压力测试更容易出发上面问题

如何来定位？  
```
1. 系统设置-开发者选项-调试GPU过度绘制，红色区域过度绘制，层级/relativelayout和linealayout
2. 
```
##### 布局优化  
1. include优化-重用公共布局
2. viewstub优化-仅在需要时才加载布局性能比VISISBLE好
3. merge优化-可以删减多余的层级，优化ui，但是无法预览
4. 减少重叠，背景图,selector normal改成透明

#### 速度优化（线程优化和网络优化）
> ANR Activity 5s触摸或者键盘输入事件无响应，broadcastReceiver 10s, service 20s

##### 线程优化
以上出现的anr多数由于线程阻塞引起的，可通过以下方法解决
* Asytask
* HandlerThread
* ThreadPool
* intentService
* 
##### 使用线程池  
```
直接创建Thread实现Runable的弊端，大量线程的创建和销毁很容易导致gc频繁的执行，发生内存抖动从而造成界面卡顿

为什么使用线程池？
1. 重用线程
2. 控制最大并发数量
3. 对线程进行一定的管理

例子：
Rxjava，rxandroid
```
##### 网络优化  
###### 图片：
* 图片优化使用webP格式，大幅节省流量，相对于jpg节省25~35%，对于png节省80%
* Bitmap优化 
    * 使用Android Matrix对图片缩放。
    * 如何在不使用 缩略图，不压缩，优化，使用android bitmapRigionDecode显示指定区域。
* Glide加载优化
  ```
  之前看到豆瓣的开源接口，思路很好，接口返回图片的数据有三种，高清大图、正常图片、缩略图，处于wifi用高清大图，4g或3g用正常，弱网条件下加载缩略图
  ```
* 
网络请求处理：  
* 对服务端返回的数据合理的缓存  
* 下载采用断点续传
* 刷新数据尽可能采用局部刷新  
* 
##### 加载Loading优化
1. 一种就是从A页面跳转到B页面时的加载loading,loading的时候页面是纯白色，加载完数据才显示内容页面
2. 在某个页面操作某种逻辑，局部loading(帧动画或者补间动画)，建议在销毁弹窗时，销毁动画。

#### 电量优化
> 重视程度低，音乐和视频耗电最大

* 网络请求前先判断网络的状态
* 网络请求最好进行批量处理，避免频繁间隔的网络请求
* 有wifi和移动网络优先使用wifi,请求wifi的耗电量远比移动耗电量低


#### 启动优化  
##### 冷启动  
耗时最长的，第一次启动app或者杀死进程第一次启动，主要包括application creat和activity的creat，因此在两个类的oncreat方法中做的事情越多，冷启动消耗的时间越长
##### 暖启动  
当应用中的Activity被销毁，但在内存中常驻时，启动方式就变成暖启动，减少了对象的初始化、布局加载等工作，启动时间更短，系统依然会展现闪屏页，直到第一个activity页面呈现
##### 热启动  
用户使用返回键退出应用，然后又马上重新启动应用  

```
1. 白屏
2. 闪屏页倒计时进行一些预加载
3. 主线程中涉及到的sp操作尽量在非ui线程执行
4. application创建过程尽量少的进行耗时操作
5. 减少布局的层次
```  
##### 启动页优化  

1. 启动时间优化，从Application的onCreat()方法开始到MainActivity的onCreat()执行结束
    * 延迟初始化
    * 后台任务
    * 启动界面预加载  
```
* intentService分担部分初始化工作
bugly\推送，热修复等。。。
```
2. 启动页白屏处理  
```
为什么会出现？
zygote在启动一个APP时，会创建一个新的进程去运行这个app,创建进程是需要时间的，创建完成之前，界面是假死状态，系统根据你的manifest文件设置的主题颜色来展示一个白屏或者黑屏，称为preview window，即预览窗口
实际就是activity默认主题的android:windowBackground为白色或者黑色决定的
APP启动--previewWindow--启动页

解决办法有透明主题
添加一个有背景的style样式
，然后闪屏页引用该theme主题，在该页面将window的背景图片设置为空


```
#### 分析性能的工具  
1. Systrace   进行埋点，使用门槛搞点，需要输入命令  
```
原理：在一些关键链路中插入label,通过label的开始和结束时间来确定某个核心过程的执行时间，framwork层和java从都插入了label信息，亦可以自定义label
系统版本越高，Framwork中添加的系统可用label就越多  

```
使用注意事项：  
```
1. 手动缩小范围
2. sdk/plaform-tools/systrace目录下执行命令
3. 执行结束后会生成一个trace.html文件
```  
自定义label  
```
在开始和结束的地方加上自定义label,抓一遍trace，分析问题
```  

如何分析非debug app  
```
debug和非debug性能差别很大，为了支持debug,系统开启了一些列功能，并且关闭掉了某些重要优化  ，要想分析app真实的运行情况必须要在非debug的APP中启动systrace功能。
通过反射调用sdk中一个@hide的函数
```  
2. traceView  
```
运行时开销很严重，trace会收集程序运行时所有方法的耗时情况
可以埋点
```
3. Profiler    
#### APK瘦身  
##### APK构成初探  
将apk包拖到android studio中就可以看到包结构了。  
```
lib 主要存储各cpu架构的so库
classes.dex java文件编译生成的字节码文件
res 资源文件
AndroidManifest.xml 配置文件
META-INFO 相关签名信息
resource.arsc 资源文件映射表
```  
##### 优化方案  
1. 使用minifyEnable混淆代码  
```
android {
    buildTypes {
        release {
            minifyEnabled true
        }
    }
}
proguard-rules.pro中编写混淆规则
```
2. 使用ShrinkResources移除无用资源  
```
android {
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
        }
    }
}
shrinkResource必须与minifyResource同时开启才有效。
注意：采用该方法其实无用的资源并没有被移除掉，而是被体积更小的占位符替代了
```
3. 删除项目中未使用到的资源文件
通过androidstudio提供的lint工具，analyze-run inspection by name然后输入unused resource查找未使用的资源文件，然后删除。
4. 删除项目中未使用的代码  
analyze-run inspection by name然后输入unused declearation 找到无用代码对其删除
> 通过反射引用的方法或者类，lint识别不了，也会给检查出来，删除的时候通过反射引用的方法或者类千万不能删除

5. 对图片进行压缩
    * 将Png图片换成体积更小的jpg格式。
    * 使用tinypng或者upng对图片进行压缩
    * 使用vector矢量图
    * 将图片转化成webP格式(要求最低的api是18)转换后jpg节省35%。png 80%
6. 使用shape代替图片
7. 使用Resource Configs进行语言配置
```
Android应用本身是支持国际化的，但国内的项目好多都是在国内使用，不需要国际化，这时候可以对相关的String进行减法操作，使其只包含特定的语言（例如中文）

android {
    defaultConfig {
        ...
        //语言资源，只支持中文
        resConfigs "zh-rCN"
    }
}
```
8. so库操作，在引入第三方sdk难免要引入对应的so库，一般给到的so库分好多架构（armeabi, armeabi-v7a, x86等等）
Android系统 目前支持以下七种不同的CPU架构：  
```
ARMv5
ARMv7
x86
MIPS
ARMv8
MIPS64
x86_64
```  
所有的架构设备x86,x8664,armeabi-v7a,arm64-v8a都支持armeabi架构的.so文件，所以可以根据自己的业务需求选择使用armeabi或者armeabi-v7a  

微信、qq、网易云音乐等都只保留了armeabi-v7a
```
android {
    defaultConfig {
        ndk {
            abiFilters 'armeabi','armeabi-v7a'
        }
    }
}
```
