#### 一、什么是flutter插件？
学习的过程中主观的将插件主要分为两种：  

* 一、flutter本质是一套ui框架，本身无法提供一些系统能力，比如蓝牙、相机、gps等，如果要在flutter APP中调用这些能力就需要和原生平台进行通信，为此flutter提供了一个平台通道，用于和原生平台通信，这个通道可称为插件。
*  二、也是我此次学习flutter插件的主要原因，公司的短信和秒验sdk给APP提供了便捷的短信登录验证的服务，有android 和IOS两个平台的，但是现在想提供给flutter APP使用，插件化便是其中的方案之一，接入放只需要调用插件对应的api，插件会根据当前所处的平台来调取对应平台的sdk，从而达到效果。

#### 二、怎么实现的可以一套插件适配两个平台
创建flutter插件项目一般都存在两个特殊的文件夹，android与ios 如果需要编写java\kotlin或者Object-C\swift代码，分别在对应的文件夹项目中进行即可，然后通过相应的方法将原生代码中开发的方法映射到dart中。  

[下一页](https://gongluis.github.io/essay/howflutterenvironment/)  


