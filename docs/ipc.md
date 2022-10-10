

> 在安卓中进程间无法共享内存（用户空间），不同进程之间一般通过AIDL（安卓接口定义语言）进行通信

使用流程

```
1.服务端在.aidl文件中定义AIDL接口并rebuild。一般放到main目录下与java目录同级
2.rebuild后在build-->generated-->aidl_source_output_dir-->debug文件夹中生成与aidl文件同名的java文件
3.创建MyService实现Service复写onBind方法，并在manifest文件夹中注册该服务，定义action 和category
4.将服务端的aidl接口及上层文件夹拷贝到客户端（包名要与服务端保持一致）并rebuild项目。
5.new Intent设置setPackage和setAction然后bindService,定义ServiceConnection,在onServiceConnection中获取service实例。
6.通过service实例调用定义的aidl方法。

```

#### 一、服务端

1. 创建.aidl文件并定义接口

   ```
   创建：选中文件夹右键 new-->AIDL-->AIDL File
   接口：aidl可以通过带参数及返回值一个或多个方法来声明接口，aidl中支持的数据类型如下
   	1.java的8种数据类型：byte short int long float double boolean char
   	2.String charSequence List Map
   	3.回调监听listener
   	4.实现parceble的bean
   ```

   

#### 二、客户端

1. 将服务端aidl接口连文件夹（包名）复制过来rebuild

2. new Intent设置setPackage和setAction然后bindService,定义ServiceConnection,在onServiceConnection中获取service实例。

   ```
   				Intent intent = new Intent();
           intent.setPackage("com.t3.aibox");
           intent.setAction("com.t3.aibox.service.aiboxService.vdrsdk");
           context.bindService(intent, serviceConnection, Context.BIND_AUTO_CREATE);
           
           private ServiceConnection serviceConnection = new ServiceConnection() {
           @Override
           public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
               Log.d(TAG, "onServiceConnected: " );
               vdrSdkService= VdrSdkService.Stub.asInterface(iBinder);//后续通过该service调用aidl中具体的方法
               isConnect=true;
           }
   
           @Override
           public void onServiceDisconnected(ComponentName componentName) {
               isConnect = false;
               Log.d(TAG, "onServiceDisconnected: " );
           }
       };
       
       
       //调用实例
       boolean isRecording = vdrSdkService.isRecording();
       
   ```

   