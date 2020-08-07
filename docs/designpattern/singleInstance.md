> 简单来说就是一个类只能构建一个对象的设计模式    

#### 第一种不加锁的单例  
>线程不安全
```
public class Singleton {
    private Singleton() {}  //私有构造函数
    private static Singleton instance = null;  
    //静态工厂方法
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

```  
饿汉式   
>在类被初始化的时候就已经在内存中创建了对象，以空间换时间，不存在线程安全问题。
```
public class SingleTon{
    private static SingleTon INSTANCE = new SingleTon();
    private SingleTon(){}
    public static SingleTon getInstance(){
        return INSTANCE;
    }
}
```
  
懒汉式  
> 在方法被调起后才创建对象，以时间换空间，在多线程环境下存在风险。  
```

public class SingleTon{
   private static SingleTon  INSTANCE = null;
   private SingleTon(){}
   public static SingleTon getInstance() {  
   if(INSTANCE == null){
      INSTANCE = new SingleTon(); 
    } 
    return INSTANCE；
  }
  }
```
#### 简单加上synchronized锁   
> 双重检测(double check lock)是为了b线程在a线程刚结束的临界点执行锁内任务，存在DCL失效问题
```
public class Singleton {
    private Singleton() {}  //私有构造函数
   private static Singleton instance = null;  //单例对象
   //静态工厂方法
   public static Singleton getInstance() {
        if (instance == null) {      //双重检测机制
         synchronized (Singleton.class){  //同步锁
           if (instance == null) {     //双重检测机制
             instance = new Singleton();
               }
            }
         }
        return instance;
    }
}

```


#### 以上实现还是有问题 ，单例第三种写法 
> JVM编译器有个指令重排的概念  
* 什么是指令重排  
java一个简单的instance = new instance()会被jvm编译成如下jvm指令  
```
1. memory  = allocate;分配对象的内存地址
2. creatinstance(memory);初始化对象
3. instance = memory设置instance指向刚分配的内存地址
```
但是这些指令顺序并非是一重不变，可能会经过jvm和cpu的优化，进行重排序  
```
1.分配对象内存地址
3.设置instance指向刚分配的内存地址
2.初始化对象
```
当线程A执行完13的时候，线程B抢到资源，返回了一个还没有初始化完成的对象
* 如何避免以上问题？  
在instance对象前面增加一个volatile修饰符  


```
public class Singleton {
    private Singleton() {}  //私有构造函数
    private volatile static Singleton instance = null;  //单例对象
    //静态工厂方法
    public static Singleton getInstance() {
          if (instance == null) {      //双重检测机制
         synchronized (Singleton.class){  //同步锁
           if (instance == null) {     //双重检测机制
             instance = new Singleton();
                }
             }
          }
          return instance;
      }
}
```
volatile阻止了变量访问访问前后指令重排序，对于线程B来说，instance要么指向null，要么指向一个完整的instance,而不会出现中间状态  
> 不仅可以防止指令重排，也可以保证线程访问的变量值是主内存种最新值，jdk1.6及以后才支持，每次均从主内存中读取，牺牲点效率，无伤大雅。

#### 静态内部类  
> 静态内部类只会被加载一次，首次调用，顾线程安全，类加载初始化阶段是单线程的同时延迟了初始化。  

类加载时机
```
1. new,involk static,putstatic,getstatic,指令时若类未加载则触发
2. 反射使用某个类时，若类未加载则触发
3. 子类加载时若父类未加载则触发
4. 程序开始时主方法所在类会被加载

```  
静态内部类的懒加载应该属于第一种情况，为什么外部类加载时，内部类未加载，静态内部类只是刚好写在了另一个类里面，实际上和外部类没什么附属关系。
  
```
public class Singleton{
private SingleTon (){}
private static class Holder{
//这里的私有没有什么意义
    private static SingleTon instance = new Singleton();
} 
    
    public static Singleton getInstance(){
        return Holder.instance
    }
}
```

#### 如何通过反射打破单例模式，只能构建一个对象
```
//获得构造器
Constructor con = Singleton.class.getDeclaredConstructor();
//设置为可访问
con.setAccessible(true);
//构造两个不同的对象
Singleton singleton1 = (Singleton)con.newInstance();
Singleton singleton2 = (Singleton)con.newInstance();
//验证是否是不同对象
System.out.println(singleton1.equals(singleton2));

```  

如何防止反射？  
使用枚举实现单例  
```
public enum SingletonEnum {
    INSTANCE;
}
```
> 使用枚举单例，不仅可以防止反序列化，而且可以保证枚举对象被反序列化的时候，返回的对象是同一个对象
```
public enum SingletonEnum {
    INSTANCE;
    public void doSomething(){
        System.out.printLn("do something");
    }
}
```  

```
public enum SingletonEnum{
    PERSON;
    private Person person = null;
    private SingletonEnum(){
        person = new Person();
    }
    public Person getPerson(){
        return person;
    }
    
}

public class Person{
    
}
```



#### 防止内存泄漏单例
```
public class A8PinPadManage {


    private A8PinPadManage(){} //私有构造函数
    private volatile static Pinpad instance; //单例对象

    public static Pinpad getInstance(Context context) {
        WeakReference<Context> contextWeakReference = new WeakReference<>(context);
        Context weakReferenceContext = contextWeakReference.get();

        if (instance == null) {
            synchronized (A8PinPadManage.class) {
                if (instance == null) {
                    instance = new Pinpad((int) SharedPrefsUtil.get(weakReferenceContext, CIL_KAP_ID, 2), "IPP");
                }
            }
        }
        return instance;
    }
}
```
* 解读  
    * 私有构造函数，全局只能构建一个对象，不能让随便的就去new,所以要私有
    * 懒汉，单例刚开始没有构建，调用才构建
    * 饿汉，调用new singleton主动构建，不需要判空
    * instance的初始值可以写成null或者new Singleton()
    * 不加锁，线程不安全
    * 不加双重验证直接锁getinstance方法，可以，增加内存开销，只用synchronize里面的判空，增加开销  
    
#### 参考  小灰漫画
```
https://www.zhihu.com/search?type=content&q=%E5%8D%95%E4%BE%8B
```