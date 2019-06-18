# **云收银APP阅读文档**
***SplashActicity***  

1.initUmeng+initSunmiBlePrint  
2.延时3s一个handler来决定接下来进入哪个页面（括号里面是条件）  
&emsp;进入GuidActivity(判断sp版本号跟配置文件版本号是否一致，进入登陆页面时保存版本号)  
&emsp;进入MainActivity(通过logintoken判断是否进入MainActivity)  
&emsp;进入LoginAcitivity(else进入LoginActivity)  
3.开启日志上传Service   
 
***GuidActivity***  
&emsp;一个简单滑动的轮播图外加一个进入LoginActivity的按钮  

***LoginActivity***   
&emsp;自定义EditText(主要功能有：清除图标，EditText抖动动画)  
&emsp;重置密码  
&emsp;登陆（包含获取公钥接口+登陆接口）  
&emsp;&emsp;1.对于登陆参数pwd字段的加密（RSA(pwd,publicKey)）  
&emsp;&emsp;2.自定义Toast(YellowToastUtils密码错误顶部显示一段黄色背景提示)  
&emsp;&emsp;3.使用Linerlayout+RelativeLayout  framlayout+权重布局法  
&emsp;4.
