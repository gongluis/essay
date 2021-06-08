### 测试在gitignore添加前就提交的项目无法再过滤的问题
```
1.首先为避免冲突需要先同步下远程仓库，status为未add的红色也不要紧
git pull
2.再本地项目目录下删除缓存,只是清楚缓存不用担心
git rm -r --cached .
3.新建.gitignore文件放到.git相同目录中，该步骤放在1\2步骤前也没事，在add前应该都行
4.再次add所有文件
git add .
5.再次添加commit
git commit -m 'add ignore'
6.最后提交到远程的仓库即可
git push origin master
7.线上可看到在添加.gitignore之前但是在ignore中的文件也会不见了
```