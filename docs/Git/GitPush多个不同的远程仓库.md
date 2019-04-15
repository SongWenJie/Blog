# git push 多个不同的远程仓库

我们可以将同一份资源提交到不同的远程仓库，例如同时使用 Gitlab 和 Github 进行版本管理。<br />假设我们已经使用 Gitlab 进行版本管理，已经存在和 Gitlab 远程仓库关联好的本地仓库。现在也想使用 Github 进行版本管理，毕竟看着 Github 绿油油的一片，还是非常有成就感的。

假设 Gitlab  远程仓库地址为 [https://gitlab.com/coder/a.git](https://www.yuque.com/songwenjie/kb/vrveow/edit)， Github 远程仓库地址为 [https://github.com/coder/a.git](https://www.yuque.com/songwenjie/kb/vrveow/edit)。

首先要为本地仓库添加一个新的远程仓库：
```git
$ git remote add <主机名> <网址>
```

```git
$ git remote add github https://github.com/coder/a.git
```

然后就可以 push 代码到 Github，需要显式指定远端分支

```git
$ git push github master
```

需要注意的是，只有在 master 分支提交代码， Github 才会[记录你的 Contributions](https://segmentfault.com/a/1190000004318632) 。如果我们是在 dev 分支上进行开发，需要先将本地 dev 分支 merge 到本地 master 分支，然后再 push 到 Github 远程仓库 。具体操作为：<br /><br />
```git
$ git checkout dev  # 切换到dev分支进行开发
...开发代码...
$ git checkout master  # 切换到主分支
$ git merge dev  # 把dev分支的更改和master合并
$ git push github master # 提交主分支代码到Github
$ git checkout dev  # 切换到dev远程分支
$ git push  # 提交dev分支到Gitlab(默认远端仓库无需指定主机名和分支名)
```

这个时候只要进行 Commits 的用户被关联到你的 Github 帐号中（可以在Github 中关联邮箱），Github 就会[记录你的 Contributions](https://segmentfault.com/a/1190000004318632) 。

