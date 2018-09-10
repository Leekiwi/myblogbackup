---
title: Git简记
date: 2018-07-27 21:04:26
tags: git
categories: git
---

## Git 食用手册

### 0. 前言

最近有个项目比较赶，于是决定 8 个人一个并行完成，单独把项目拿出来用 gitea 管理，没人分一个模块开发，对应也给分支，效果不过，从中也帮助自己重新温习了一下 git 的使用，小记一下。

<!-- more -->

### 1. 安装和使用

- 在 MAC 上，安装 homebrew，然后通过[homebrew](https://brew.sh/index_zh-cn) 安装 Git。

  > 在 MAC 上另一种安装方法，从 AppStore 安装 Xcode，Xcode 集成了 Git，不过默认没有安装，你需要运行 Xcode，选择菜单“Xcode”->“Preferences”，在弹出窗口中找到“Downloads”，选择“Command Line Tools”，点“Install”就可以完成安装了。

- 在 Windows 上，下载[安装包](http://rj.baidu.com/soft/detail/40642.html)，默认下一步，安装完成即可。

- 安装完后自报家门

  ```
  $ git config --global user.name "wuwhs"
  $ git config --global user.email "email@example.com"
  ```

- 创建版本库
  初始化一个 Git 仓库，使用`git init`命令。添加文件到 Git 仓库，分两步：
  1. 第一步，使用命令`git add <file>`，注意，可反复多次使用，添加多个文件；
  2. 第二步，使用命令`git commit`，完成。

### 2. 时光穿梭

![](https://note.youdao.com/yws/public/resource/813e8f68e489060d70ccfdff42b3aecc/xmlnote/8849F03FF3934C8087DA9645B9BD9C5F/14633)

- 如果`git status`告诉你有文件被修改过，用`git diff`可以查看修改内容。

- HEAD 指向的版本就是当前版本，因此，Git 允许我们在版本的历史之间穿梭，使用命令`git reset --hard commit_id`。

- 穿梭前，用`git log`可以查看提交历史，以便确定要回退到哪个版本。

- 要重返未来，用`git reflog`查看命令历史，以便确定要回到未来的哪个版本，`git log --pretty=oneline --abbrev-commit`在一行显示缩写提交号。

- 场景 1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。

- 当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令`git reset HEAD file`，就回到了场景 1，第二步按场景 1 操作。

- 命令`git rm`用于删除一个文件。如果一个文件已经被提交到版本库，那么你永远不用担心误删，但是要小心，你只能恢复文件到最新版本，你会丢失最近一次提交后你修改的内容。

### 3. 远程仓库

- 创建 SSH Key。`$ ssh-keygen -t rsa -C "youremail@example.com"`。

- 登陆 GitHub，打开“Account settings”，“SSH Keys”页面。
  然后，点“Add SSH Key”，填上任意 Title，在 Key 文本框里粘贴 id_rsa.pub 文件的内容。

- 要关联一个远程库，使用命令`git remote add origin git@server-name:path/repo-name.git`。

- 关联后，使用命令`git push -u origin master`第一次推送 master 分支的所有内容。

- 此后，每次本地提交后，只要有必要，就可以使用命令`git push origin master`推送最新修改。

- 要克隆一个仓库，首先必须知道仓库的地址，然后使用`git clone`命令克隆。

### 4. 分支管理

- Git 鼓励大量使用分支。

- 查看分支：`git branch`。

- 创建分支：`git branch <name>`。

- 切换分支：`git checkout <name>`。

- 创建+切换到当前分支：`git checkout -b <name>`。

- 合并某分支到当前分支：`git merge <name>`。

- 删除分支：`git branch -d <name>`。

- 当 Git 无法自动合并分支时，就必须首先剞劂冲突，解决冲突后，再提交，合并完成用`git log --graph`命令可以看到分支合并图。

- 合并分支时，加上`--no-ff`参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而`fash-forward`合并就看不出来曾经做过合并。

- 当手头工作没有完成时，先把工作现场`git stash`一下，然后去修复 bug，修复后，再`git stash list`查看历史 stash，一是用`git stash apply`恢复，但恢复后，stash 内容并不删除，你需要用`git stash drop`来删除；另一种方式是用`git stash pop`，恢复的同时把 stash 内容也删了。

- 查看远程库信息，使用`git remote -v`。

- 从本地推送分支，使用`git push origin branch-name`，如果失败，先用`git pull`抓取远程的新提交。

- 再本地创建和远程分支对应的分支，使用`git checkout -b branch-name origin/branch-name`，本地和远程分支的名称最好一致。

- 从远程抓取分支，使用`git pull`，如果有冲突，要先处理冲突。

### 5. 标签

- 命令`git tag <name>`用于新建一个标签，默认为`HEAD`,也可以指定一个 commit id。

- `git tag -a <tagname> -m "balabala..."`可以指定标签信息。

- `git tag -s <tagname> -m "balabala..."`可以用 PGP 签名标签。

- 命令`git tag`可以查看所有标签。

- 命令`git push origin <tagname>`可以推送一个本地标签。

- 命令`git push origin --tags`可以推送全部未推送过的本地标签。

- 命令`git tag -d <tagname>`可以删除一个本地标签。

- 命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签。

### 6. 举个应用栗子

1. 最初在远程创建项目仓库有`master`和`develp`分支，参与开发人员先在自己一个文件夹下，调出`git Bash`，然后输入命令`git init`，再把仓库`git clone`下来

```
MINGW32 /d/appSoft/wampserver/wamp64/www
$ git init
Initialized empty Git repository in D:/appSoft/wampserver/wamp64/www/.git/

MINGW32 /d/appSoft/wampserver/wamp64/www (master)
$ git clone git@github.com:wuwhs/demo.git
Cloning into 'demo'...
Warning: Permanently added the RSA host key for IP address '13.229.188.59' to the list of known hosts.
warning: You appear to have cloned an empty repository.
Checking connectivity... done.
```

2. `cd demo`进入`clone`下载的目录里，用`git branch develop`在本地创建一个对应的`develop`分支

```
MINGW32 /d/appSoft/wampserver/wamp64/www (master)
$ cd demo

MINGW32 /d/appSoft/wampserver/wamp64/www/demo (master)
$ git branch
* master

MINGW32 /d/appSoft/wampserver/wamp64/www/demo (master)
$ git branch develop

MINGW32 /d/appSoft/wampserver/wamp64/www/demo (master)
$ git branch
  develop
* master
```

再次用`git branch`查看已经新建了一个`develop`分支

3. `git checkout develop`切换到当前`develop`分支

```
MINGW32 /d/appSoft/wampserver/wamp64/www/demo (master)
$ git checkout develop
Switched to branch 'develop'
```

4. 用`git pull origin develop:develop`，即：`git pull origin <远程主机名> <远程分支名>:<本地分支名>`，当本地和远程分支名相同时，可以简写成一个，也就是`git pull origin develop`，拉取远程`develop`分支完成，然后开发人员就可以在这个分支上工作了

```
MINGW32 /d/appSoft/wampserver/wamp64/www/demo (develop)
$ git pull origin develop:develop
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
From github.com:wuwhs/demo
   7ff2cb0..7ab2842  develop    -> develop
   7ff2cb0..7ab2842  develop    -> origin/develop
warning: fetch updated the current branch head.
fast-forwarding your working tree from
commit 7ff2cb0627be357fa15db4e38e1bfe8fc820b8ec.
Already up-to-date.
```

5. 当一天了工作完成，要提交到远程分支，首先要拉取一下别人提交的代码，防止版本冲突

```
MINGW32 /d/appSoft/wampserver/wamp64/www/demo (develop)
$ git pull
remote: Counting objects: 3, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
From github.com:wuwhs/demo
   f848dc7..d696375  develop    -> origin/develop
Updating f848dc7..d696375
Fast-forward
 demo.txt | 2 ++
 1 file changed, 2 insertions(+)
```

PS：直接偷懒`pull`可能会出现没有找到`tracking`的分支

```
MINGW32 /d/appSoft/wampserver/wamp64/www/demo (develop)
$ git pull
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details.

git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

git branch --set-upstream-to=origin/<branch> develop
```

这时候要手动添加一下对应分支依赖`git branch --set-upstream-to=origin/<branch> develop`

```
 MINGW32 /d/appSoft/wampserver/wamp64/www/demo (develop)
$ git branch --set-upstream-to=origin/develop develop
Branch develop set up to track remote branch develop from origin.

MINGW32 /d/appSoft/wampserver/wamp64/www/demo (develop)
$ git pull
remote: Counting objects: 3, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
From github.com:wuwhs/demo
   f848dc7..d696375  develop    -> origin/develop
Updating f848dc7..d696375
Fast-forward
 demo.txt | 2 ++
 1 file changed, 2 insertions(+)
```

6. 将本地分支提交到对应远程分支上，`git push origin develop:develop`，即：`git push origin <远程主机><本地分支名>:<远程分支名>`，如果名称一样可以简写，也就是`git push origin develop`

```
MINGW32 /d/appSoft/wampserver/wamp64/www/demo (develop)
$ git push origin develop:develop
Counting objects: 9, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (9/9), 759 bytes | 0 bytes/s, done.
Total 9 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), done.
To git@github.com:wuwhs/demo.git
   d696375..3c00c0c  develop -> develop
```

7. 项目测试 OK 了，本地分支合并到`master`分支上，要用到`git merge <branch>`

```
MINGW32 /d/appSoft/wampserver/wamp64/www/demo (develop)
$ git checkout master
Switched to branch 'master'

MINGW32 /d/appSoft/wampserver/wamp64/www/demo (master)
$ git merge develop
Updating c4d0377..3c00c0c
Fast-forward
 demo.txt | 9 +++++++++
 1 file changed, 9 insertions(+)
```

常用的操作就以上七步了，当然会有不同情形的应用。

### 7. 附录：git-cheat-sheet

一般而言，常用的就是以上那些命令，有人专门的整理了一份比较全一点的文档 git-cheat-sheet，方便查阅。

#### 配置

列出当前配置：

```
$ git config --list
```

列出 repository 配置：

```
$ git config --local --list
```

列出全局配置：

```
$ git config --global --list
```

列出系统配置：

```
$ git config --system --list
```

设置用户名：

```
$ git config --global user.name "[firstname lastname]"
```

设置用户邮箱：

```
$ git config --global user.email "[valid-email]"
```

设置 git 命令输出为彩色：

```
$ git config --global color.ui auto
```

设置 git 使用的文本编辑器设：

```
$ git config --global core.editor vi
```

#### 配置文件

Repository 配置对应的配置文件路径[--local]：

```
<repo>/.git/config
```

用户全局配置对应的配置文件路径[--global]：

```
~/.gitconfig
```

系统配置对应的配置文件路径[--local]：

```
/etc/gitconfig
```

#### 创建

复制一个已创建的仓库:

```
# 通过 SSH
$ git clone ssh://user@domain.com/repo.git
```

```
#通过 HTTP
$ git clone http://domain.com/user/repo.git
```

创建一个新的本地仓库:

```
$ git init
```

#### 本地修改

显示工作路径下已修改的文件：

```
$ git status
```

显示与上次提交版本文件的不同：

```
$ git diff
```

把当前所有修改添加到下次提交中：

```
$ git add .
```

把对某个文件的修改添加到下次提交中：

```
$ git add -p <file>
```

提交本地的所有修改：

```
$ git commit -a
```

提交之前已标记的变化：

```
$ git commit
```

附加消息提交：

```
$ git commit -m 'message here'
```

提交，并将提交时间设置为之前的某个日期:

```
git commit --date="`date --date='n day ago'`" -am "Commit Message"
```

修改上次提交
（请勿修改已发布的提交记录!）

```
$ git commit --amend
```

修改上次提交的 committer date：

```
GIT_COMMITTER_DATE="date" git commit --amend
```

修改上次提交的 author date：

```
git commit --amend --date="date"
```

把当前分支中未提交的修改移动到其他分支：

```
git stash
git checkout branch2
git stash pop
```

将 stashed changes 应用到当前分支：

```
git stash apply
```

删除最新一次的 stashed changes：

```
git stash drop
```

#### 搜索

从当前目录的所有文件中查找文本内容：

```
$ git grep "Hello"
```

在某一版本中搜索文本：

```
$ git grep "Hello" v2.5
```

#### 提交历史

从最新提交开始，显示所有的提交记录（显示 hash， 作者信息，提交的标题和时间）：

```
$ git log
```

显示所有提交（仅显示提交的 hash 和 message）：

```
$ git log --oneline
```

显示某个用户的所有提交：

```
$ git log --author="username"
```

显示某个文件的所有修改：

```
$ git log -p <file>
```

仅显示远端分支与远端分支提交记录的差集：

```
$ git log --oneline <origin/master>..<remote/master> --left-right
```

谁，在什么时间，修改了文件的什么内容：

```
$ git blame <file>
```

显示 reflog：

```
$ git reflog show
```

删除 reflog：

```
$ git reflog delete
```

#### 分支与标签

列出所有的分支：

```
$ git branch
```

列出所有的远端分支：

```
$ git branch -r
```

切换分支：

```
$ git checkout <branch>
```

创建并切换到新分支:

```
$ git checkout -b <branch>
```

基于当前分支创建新分支：

```
$ git branch <new-branch>
```

基于远程分支创建新的可追溯的分支：

```
$ git branch --track <new-branch> <remote-branch>
```

删除本地分支:

```
$ git branch -d <branch>
```

强制删除一个本地分支：
将会丢失未合并的修改！

```
$ git branch -D <branch>
```

给当前版本打标签：

```
$ git tag <tag-name>
```

给当前版本打标签并附加消息：

```
$ git tag -a <tag-name>
```

#### 更新与发布

列出当前配置的远程端：

```
$ git remote -v
```

显示远程端的信息：

```
$ git remote show <remote>
```

添加新的远程端：

```
$ git remote add <remote> <url>
```

下载远程端版本，但不合并到 HEAD 中：

```
$ git fetch <remote>
```

下载远程端版本，并自动与 HEAD 版本合并：

```
$ git remote pull <remote> <url>
```

将远程端版本合并到本地版本中：

```
$ git pull origin master
```

以 rebase 方式将远端分支与本地合并：

```
git pull --rebase <remote> <branch>
```

将本地版本发布到远程端：

```
$ git push remote <remote> <branch>
```

删除远程端分支：

```
$ git push <remote> :<branch> (since Git v1.5.0)
# or
git push <remote> --delete <branch> (since Git v1.7.0)
```

发布标签:

```
$ git push --tags
```

合并与重置(Rebase)
将分支合并到当前 HEAD 中：

```
$ git merge <branch>
```

将当前 HEAD 版本重置到分支中:
请勿重置已发布的提交!

```
$ git rebase <branch>
```

退出重置:

```
$ git rebase --abort
```

解决冲突后继续重置：

```
$ git rebase --continue
```

使用配置好的 merge tool 解决冲突：

```
$ git mergetool
```

在编辑器中手动解决冲突后，标记文件为已解决冲突：

```
$ git add <resolved-file>
$ git rm <resolved-file>
```

合并提交：

```
$ git rebase -i <commit-just-before-first>
```

把上面的内容替换为下面的内容：

原内容：

```
pick <commit_id>
pick <commit_id2>
pick <commit_id3>
```

替换为：

```
pick <commit_id>
squash <commit_id2>
squash <commit_id3>
```

#### 撤销

放弃工作目录下的所有修改：

```
$ git reset --hard HEAD
```

移除缓存区的所有文件（i.e. 撤销上次 git add）:

```
$ git reset HEAD
```

放弃某个文件的所有本地修改：

```
$ git checkout HEAD <file>
```

重置一个提交（通过创建一个截然不同的新提交）

```
$ git revert <commit>
```

将 HEAD 重置到指定的版本，并抛弃该版本之后的所有修改：

```
$ git reset --hard <commit>
```

用远端分支强制覆盖本地分支：

```
git reset --hard <remote/branch> e.g., upstream/master, origin/my-feature
```

将 HEAD 重置到上一次提交的版本，并将之后的修改标记为未添加到缓存区的修改：

```
$ git reset <commit>
```

将 HEAD 重置到上一次提交的版本，并保留未提交的本地修改：

```
$ git reset --keep <commit>
```

删除添加.gitignore 文件前错误提交的文件：

```
$ git rm -r --cached .
$ git add .
$ git commit -m "remove xyz file"
```

完~

可参考文章：

1. [git-guide](http://rogerdudler.github.io/git-guide/)
2. [廖雪峰 git 教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
3. [git-scm](https://www.git-scm.com/)
4. [Git Cheat Sheet 中文版](https://blog.csdn.net/github_37515447/article/details/56840610)
