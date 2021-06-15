#### 目的
1. 可以快速了解每个commit的目的，也能方便查找想要的信息
2. 代码bug链接能让能Code Reviewer和后面的人看懂代码修改
3. 标准化log有利于项目交接，同时也方便新人了解项目历史
4. 可以通过log自动生成change log  


#### 规则
```
<type>(<scope>): <version>: <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<extend>
```

```
1. type定义
type代表了此次提交的目的，根据目的的不同可以详细的区分：

feat：新功能
fix：修复bug
docs: 仅仅修改了文档，如 README等
style: 代码格式的修改，不影响代码的功能
refactor: 代码重构，没有加新功能或者修复bug
perf: 性能优化方面的改动，可以等同于refactor
test: 对测试用例模块的改动，包括单元测试、集成测试等
chore: 改变构建流程、或者增加依赖库、工具等
revert: 回滚版本
2. scope定义
可选部分，用于说明 commit 影响的范围。如只影响push socketservice或adpush。尽量能确定scope范围就尽量填上，并且越小越好。

3. version定义
就是这个commit是在哪个版本下进行的修改。如1.3.1。

4. subject定义
是本次commit目的的简短描述。在能描述清楚此次提交目的的前提下用尽可能少的文字，最好不要超过30个汉字。

5. body定义
可选部分，对本地提交更详细的描述。

6. extend定义
可选部分。对于fix等有jira的单子的，可以将单子的链接贴在这里。
```

例子：  
```
feat例子
feat($browser): onUrlChange event (popstate/hashchange/polling)

Added new event to $browser:
- forward popstate event if available
- forward hashchange event if popstate not available
- do polling when neither popstate nor hashchange available

Feature #392
fix的例子
fix(adpush): 1.3.1: couple of unit tests for IE9

Older IEs serialize html uppercased, but IE9 does not...
Would be better to expect case insensitive, unfortunately jasmine does
not allow to user regexps for throw expectations.

Fixed #392
doc的例子
docs(guide): 1.3.1: updated fixed docs from Google Docs

Couple of typos fixed:
- indentation
- batchLogbatchLog -> batchLog
- start periodic checking
- missing brace
```

#### 生成changelog
脚本步骤：

npm install -g conventional-changelog-cli
cd ${project}
conventional-changelog -p angular -i CHANGELOG.md -w -r 0 > CHANGELOG.md