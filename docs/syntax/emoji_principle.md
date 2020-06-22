本文中主要介绍mkdocs依赖pymdownx.emoji进行匹配、渲染，因此下面的说明均是关于pymdownx.emoji。

pymdownx支持3个主流的emoji厂商：Gemoji（github.com做的）、EmojiOne（第三方的，分为免费版、付费版、战略合作版）、Twemoji（twitter做的），pymdownx官方推荐用EmojiOne，默认也是EmojiOne

## **有2个关键参数**

---

1. emoji_index，选择emoji厂商，默认为EmojiOne

2. emoji_generator，选择厂商对应的图片格式，默认为to_png

## **generator有3类**

---

**to_png/to_svg**:

1. markdown在渲染时候，其中的 *:短名:* 根据emoji_index(即Gemoji/EmojiOne/Twemoji)被映射为相应的unicode码
2. unicode码被转为emoji_index对应的图片CDN地址(Gemoji/EmojiOne/Twemoji都有各自的CDN)
3. 渲染成html时候图片地址被写入<img>标签

**to_png_sprite/to_svg_sprite**: 暂不了解

**to_alt**:

1. markdown在渲染时候，其中的 *:短名:* 根据emoji_index(即Gemoji/EmojiOne/Twemoji)被映射为相应的unicode码
2. 渲染成html时候unicode码被直接写入<img>标签

!!! note ""
	to_alt效果等同于直接将emoji图标复制到markdown文档中
