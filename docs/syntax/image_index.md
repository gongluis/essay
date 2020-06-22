当一个页面里多次调用相同图片时，这种方法更适用:

```text
![alt][索引]

[索引]: 图片地址 "title"
```

注意: `[索引]: 图片地址 "title"`可以写在任意地方，通常习惯于放在markdown本页文档最下方

如

```text
![alt][test icon]

...

[test icon]: /img/test_icon.png "title"
```

效果

![alt][test icon]

[test icon]: /img/test_icon.png "title"

鼠标悬停在图标上可以看到"title"字样
