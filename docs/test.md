### 单元测试JUNIT4   
每个项目都是由成百上千个函数组成，每编写完一个函数都要对这个函数的方方面面进行测试，这样的测试称之为单元测试。
#### 本地单元测试Local Unit Test  
这种测试运行在本地开发环境的java虚拟机上，不需要连接安卓设备或者虚拟器，无法获得安卓相关api,只能使用java相关api  
测试类代码编写也很简单，主要通过一些注解和断言来实现。  
##### Junit4 注解
```
@Before标注setup方法，每个单元测试用例方法调用之前都会调用
@After标注teardown方法，每个单元测试用例方法调用之后都会调用
@Test标注的每个方法都是一个测试用例
@BeforeClass标注的静态方法，在当前测试类所有用例方法执行之前执行
@AfterClass标注的静态方法，在当前测试类所有用例方法执行之后执行
@Test(timeout=)为测试用例指定超时时间
```  

##### 断言  

```
Junit提供了一系列断言来判断是pass还是fail

assertTrue(condition)：condition为真pass，否则fail
assertFalse(condition)：condition为假pass，否则fail
fail()：直接fail
assertEquals(expected, actual)：expected equal actual pass，否则fail
assertSame(expected, actual)：expected == actual pass，否则fail

```  

#### 设备单元测试Instrumented Unit Test   

这种测试方法需要连接安卓设备或者模拟器，可以调用android中api(如设备信息，上下文等)  ，使用方法是给类加上注解@RunWith(AndroidJunit4.class)注解作为前缀。  
```
@RunWith(AndroidJUnit4.class)
public class ExampleInstrumentedTest {
    @Test
    public void useAppContext() throws Exception {
        // Context of the app under test.
        Context appContext = InstrumentationRegistry.getTargetContext();
        assertEquals("top.qingningshe.test", appContext.getPackageName());
    }
}
```  
  
  相对于Local Unit Test 多了访问设备信息和测试筛选  
##### 访问设备信息  
    可以使用instrumentationRegisty类访问与测试运行相关的信息，包括instrumentation对象，目标应用context对象，测试应用context对象。  
##### 测试筛选  
    * @RequiresDevice:指定测试仅在物理设备而不在模拟器上运行。
    * @SdkSupress:禁止在低于给定级别的Android API 上运行测试，如：@SDKSupress(minSdkVersion=18)  
    * @SmallTest、@MediumTest和@LargeTest：指定测试的运行时长以及运行频率。
  
### 压力测试Monkey  
Monkey是Android中的一个命令行工具，可以运行在模拟器里或实际设备中。它向系统发送伪随机的用户事件流(如按键输入、触摸屏输入、手势输入等)，实现对正在开发的应用程序进行压力测试。Monkey测试是一种为了测试软件的稳定性、健壮性的快速有效的方法。
  
 #### mokey的特点  
  ```
  测试的对象仅为应用程序包，有一定的局限性。
Monky测试使用的事件流数据流是随机的，不能进行自定义。
可对MonkeyTest的对象，事件数量，类型，频率等进行设置。
  ```   
 #### monkey用法  
  ```
  adb shell monkey [options] <event-count>
  options这个是配置monkey的设置，如指定启动哪个包，不指定将会随机启动所有程序，event-count这个是让monkey发送多少次事件
  例：adb shell monkey -p com.android.test -v 5000  
  向test包发送五千次随机事件
  ```   
  
  ```
  事件  
  
  ```

#### monkey停止条件  

```
如果限定了Monkey运行在一个或几个特定的包上，那么它会监测试图转到其它包的操作，并对其进行阻止。
如果应用程序崩溃或接收到任何失控异常，Monkey将停止并报错。
如果应用程序产生了应用程序不响应(ANR)的错误，Monkey将会停止并报错。
```
