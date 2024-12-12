# essay
### 说明  

这是一个用github的pages功能做的网站，开发的初始阶段主要是一个记录的作用，记录自己对安卓的一些知识点的理解，及工作中一些问题的总结。

### 效果预览  
可通过浏览器访问以下地址查看网站的具体内容
 https://gongluis.github.io/essay/
 
### 后续维护方法
在有python pip 和mkdocs的环境下：
1.提交代码到仓库
```
git add .
git commit -m "DESC:部分优化内容xxx"
git push origin master
```
2.静态部署命令
```
mkdocs gh-deploy
```

在没有上述环境下：
> 环境搭建  
 1. python环境（参考廖雪峰网站）  
	* 通过以下命令检查是否具备python环境
```
$ python --version`  
 Python 2.7.2
```
 2. pip环境（pip属于python包里面的工具） 
	* 通过以下命令检查是否具备pip 
```
$pip --version`  
 pip 1.5.2
```
 3. 具体环境搭建参见[http://www.mkdocs.org/](http://www.mkdocs.org/ "mkdocs使用")
 4. 使用pip安装mkdocs  
	`$ pip install mkdocs`