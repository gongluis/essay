### 故障复盘记录

#### 新增回调事件引发的空指针异常
> 场景：比如给弹出广告事件增加回调，暴露出去一个setAdvertiseMentListener(AdvertiseMentListener);方法。然后在sdk对外暴露的接口类中，新增私有set和get方法，然后再sdk内部合适的地方来调用get方法将事件回调出去，

复现：用户如果没有调用设置监听的方法，然后sdk内部回调事件触发的时候去调用get方法来获取监听进行回调会触发空指针。

