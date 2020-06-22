## **黄色**

---

2个模块均提供了相似的效果

**模块1: pymdownx.mark**

```text
==hello world==
```

效果

==hello world==

**模块2: pymdownx.critic**

![](./../../img/hl_critic.png)

效果

{==hello world==}

**但这2个模块也有区别**:

区别1:

- pymdownx.mark渲染出的html是`<p><mark>hello world</mark></p>`
- pymdownx.critic渲染出的html是`<p><mark class="critic">hello world</mark></p>`

区别2:

- pymdownx.mark不支持块背景高亮
- pymdownx.critic支持块背景高亮

	用法:

	![](./../../img/hl_critic_block.png)

	效果:

	{==

	* AAA
		* BBB
	* CCC

	==}

	支持嵌套，详见[黄色区块嵌套高亮代码示例](./../../syntax/nest_yellow_code/)

## **绿色(下划线)**

---

表示插入文字

依赖模块: pymdownx.critic

![](./../../img/hl_critic_underline.png)

效果

Hello, {++my++} world!

支持嵌套，详见[绿色区块嵌套高亮代码示例](./../../syntax/nest_green_code/)

## **红色(横线)**

---

表示删除文字

依赖模块: pymdownx.tidle

![](./../../img/hl_tilde.png)

效果

{--delete me--}

支持嵌套，详见[红色区块嵌套高亮代码示例](./../../syntax/nest_red_code/)

## **绿色+红色(下划线+横线)**

---

表示替换文字

2种实现方式:

- 先使用红色(横线)，再使用绿色(下划线)

	![](./../../img/hl_delete_and_insert.png)

- 使用专门的语法，依赖模块pymdownx.tidle

	![](./../../img/hl_replace.png)

这2种方式效果完全一致，包括实际看到的样式和html都是相同的

Hello, {~~my~>our~~} world!

支持嵌套，详见[绿色+红色区块嵌套高亮代码示例](./../../syntax/nest_greenred_code/)

## **灰色**

---

表示注解

依赖模块: pymdownx.critic

![](./../../img/hl_critic_note.png)

效果

This is a test{>>What is it a test of?<<}.

注意:

1. 只支持单行用法，不支持区块用法
2. 无法与代码高亮结合，包括行内代码高亮和区块代码高亮
