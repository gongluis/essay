### Android端开发版本管理流程

项目所有分支解释：

1. 本地分支：本地开发的分支，拉取的线上dev分支or定制分支

2. dev分支：主干需求开发分支，dev与master是同步的

3. 定制分支：定制客户的需求开发分支，每一次定制都是从master分支上拉的最新已发布上线的tag代码

4. 定制项目：从标准项目上抽出来的特定项目，如：mini公共库是从标准公共库抽出来的，秒验无ui版本是从标准版本中抽出来的等  

>注意1：不论是“标准项目”还是“定制项目”，都必须包含：dev、master分支，如有定制需求，可拉定制分支

>注意2：如果如果master分支是超过dev分支的，拉取mater分支后是提交不到dev分支（报错），可以远端将mater分支代码合并到dev分支，然后本地再切出dev分支，然后提交到远端dev。
### 客户端开发版本管理流程如下：

注意：针对“主干需求开发”和“定制需求开发”有不同的流程

![image](![client_git_flow.png](https://i.loli.net/2021/06/15/Uu3fGqnSxdMPaNX.png)