依赖模块: pymdownx.betterem(smart_enable=all)

```text
*text* 或 _text_ 斜体

**text** 或 __text__ 粗体

***text*** 或 ___text___ 粗斜体
```

效果

*text* 斜体

**text** 粗体

***text*** 粗斜体

!!! warning "避免冲突"
	这个扩展和markdown.extensions.smartstrong扩展冲突，不能同时载入，前者替代了后者，实现更好的效果。
