对于版本控制工具来说，尤为重要的就是每次提交版本到代码库的日志撰写。清晰、规范、格式化的提交日志有助于追踪版本修改，查看历史记录等。 Git 不允许提交日志为空，这里推荐使用目前使用最广泛的 angular 规范。



angular 规范的 commit message 包括 3 个部分 ，header 、body 和 footer 。

```
<type> (<scope>) : <subject>
//空一行
<body>
//空一行
<footer>
```

- type 用于说明 commit 的类型，只允许使用下面 7 个标识
  - feat: 新功能（feature）
  - fix: 修补 Bug
  - docs: 文档 （documention）
  - style: 样式 （不影响代码运行的变动）
  - refactor: 重构 （既不是新增功能，也不是修改 Bug 的代码变动）
  - test: 增加测试
  - chore: 构建过程或辅助工具的变动

- scope 用于说明 commit 影响的范围，比如数据层、控制层、视图层等，视项目不同而不同
- subject 是 commit 目的的简短描述，不超过 50 个字符
- body 部分是对本次 commit 的详细描述，可以分成多行
- footer 部分只用于两种情况
  - 不兼容变动时，以 BREAKING CHANGE 开头，后面是对变动的描述以及变动理由和迁移方法
  - 如果当前 commit 针对某个 issue ，那么可以在 footer 部分关闭这个 issue



参考：

《Java工程师修炼之道》