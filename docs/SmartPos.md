# SmartPos  
## **项目依赖关系图**  
![posstructure](https://i.loli.net/2019/04/11/5caf069822108.png)  
## **激活模块**
1. build.gradle里面的depandence中
	* implementation project(':PosSDK')
	* implementation project(':jbig-kit')
	* implementation project(':kprogresshud')
2. MyApplication中初始化
	* CILSDK.setCILAppVersion();
	* CILSDK.setDebug(BuildConfig.DEBUG);
	* CILSDK.setAddress(BuildConfig.ADDRESS);
	* CILSDK.connect(this);
	* 如果还适配了联迪A8，login需要传入activity的context建议在每个可能调用设备刷卡，加密，打印的actiivty调用CILSDK.connect()保证使用过程中设备连接不断开推荐使用activitylifecyclerCallbacks.
3. SplashActivity
	* 设置了activity的进入动画。
	* 下方设置了版本号+背景颜色表示不同环境。
	* 跑了一个handler(根据是否激活和是否下载参数来进入MainActivity还是ActivateActivity)。  
	
4. ActivateActivity
	* 两种激活方式
		* 扫二维码
		* 输入商户号和终端号（要求在服务平台手动录入sn号）  
	两种方式的区别：扫二维码比手输多了个去服务平台请求商户号和终端号的接口
	* 激活的过程（服务平台）
		* 扫二维码：读码成功立马调用getMerchantInfo获取商户信息，点击激活按钮调用activeDevice接口（接口内部调用了downloadParameters终端参数下载接口+notifyActive激活完成）
		* 输入商户号和终端号：active+loadparameters
		* ACTIVE:服务平台激活
		* LOADPARAMETERS:下载交易需要的相关信息；交易Ip和端口，币种，超时时间，权限，报文加密标志位。
	  
5. MainActivity  
	* 检查APP版本  jobScheldure  后台服务进行下载apk
	* 终端密钥下载  
		* 下载RSA->装载RSA->下载主密钥->装载主密钥->启用主密钥->下载工作密钥->装载工作密钥->下载aids->装载aids->下载ICPKS->装载ICPKS  
		* 
	* 签到（更新工作密钥（下载+装载工作密钥）的过程）网关平台要求应用每天签到一次  
		* CILSDK.signIn()  

6.交易  

* 扫码交易
	* 先检查本地缓存的币种与交易报文的币种是否一致  check transe currency
		* 不一致--->结算
		* 一致，继续
	* 最小交易金额Math.pow(10,-2)   0.01    根据交易币种小数位来确定
	* 
* 银行卡交易  

api  
## **收单服务平台**  
1. getMerName   终端注册码获取商户信息接口    
	*  ActiveActivity的onActivityResult中调用
	*  sdk中调用
2. active  获取Authentication Token接口     
	*  旧版本激活方式使用的是devicetoken(推送模块生成)
3. activeCode  新激活接口，使用激活码
	* 激活码方式激活使用的是sn，token传的null  
	
graph TD;  
	A-->B;
	A-->C;
	B-->D;
	C-->D;



## **sdk功能点整理**
**激活模块（在录入终端的时候需要选择方式）  app~收单服务平台**
1. 传统输入商户号和终端号的方式激活（重置的方法是清空sn号然后点击确定）
	* 调用sdk激活接口  
    CILSDK.active(merCode, termCode, callback); 
	* 接口调用成功后立马调用终端参数下载接口下载交易时使用的参数
	CILSDK.downloadParam(mer,term,callback);
	* 跳转到MainActivity onCreat的时候调用终端密钥下载接口
	CILSDK.downloadParamWithProgress(callback);  

汇总：  
	1.sp的抽取  
		&emsp; 传入三个参数put(Context context,String str,Object &emsp; object),根据object的类型，选择putString还是putInt。。。get(...)同上
	2.KProgressHUD  github上第三方包直接引用源代码
	3.抽取CILlog类，只有在debug模式下才输出日志，另使用sd记录一些错误日志
	4.存储sn（会率先使用A没有后然后使用B）
		 &emsp; A.virtualSN：使用激活码激活成功后，服务平台会返回sn，存储到sp
		 &emsp; B.REAL_SN：a8,联迪。。。在设备连接的时候会获取一个REAL_SN  
	5.devicetoken  
		 &emsp; 推送平台使用的设备唯一标识识别，可以通过推送服务的注册获得  
		 &emsp;	旧版本激活前如果devicetoken为空，会尝试进行重新注册一次来获取devicetoken,如果依然为空，传 ""    
	6.versionname
		&emsp;  2.4.0：20190527：SDK	
	7.激活同步、异步-------->是否使用RXjava进行切换的区别  
	8.激活成功后进行参数下载，sn号传的null值，然后通过4.获取真正的sn
	9.参数下载（交易相关的）：流水号，批次号，交易币种，交易ip,商户信息，全报文加密标志位
		
2. 输入/扫描激活码激活设备  
	* 通过打开扫描页面扫激活码，在扫码成功的回调中根据激活码查询商户名等信息  
	CILSDK.getMechantInfo(activeCode,time.appVersion);
	* 点击激活按钮调用激活码激活接口
    CILSDK.activeWithCode(activeCode,deviceToken, callback);   
	* 激活接口调用成功后紧接着调用下载终端参数的接口
	CILSDK.downloadParamWithProgress(callback);
**终端密钥下载**  
终端密钥下载只需要成功执行一次即可，密钥会被转载到POS的硬件模块  
* 下载RSA公钥
* 装载RSA公钥
* 下载主密钥
* 装载主密钥
* 启用主密钥
* 下载工作密钥
* 装载工作密钥
* 下载AIDS
* 装载AIDS
* 下载ICPKS
* 装载ICPKS
* 上送联机版本号
激活完成接口  
CILSDK.notifyActived(mer,term);  

**交易**  
1. 扫码交易（不需要硬件模块读卡器）  
* 扫码消费consume  
request.setAmount()  
request.setScanCode()  
request.setOrderId()  
CILSDK.consumeQr(request,callback); 
在callback中判断如果有应答09/98需要发起查询操作  
CILSDK.QueryQr(request,callback);//查询6次10s不超过60s,查询出结果返回，如果最后一次仍然是09、98sdk会自动发起取消操作   
* 扫码取消/冲正cancel  
CILSDK.sCanCodeCancel(request,callback);
* 扫码撤销revoke
CILSDK.revokConsumerQr(request,callback);  
* 扫码退货refund  
* CILSDK.returnConsumeQr  
2. 刷卡交易区分卡类型（mscCard,icCard ,nfcCard,默认使用mscCard）
通过继承BaseCardActivity复写
getAmount()接口方法   通过跳转的时候传过来 
cardReaderHandler()  读卡信息的接听返回




密码键盘调用
1.在BaseCardActivity的oncreat方法中实例话的类MyPosTransferListener中有个方法监听键盘事件sendPinRequest(String accNo).
2.监听被带进cardEventDelegate--->Pos--->指定的卡事件处理类
3.输入密码时候锁住当前线程
  
CILSDK 类直接对外提供方法的类
SwipCardTrade 类中处理刷卡相关报文的发送
transactionUtils 交易相关的辅助类  
cardEventDelegate 代理卡片事件管理  basecardactyivity调用这个类 

读卡
打开键盘
完成读卡
失败

通过startactivityforResult   返回时判断requestcode来处理unlock线程
每个品牌posreader类都实现接口（openCardread,cancelCardReader）,通过工厂模式创建各品牌的读卡模块


汇总：
1.抽象类实现一个接口(cardHandlerResult)，然后具体的类去继承抽象类（BaseCardActivitivity），便可选择性的复写需要的方法。
 
WithOutCardTrade 两种不需要卡的交易分别是小费和DCC转EDC

* 刷卡消费   
CILSDK.consume(request,callback);
* 刷卡撤销
CILSDK.revokeConsume(request,cardType,Callback);
* 刷卡退货
CILSDK.returnConsume(requst,cardtype,callback);
* 刷卡余额查询
CILSDK.checkBalance(request,cardtype,callback);
* 刷卡预授权
CILSDK.preAuth(request,cardtype,callback);
* 刷卡预授权完成
CILSDK.preAuthComplete(request,cardtype,callback);
* DCC转EDC
CILSDK.dccToEdc(request,callback);  


**账单查询**  
* 获取账单列表  
CILSDK.getBillsAsync(pager,size,txtype,callback);
* 获取账单统计  
CILSDK.getBillStateAsync(type,callback)
* 根据凭证号获取订单详情（获取当前批次下得订单详情，凭证号：）
CILSDK.getBillByTraceNumAsync(trancenum,callback)


**结算**   
每日交易结束得时候或收银员交接班时，对某段时间内得账款核对，商户每日交易结束得时候，收银员需要统计并核对所有得交易，核对交易统计准确后结算，打印出结算单，结算会涉及一个概念 批次号，我们在前面的交易都会传入一个批次号给request,调用结算后，后续的交易需要将这个批次号加一，因为此批次号已经打包结算掉了
CILSDK.transSettleAsync(batchNum,callback)  

**打印**  
* 打印银行卡类交易、扫码类交易  
CILSDK.printKindsReceipts(trans，linebreak,formartTransCode,kind,isForeignTrans,callback);
* 打印结算小票
CILSDK.pintSettleReceipts(transSettles,transDatetime,batchNum,formatTransCode,lineBreak,formatTransCode,callback);
* 自定义打印 (超过2000个字符需要采用分段式打印，否则出现异常DeviceRTException)   
CILSDK.printBufferReceipt(buffer,lineBreak,callback);
* 打印二维码
CILSDK.	printQRCode(qrCode,positon,width,linebreak,callback);
* 打印条形码
CILSDK.printBarCode(barCode,posion,lineBreak,callback);
* 打印图片
CILSDK.printImage(bitmap,lineBreak,offset,callback)  


其他设置  
* 获取sdk版本号
CILSDK.VERSION_NAME;
CILSDK.VERSION_CODE;
* 获取sn号
CILSDK.getDeviceSN();  
* 设置流水号
CILSDK.setSericalNum(int serialNum);
* 获取流水号
CILSDK.getSerialNum();
* 设置批次号
CILSDK.setBatchNum(int batchNum);  
* 获取批次号
 CILSDK.getBatchNum();  
* 设置联迪密钥区(取值范围1~15)  需要在CILSDK.connect之前调用
CILSDK.setTingA8KeyIndex(2);
* 设置密钥索引(MAIN,MAC,PIN,MES)  需要在CILSDK.connect之前调用
CILSDK.setTingKeyIndex(4,101,10,150);

**日志收集（uncaughtExceptionHandler）**  
将日志写到sd文件中，然后上传的时候在从文件中取出上传

1.编译时异常在编码的时候就会提示编码人员进行try catch,运行时异常会出现忘记trycatch的现象，可以通过uncaughtExceptionHandler来进行捕获  
具体用法：继承这个类复写uncaughtException方法，具体的异常会回调到这个方法中（https://www.cnblogs.com/jadic/p/3532580.html）  
捕获后写到本地的sdcard中
2.上传log文件IntentService
     debugInfoActivity右上角的optionsMenu有上传日志的选项
	开启IntentService在handleIntent中执行上传日志操作
    开启线程池一条一条的上传日志SingleThreadExcutor
 
MainActivity-->mainpagerFragment-->DebugInfoActiivty标题栏的头像点击8下，进入调试暗门