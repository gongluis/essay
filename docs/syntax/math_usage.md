行内写法有2种: `$...$` 和 `\(...\)`

区块写法有3种: `$$...$$`, `\[...\]`, 和 `\begin{}...\end{}`

具体用法详见:

- https://math.meta.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference

- http://www.andy-roberts.net/writing/latex/mathematics_1

- http://www.andy-roberts.net/writing/latex/mathematics_2

此处仅举个例子(因为begin end方式我还不够了解)：

```text
$$
p(x|y) = \frac{p(y|x)p(x)}{p(y)}
$$
```

效果

$$
p(x|y) = \frac{p(y|x)p(x)}{p(y)}
$$

```text
$p(x|y) = \frac{p(y|x)p(x)}{p(y)}$
```

效果

$p(x|y) = \frac{p(y|x)p(x)}{p(y)}$
