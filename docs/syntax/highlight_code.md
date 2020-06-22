## **行内代码高亮**

---

依赖模块: pymdownx.inlinehilite

使用shebang可以在一行文本里实现代码高亮

```text
`#!python print "Hello, world!"`或`:::python print "Hello, world!"`
```

效果

`#!python print "Hello, world!"`或`:::python print "Hello, world!"`

!!! warning "开头不能有空格，如果有空格的话，就会被认为是正文"
    ```text
    ` #!python print "Hello, world!"`
    ```
	效果
	` #!python print "Hello, world!"`

	```text
	` :::python print "Hello, world!"`
	```
	效果
	` :::python print "Hello, world!"`

## **区块代码高亮**

---

### **用法**

---

依赖模块: codehilite

写法1: 3个\`

    ```python
	text = "Hello, world!"
	print text
    ```

效果

```python
text = "Hello, world!"
print text
```

写法2: 4个空格+shebang
```text
    #!/usr/bin/python
	text = "Hello, world!"
	print text
```

效果

	#!/usr/bin/python
	text = "Hello, world!"
	print text

!!! note "支持434种格式"
    codehilite是基于pygments实现高亮的，支持434种格式，详见[支持代码高亮的语言](./../../appendix/pygments/)

    若指定了不存在的格式，则等同于text，不做任何高亮渲染

若不想使用高亮，用text，如：

    ```text
	text = "Hello, world!"
	print text
    ```

效果

```text
text = "Hello, world!"
print text
```

### **参数1. 自动猜测语言**

---

在mkdocs.yml里markdown_extensions部分设置如下

- \- codehilite(guess_lang=false) 关闭猜测，若不指定语言，则关闭高亮
- \- codehilite(guess_lang=true) 开启猜测，若不指定语言，则自动猜测

### **参数2. 显示行号**

---

在mkdocs.yml里markdown_extensions部分设置

- \- codehilite(linenums=true) 显示行号
- \- codehilite(linenums=false) 不显示行号

!!! warning ""
    若codehilite(linenums=true)，4个空格的代码块方式也会被自动加上行号

!!! warning "指定代码块显示行号"
	**因此，如果想全局默认不显示行号，而指定代码块显示行号，可以在mkdocs.yml里指定linenums=false，然后想显示行号的代码块用linenums="1"**

### **参数3. 指定第几行背景高亮**

---

在mkdocs.yml里不用额外设置

    ```python hl_lines="2 4"
	text1 = "Hello, "
	text2 = "world!"
	print text1 + text2
    ```

效果

```python hl_lines="2 4"
text1 = "Hello, "
text2 = "world!"
print text1 + text2
```

### **参数4. 指定行号从多少开始编号**

---

    ```python linenums="2"
	text1 = "Hello, "
	text2 = "world!"
	print text1 + text2
    ```

效果

```python linenums="2"
text1 = "Hello, "
text2 = "world!"
print text1 + text2
```

!!! warning "强制显式行号"
    当代码块指定了linenums后，即使mkdocs.yml里linenums=false，也会自动显示行号

	**因此，如果想全局默认不显示行号，而指定代码块显示行号，可以在mkdocs.yml里指定linenums=false，然后想显示行号的代码块用linenums="1"**

注意：代码块linenums参数不会影响hl_lines，即无论linenums指定从多少行开始编号，hl_lines实际都以行号从1开始编号来找到对应行进行背景高亮渲染，如：

    ```python linenums="2" hl_lines="3"
	text1 = "Hello, "
	text2 = "world!"
	print text1 + text2
    ```

效果

```python linenums="2" hl_lines="3"
text1 = "Hello, "
text2 = "world!"
print text1 + text2
```
