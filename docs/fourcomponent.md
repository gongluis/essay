## 四大组件及生命周期  
### Activity  
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
1. standard 标准的栈结构
2. singleTop 在栈顶可复用
3. singleInstance 全局复用，一个独立的栈中，有坑
```
ActivityA,ActivityB(singleInstance),ActivityC
A中启动B,B中启动C
此时按理在任务栈中的顺序从上到下是CBA
按理在C中按返回键应该回到B中，此时却回到了A中。
方法在B中定义全局变量returnActivityB,然后在A的onstart方法中判断全局变量returnActivityB，如果为true则跳转到B
```
4. singleTask 同栈的复用，顶出栈
经典题目：各个模式的使用场景
```
3. 数据传递
```
1. intent  
2. startActivityForResult

```
4. 事件分发  
### Service  
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
```  
3. 使用方法
    * 第一步在Manifest中注册  
    * 第二步启动startService/bindService
    * 第三步解绑unBindService   
    * 第四步暂停stopService
4. Activity和Service通信  
