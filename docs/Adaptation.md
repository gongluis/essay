### Android屏幕适配  

### 基础  
##### 单位  
```
1. px(Pixel像素)  
构成图像的基本单元
2. resolution(分辨率)  
屏幕水平方向和垂直方向的像素数量1920*1080
3. Dpi(dot per inch,每英寸像素数)
 dpi = density*160
4. Density
屏幕密度，但是别被四个字误导了，值跟屏幕物理大小一点关系没有  
1080*1920：density为3.0
1080*2160：density为2.75
720*1280：density为2.0
Dip/dp(设备独立像素)  

```  

##### 换算  
```
dp的定义，1dp约等于中等密度屏幕（160dpi，基准密度）上的1像素。
px = density*dp
density = dpi/160
px = dp*(dpi/160)
px = dp*density


px和density是屏幕自带的东西，dpi和dp是通过计算得来的东西
```
##### 屏幕尺寸
1英寸（inch）=2.54厘米（cm）  
一款手机写的屏幕是5.2英寸，表示屏幕对角线长度5.2英寸  
##### 分辨率  
1920px x 1080px意思就是竖向有1920个像素块，横向有1080个像素块  
##### 屏幕像素密度（PPI）  
每英寸屏幕所拥有的像素数，每英寸也是指的对角线  
![ppi.jpg](https://i.loli.net/2020/09/03/1ubfwWQ3lei8U4O.jpg)  

1920*1080最后计算出来的是423.63599近似等于424  
##### 像素的大小是固定的吗？  
借用网上一个例子：  
![iphone6huawei7.jpg](https://i.loli.net/2020/09/03/kWJpLQYU6MDARGd.jpg)  

上面的是iphone6的参数，下面是荣耀7的参数，两者的分辨率都是1920*1080但是，苹果手机的屏幕尺寸比华为7的要小0.2英寸，苹果6手机的ppi却比华为6的高45，说明每英寸的屏幕苹果6用469ppi显示，华为7只用了424个。英寸是物理尺寸大小是固定的，所以可以推断出像素的尺寸大小不是固定的。  

##### 结论  
PPI越高的手机显示效果就越精细

