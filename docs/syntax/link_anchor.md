英文叫Permalinks，是toc扩展模块的一个功能

依赖模块: toc

默认值: toc(permalink=false)

笔者推荐值: toc(permalink=true)

参数设置:

- toc(permalink=true)

点击锚点或者跳转至锚点后会自动在锚点处显示图标或文本，并自动固定在该位置，刷新页面后也会自动回到该位置，效果如下：

![](./../../img/permalinks_true.png)

- toc(permalink=false)

看不到锚点处的图标或文本，但跳转至锚点处后自动固定在该位置，刷新页面后也会自动回到该位置
toc(permalink=false)等同于注释掉这个扩展，效果均如下：

![](./../../img/permalinks_false.png)

- toc(permalink=自定义名)

可以支持自定义名，比如toc(permalinks=hello)，效果如下：

![](./../../img/permalinks_hello.png)
