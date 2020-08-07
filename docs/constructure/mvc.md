
## MVC
```
View：XML布局文件。
Model：实体模型（数据的获取、存储、数据状态变化）。
Controller：对应于Activity，处理数据、业务和UI。
```

从上面这个结构来看，Android本身的设计还是符合MVC架构的，但是Android中纯粹作为View的XML视图功能太弱，我们大量处理View的逻辑只能写在Activity中，这样Activity就充当了View和Controller两个角色，直接导致Activity中的代码大爆炸。相信大多数Android开发者都遇到过一个Acitivty数以千行的代码情况吧！所以，更贴切的说法是，这个MVC结构最终其实只是一个Model-View（Activity:View&Controller）的结构。





著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
这是苹果开发者文档中摘过来的图片，表明了三者之间的关系，简单描述了三者作用Model：数据模型，用来存储数据View：视图界面，用来展示UI界面和响应用户交互Controller：控制器(大管家角色)，监听模型数据的改变和控制视图行为、处理用户交互他们工作和关系看起来是如此清晰，是一种非常好的设计思想，是的，首先声明MVC是一个非常好的架构思想。

