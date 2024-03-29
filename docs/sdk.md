[TOC]
#### 一、初识：
工作中有很大一部分时间是在参与开发或者主导开发一些sdk产品，本篇着重在于记录开发过程总结的一些经验和个人见解，以及遇到的一些常见问题和处理方式。对于sdk的定义可以概括为：“sdk即对于jar so aar 一些方法和逻辑合集，对资源api封装后的产物“，从功能方面细分的话可以分为以下几类：

1. 工具类sdk,常见于大公司内部的公共组件，目的是为了提高不同安卓部门间的工作效率减少造轮子和规范符合公司规定的标准数据处理方式，比如各种FileUtils,ByteUtile,ListUtils,SPUtils的合集等，这类sdk不用考虑功能的多样性及各种设计，只需提供一套标准且稳定的接口能力即可。很多公司公共组件部门会引起其它部门同事的吐槽，原因有三点，一是接口对于通用性的设计不够，不能满足不能部门间的使用；二是不够稳定，经常引起调用方应用闪退；三是调用方提出的不合理的定制化要求。对于一二两点需要开发者做一些提升，不管是从了解实际需求角度、接口设计、还是异常处理等方面，后文回逐步介绍，第三点定制化需求，要尽量的拒绝过度定制化的接口，该种需求不在sdk能提供的通用能力之中，或者实在要做就要想办法尽量规范这些需求。
2. 功能（业务）类sdk,用于提供某一个完整的业务功能类sdk,如短信SMSSDK,支付类sdk,分享类sdk等，多见于公司提供出售的一些有技术优势的软件产品，该类产品功能单一，但是要保证功能的可用性和产品的稳定性，其次就是要突出的与竞品的优势，比如sdk体积小，无侵入性，接入简单，一行代码引入，崩溃率低等。
3. 跨进程通信类sdk，用于某块功能很复杂但是足够独立，可以单独放到另一个应用来做，通过进程间通信与主程序进行通信，比如当下比较热门的地图众包类应用，或者其他一些因为法律法规不能在自己应用中做的一些功能，该类应用要求开发者在主程序中完成服务端和接口端设计开发，然后提供一个包含客户端功能的sdk给另一个程序来集成调用，在保持上面两点需要注意的点外，需要注意下进程间通信的稳定性，断开重连机制，还有各种异常返回设计。

#### 二、交互流程设计

在实际开发sdk前，绘制下交互流程图，不仅可以让自己和团队以及接入方清晰的理解业务交互方式和步骤，更重要的是还可以去查漏补缺，及时的发现流程设计的问题及风险点。

这里举一个自己参与设计开发过的支付sdk例子：

```
需要重点考虑的如下三点：
1. 核心是前端没有绝对的安全
2. 错误的返回  
   错误的返回并不会对业务造成损失，比如支付失败，直接当作失败处理即可。
3. 成功的返回
   成功的返回有时候会造成损失，比如支付后发货行为，因此必须要判断返回的可靠性。
   前端代码，容易被破解者通过hook等非正常手段伪造或者篡改，不能信任前端返回码。
```

所以一般支付类sdk产品基本要包含以下重要流程：

