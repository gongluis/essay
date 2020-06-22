## **行内式**

---

```text
[example](http://www.example.com/ "title")
```

效果

[example](http://www.example.com/ "title")

鼠标悬停在链接上可以看到"title"字样

## **参考式**

---

当一个页面里多次调用相同链接时，这种方法更适用:

```
[example][索引]

...

[索引]: http://www.example.com/ "title"
```

注意: `[索引]: http://www.example.com/ "title"`可以写在任意地方，通常习惯于放在markdown本页文档最下方

效果

[example][索引]

[索引]: http://www.example.com/ "title"

鼠标悬停在链接上可以看到"title"字样
