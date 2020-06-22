## **什么是emoji**

---

emoji是日本在无线通信中所使用的视觉情感符号，已被unicode采纳，当前unicode包含了非常多的emoji图标，每一次unicode更新都会发布新的emoji图标。只要操作系统支持unicode，就能在任意使用unicode编码的地方看到漂亮的彩色图标，比如文本、mysql表里、网页。

## **emoji现状**

---

虽然unicode官网可以查到所有emoji图标，但并未提供一个便利的图标分类查询和使用方法。因此就有一些第三方emoji厂商将部分emoji图标，制作成不同格式、不同尺寸(例如64x64 PNG、128x128 PNG或SVG等格式)的图片放入自己的CDN库。然后将这些CDN图片以免费或付费方式对外提供。

## **如何获得emoji**

---

根据使用需求分为2种:

- 不满足于unicode的emoji图标，希望更好看，并且尺寸可选，那么就使用第三方emoji厂商库，推荐使用EmojiOne的png图库，分为免费和付费，其中免费的图库已足够满足一般性使用需求，如果想使用svg，免费的只有EmojiOne-2.2.7以内版本，最新的svg都是要付费的。使用方法是在文档里用 *:短名* 来让mkdocs自动匹配并渲染为<img>标签

	??? note "举例说明2.2.7与高版本区别"
		如果用的 *:短名:* 是2.2.7包含的，那么图标能正常显示，否则显示不出来

		比如：

		https://cdn.jsdelivr.net/emojione/assets/svg/1f600.svg 这个是2.2.7就有的，访问这个url可以看到图标

		https://cdn.jsdelivr.net/emojione/assets/svg/1f92e.svg 这个是2.2.7之后才有的，需要付费才能获得图标，未付费直接访问会得到报错：

		![](./../../img/emoji_denied.png)

		https://cdn.jsdelivr.net/emojione/assets/3.1/png/64/1f92e.png 这个是png，不属于付费部分，访问这个url可以看到图标

		因此推荐的最佳入门实践应当是首选png。

- 能够接受unicode的emoji图标，就简单了，直接将图标复制粘贴到文本里即可

这2种方法并不冲突，可以在一个markdown文本里混合使用