![sdk-safe.jpg](https://i.loli.net/2020/08/25/Tizgt8PYWMeQEVr.png)

#### 三、接口设计-初始化接口

> 初始化的本质是将APP的上下文注入到SDK，使其能通过上下文获取到app的资源和上下文

初始化的方式基本可以概括为以下三种：

1. 自定义Application（已经被废弃）

   ```
   跟调用者自己定义的application冲突，接口反射什么的来处理，没必要  
   
   public class App extends Application {
   
       @SuppressLint("StaticFieldLeak")
       private static Context sContext;
   
       @Override
       public void onCreate() {
           super.onCreate();
           sContext = this;
       }
   
       public static Context getContext() {
           return sContext;
       }
   }
   ```

   早期sdk多采用该种方式，后因其侵入性太强，及多sdk兼容问题已逐渐被替代。

2. 静态方法初始化（最常用）

   ```
   public class MySDK {
   
       private static Context sContext;
   
       private MySDK() {
       }
   
       public static void initSdk(Context context) {
                   //获取ApplicationContext防止内存泄漏
           sContext = context.getApplicationContext();
           initSomething();
   
       }
   
       public static Context getContext() {
           return sContext;
       }
   
       private static void initSomething() {
           //init something
       }
   
   }  
     
     //调用方
   public class App extends Application {
   
       @Override
       public void onCreate() {
           super.onCreate();
   
           //初始化SDK
           MySDK.initSdk(this);
   
       }
   }
   ```

   使用方式最简单，方便，因此为目前最常用的方式。

   

   3. "无侵入"contentProvider初始化方案

      ```
      利用contentProvider获取了app的上下文  
      源码：App的启动过程中加载了provider，并且传了一个Application实例进去，最终在ContentProvider中调用了onCreate()方法。因此，在自定义的ContentProvider中，通过getContext()方法就可以获取到Application的实例了，ContentProvider中的onCreate()方法是先于Application中的onCreate()方法执行的（注意：此时Application对象已经创建）。关于App启动耗时的优化思路，是不是又多了一个关注点？  
      
      1. 自定义用于初始化的provider
      
      public class MySDKInitProvider extends FileProvider{
      
          @Override
          public boolean onCreate() {
              //初始化
              MySDK.initSdk(getContext());
              return super.onCreate();
          }
      } 
      
      在SDK Module下的manifest中注册，编译时将会合并至主工程项目
      
      <provider
          android:name=".MySDKInitProvider"
          android:authorities="${applicationId}.MySDKInitProvider"
          android:exported="false"
          android:grantUriPermissions="true">
          <meta-data
              android:name="android.support.FILE_PROVIDER_PATHS"
              android:resource="@xml/my_sdk_provider_paths" />
      </provider>  
      
      这里需要注意的是Provider的authorities千万别写死，否则两个引入同样SDK的App就无法共存了，这大概是SDK最不该犯的错误之一吧！
      ```

      4. 初始化新姿势-App 谷歌新组件，Startup(alpha的阶段，就不太建议用于生产环境中) 

         

#### 四、接口设计-业务接口  

> 如无必要勿增实体，如无必要勿增依赖  
1. sdk接收什么？输出什么？明确职责
##### （一）、出入参设计
1. map
2. String
3. 实体类 
```
1. 自定义实体类作为出入参在activity间传输，无法直接传播实体，需要经过序列化。
    * 实现serializable接口通过bundle传输
    * 实现parcelabel接口，通过budle传输
2. intent中的bundle是通过Binder机制进行数据传输，有大小限制，建议最大不超过500k越小越好，避免传输图片
```

##### （二）、调用封装设计

如果app需要通过sdk的入口activity进行调用与业务开发，如何做一个优雅的调用封装。  

封装前的调用：
```
//繁琐的调用代码
Bundle bundle = new Bundle();
//这里要暴露或者约定一个静态KEY
bundle.putParcelable(KEY_REQ, request);
Intent intent = new Intent(context, EntryActivity.class);
//屏蔽转场动画，让画面跳转更和谐
intent.setFlags(Intent.FLAG_ACTIVITY_NO_ANIMATION);
intent.putExtras(bundle);
context.startActivity(intent);
```

封装后的调用：
```
//简洁的调用代码
MySDK.launchSdk(context,req);

//将复杂逻辑或细节内置于SDK中，对外提供封装好的静态方法即可
public static void launchSdk(Context context, ReqContent request) {
    Bundle bundle = new Bundle();
    bundle.putParcelable(KEY_REQ, request);
    Intent intent = new Intent(context, EntryActivity.class);
    //屏蔽转场动画，让画面跳转更和谐
    intent.setFlags(Intent.FLAG_ACTIVITY_NO_ANIMATION);
    intent.putExtras(bundle);
    context.startActivity(intent);
}
```

##### （三）、数据返回设计  

调用即响应，不可无响应无回调  
背景：  
```
1. 需要在app与sdk的activity间进行数据传输
2. 调用sdk后，需要执行一段异步逻辑，结束后需要及时关闭
3. 需要在逻辑代码处理过程中随时随地的构造返回参数并返回。
4. sdk目标用户，各类第三方app，需要通用的交互机制，避免引起兼容问题。
```
方案：  
```
startActivityForResult()?理论可行，但不够灵活，弃。
采用EventBus ? 避免引入非必须第三方依赖，暂时放弃。
采用BroadcastReceiver？Android 内置的四大组件之一，为组件间交互而生。而且灵活方便，与入参方法呼应，可以从Intent中传输数据。

采用应用内广播，无进程间通信，效率更高，更安全
```
##### （四）、发送数据  
```
//SDK内部封装的静态方法，提供给SDK内部模块统一调用
public static void sendResult(String token, int code, String message) {

    ResultContent result = new ResultContent();
    result.setToken(token);
    result.setCode(code);
    result.setMessage(message);

    Bundle bundle = new Bundle();
    bundle.putParcelable(KEY_RESULT, result);

    Intent intent = new Intent();
    intent.setAction(MY_SDK_RECEIVER_ACTION);
    intent.putExtras(bundle);
    LocalBroadcastManager.getInstance(MySDK.getContext()).sendBroadcast(intent);
}
```
##### （五）、接收数据  
```

//SDK内部封装的注册方法，提供给APP模块统一调用
public static void registerReceiver(BroadcastReceiver receiver) {
    IntentFilter filter = new IntentFilter();
    filter.addAction(MY_SDK_RECEIVER_ACTION);
    LocalBroadcastManager.getInstance(getContext()).registerReceiver(receiver, filter);
}

//SDK内部封装的反注册方法，提供给APP模块统一调用，如果在onCreate()方法中注册，则在onDestroy()方法中反注册。
public static void unregisterReceiver(BroadcastReceiver receiver) {
    LocalBroadcastManager.getInstance(getContext()).unregisterReceiver(receiver);
}


//APP 中定义的广播接收器，用于接收返回参数
public class MyReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        ResultContent result = MySDKHelper.getResult(intent);
        if (result == null) {
            return;
        }
        int code = result.getCode();
        String message = result.getMessage();
        String token = result.getToken();
        String log = String.format(Locale.CHINA,"返回码: %d 返回信息: %s Token: %s", code, message, token);
        Log.d(TAG, log);
    }
}
```
##### （六）、参数过滤设计  

对入参数据进行过滤，避免无意义的后续业务流程，及时提供调用反馈。   
非空检测，数据类型检测，数据格式检测，自定义注解。  

```
    public static final String PRINTER = "PRINTER";
    public static final String HORN = "HORN";
    public static final String SOUND = "SOUND";
    @StringDef({PRINTER, HORN, SOUND})
    public @interface DeviceType{}
    
    
    public void getDevice( @DeviceType String deviceType){
        
    }
```
 

##### （七）、异常处理

1. 异常概念（throwable为exception和error父类）
2. 异常的层次
    * checked Exception 如IoException要么用try catch捕获要么throw出去，在编码的时候会强提醒
    * unCheckException所有继承自error或者RuntimeException的类如nullPointException
3. APP开发者和sdk开发者看代异常的角度。
```
因为sdk内部的异常中断了sdk内部的流程而不告诉app这显然是不可接受的，处理sdk内部的异常，原则就是不应该内部消化，而应该向外抛出，以同样的异常形式或者自定义返回码。
```

4. Exception or Error Code?
在sdk设计中既可以通过异常告知接入方，也可以通过返回码告知接入方，如何选择？  
场景：  
```
1. sdk入参中需要app传入手机号，并校验参数合法性
2. sdk需要获取app包名/appid进行白名单校验
```

自定义异常是一种比较重的告知方式，意味着中断，返回码是一种比较轻的告知方式。  

在与业务有关的逻辑且涉及用户（非app开发者）可以重试的部分，应该采用返回码，方便后续处理，比如用户输入了不符合手机号码规则的信息，可以通过我们的检查逻辑返回对应的返回码，集成sdk的开发者针对处理即可。  
在与业务无关的逻辑且一旦发生用户无法处理的部分，可以采用抛出自定异常的方式，如sdk对app包名的校验，是一个面向开发者的校验逻辑，坦然让app崩溃，让开发者从错误堆栈中找问题,这就是uncheck exception的使用场景  
开发者未遵循约定造成的无法重试的专有异常  

 

#### 五、个性化配置  
> 使用流式比较优雅的实现sdk的个性化配置。  
sdk配置的本质是为sdk相关功能提供默认配置，并且接收开发者的自定义配置  

```
private void initsdk(){
    LuisSDK.setdebug(true);
    LuisSDK.init(this);
    LuisSDK.setVoice(true); 
}
```

流式api的样例  
```
MySDKConfig.getConfig().setDebug(true).setTimeout(8000L);
```

```
public class LuisSDKConfig{
private static final LuisSDKConfig.Config CONFIG = new LuisSDKConfig.Congi();
    
    public static class Config{
        private Config(){
            
        }
        
        public LuisSDKConfig.Config setDebug(final boolean isDebug) {
            sDebug = isDebug;
            return this;
        }
        
    }
    
    public static boolean isDebug() {
        return sDebug;
    }
    
    public static LuisSDKConfig.Config getConfig() {
        return CONFIG;
    }
    
}


//调用示例
LuisSDKConfig.getConfig().setdebug(true);

```
#### 六、SDK安全与校验  
1. 资源安全，如何保证sdk包的完整性  
    * 在编译的时候aar被合并到apk中，直接校验aar包的摘要不可行，但是可以通过校验aar包中的资源,raw,assets包中文件的摘要，这两个目录下的文件在编译的时候不会被压缩，摘要也不会变。
2. 警惕资源覆盖造成的安全问题Android 会将Library模块中资源与Application模块中的资源进行合并，APP Moduler的资源将会覆盖Library Moduler中的同名资源。  
在对文件或者资源进行命名的时候务必添加唯一性的前缀，或其他唯一性命名方案。  

这一特性也为我们替换SDK资源，进行个性化改造提供了一种思路  
3. 存储安全
    * sp存储，采用xml文件格式来存储文件文件所在目录data/data/shared_prefs如果直接将账号密码或者个人敏感信息存储进去，手机root后可轻松获取，存储的时候简单做一层加密逻辑。
    * so安全存储，不仅把密钥存储与so，还要把加解密逻辑或者传输逻辑放置于so。
4. 传输安全
    安全环境监测
    * vpn
    * root
    * 多开
    * 模拟器环境  
5. 混淆与配置
    * 手动继承
    * SDK内置
```
将sdk中混淆配置也打包进aar中  
指定consumerProguardFiles属性，自定义引入混淆规则，该属性只针对Library有效，对app无效


as4.0新建library会会自动创建出两个***rules.pro文件，其中consumer-rules.pro，自动被defaultConfig下的consumerProguardFiles所引用； proguard-rules.pro，自动被buildType下的proguardFiles引用。
区别：
default下的consumerProguardFiles配置的*.pro文件将会在library mopduler打包成aar,以proguard.txt的形式存在
```

#### 七、SDK打包方法  

##### （一）、library module下的build.gradle模板配置：  
```
//区别于application，这意味着该module将被视为library
apply plugin: 'com.android.library'

//自定义的一个方法，用来获取当前时间
static def releaseTime() {
    return new Date().format("yyyy-MM-dd", TimeZone.getTimeZone("UTC"))
}

android {
    compileSdkVersion 29
    buildToolsVersion "29.0.3"

    defaultConfig {
        minSdkVersion 21
        targetSdkVersion 29
        versionCode 1
        versionName "1.0"
        consumerProguardFiles "proguard-rules.pro"//配置自定义的混淆规则，该字段下的混淆规则将作用于集成该SDK的APP
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
        ndk {
            abiFilters 'armeabi-v7a','arm64-v8a'//显式限定sdk支持的arm架构指令集
        }
    }

    signingConfigs {
        release {
            storeFile file("../key/mylib.jks")//相对路径下的签名文件
            storePassword "Android_2333"
            keyAlias "key-sdk"
            keyPassword "Android_2333"
        }
    }

    buildTypes {
        release {
            minifyEnabled true
            shrinkResources false//sdk module编译时，不支持设置shrinkResources为true
            signingConfig signingConfigs.release
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'//编译本module时参与混淆配置的文件
        }
        debug {
            minifyEnabled false
            shrinkResources false //sdk module编译时，不支持设置shrinkResources为true
            signingConfig signingConfigs.release
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'//编译本module时参与混淆配置的文件
        }
    }

    //自动提取编译生成的aar文件，重命名，最后自动复制到appmodule下的libs目录，方便调试验证。
    libraryVariants.all { variant ->
        if (variant.buildType.name == 'release') {
            variant.assembleProvider.get().doLast {
                variant.outputs.each { output ->
                    def outputFile = output.outputFile
                    if (outputFile != null && outputFile.name.endsWith('release.aar')) {
//                        def fileName = "${project.name}-release-${android.defaultConfig.versionName}-${releaseTime()}"
                        def outputPath = "../mylib/build/aar"
                        def libPath = "../app/libs"
                        copy {
                            from outputFile
                            into outputPath
//                            rename { fileName + ".aar" }
                        }
                        //直接复制到App的libs目录下，方便调试
                        copy {
                            from outputFile
                            into libPath
                        }
                    }
                }
            }
        }
    }
}

dependencies {
    //这句话代表集引入libs目录下的所有jar包
    implementation fileTree(dir: "libs", include: ["*.jar"])
    implementation 'com.android.support:support-v4:28.0.0'
    implementation 'com.android.support:appcompat-v7:28.0.0'
}
```

如何打一个aar包：  
```
双击gradle脚本assembleRelease，执行正式aar编译生成过程;
编译完成，可以看到outputs目录下自动生成了一个aar文件，同时我们的脚本也生效了，将其复制到了指定的目录下（这里可以做一下自动版本命名之类的工作）
```
依赖原则，尽量避免第三方库的依赖  

##### （二）、自定义aar名称

```
//如果是修改apk和jar的名称直接修改后缀即可。
android{
    //...
    android.libraryVariants.all{ variant ->
        variant.outputs.all{
           def fileName = "${project.name}_${buildType.name}_v${defaultConfig.versionName}_${defaultConfig.versionCode}.aar"
           outputFileName = fileName
        }
    }
    //...
}

${project.name}也就是当前module的名字
${buildType.name}就是当前构建的类型，例如 debug 或者 release
${defaultConfig.versionName}和${defaultConfig.versionCode}则分别对应了defaultConfig中的versionName和versionCode
```