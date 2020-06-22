依赖模块: footnotes

第一步. 插入引用

    Lorem ipsum[^1] dolor sit amet, consectetur adipiscing elit.[^2]

第二步. 插入内容

    [^1]: Lorem ipsum dolor sit amet, consectetur adipiscing elit.
    [^2]:
        Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
        nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
        massa, nec semper lorem quam in massa.

!!! note ""
    可以看到，上面脚注1内容是单行方式，脚注2内容是多行方式

效果

Lorem ipsum[^1] dolor sit amet, consectetur adipiscing elit.[^2]

[^1]: Lorem ipsum dolor sit amet, consectetur adipiscing elit.
[^2]:
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
    nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
    massa, nec semper lorem quam in massa.

!!! warning "注意"
    1. 不一定要用\[\^1\]，也可以用自定义关键字，比如\[\^mykey\]，但这样下面也要改为\[\^mykey\]，也可以支持空格，比如\[\^my key\]（当然后面也要\[\^my key\]:

    2. 无论脚注标签写的是什么（即\[\^1\]或者\[\^mykey\]，在展示脚注标签和展示脚注内容时候都是自动用数字从1开始累加标记，而不是自己写的内容）

    3. \[\^1\]: 脚注内容无论写在.md中的任何位置，在展示时候都是自动位于文档的最下方，并且会自动添加---这样一个横线
