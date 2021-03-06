# git操作教程

![git 结构图](img/git.png "git结构图")

## 基本命令

```
git init　初始化仓库
git init --bare 初始化裸仓库
git add <file> 将工作区文件添加到暂存区
git commit -m <message> 暂存区提交到本地库
git status 查看当前状态
git rm --cached 从暂存区移出
git mv README.md README 移动文件
git log --pretty=oneline 日志
git checkout -- <file>　撤销文件修改
git push [remote-name] [branch-name] 推送到某个仓库的某个分支
```

## 日志查看
```
git log 查看日志
git log --pretty=oneline    显示为一行
git log --oneline           显示为一行(hash值显示一部分)
git reflog                  显示HEAD位置
```

## 版本跳转 - [git reset命令](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E7%BD%AE%E6%8F%AD%E5%AF%86)

```git reset --soft```
![reset soft](img/reset-soft.png "reset soft")

```git reset --mixed```
![reset mixed](img/reset-mixed.png "reset mixed")

```git reset --hard```
![reset hard](img/reset-hard.png "reset hard")

基于索引值跳转
```
git reset --hard [索引值]
git reset --hard 5cb677d
```
版本回退
```
git reset --hard HEAD^      回退一个版本
git reset --hard HEAD^^     回退两个版本
git reset --hard HEAD^^^    回退三个版本
git reset --hard HEAD~4     回退四个版本
```

git reset参数
```
--soft  仅仅在本地库移动HEAD指针
--mixed 本地库移动HEAD指针，重置暂存区
--hard  本地库移动HEAD指针，重置暂存区，重置工作区
```

## 差异对比
```
git diff <file>                     工作区和暂存区对比
git diff <本地库历史版本> <file>     工作区和本地库对比
git diff                            工作区和暂存区对比(所有文件)
```

## 分支操作
```
git branch -v                                       查看当前分支
git branch <branch name>                            创建分支
git checkout <branch name>                          切换分支
git merge <branch name>                             分支合并
git fetch <远程库别名> <远程库分支名>                 拉取远程库内容(不合并到本地库)
git pull                                            拉取(fetch + merge <远程库别名/远程分支名>)
```

## 保存现场

暂存区没有commit时，无法checkout到另一个分支，这时需要将当前工作stash后再切换。

stash list使用栈数结构，先入后出

```
git stash               保存现场
git stash list          查看保存的记录

git stash pop           还原最新保存记录并从stash list中删除

git stash apply         还原最新保存记录，不从stash list中删除
git stash drop          删除最新保存记录
```

## git搜索

```
git grep <str>          搜索str字符串
git grep -n <str>       搜索str字符串(显示行号)
git grep --count <str>  搜索str字符串(显示每个文件匹配数)
git grep -p <str> *.c   搜索str字符串(显示函数)
git grep --break <str>  搜索str字符串(两个文件中显示空行)
git grep --heading <str>搜索str字符串(文件名显示在上方)
```

## GitWeb 简易网页查看器

通过```git instaweb```命令启动web服务

支持apache2,lighttpd, mongoose, plackup and webrick服务器，默认为lighttpd

通过```git instaweb --httpd=webrick```命令指定web服务器

## git服务

使用 ```git daemon```命令搭建简单git服务，默认监听9418端口

初始化一个裸仓库(不带工作区)

```git init --bare /opt/git-repo/repo1.git```

```git daemon --verbose --export-all --reuseaddr --enable=receive-pack --base-path=/opt/git-repo/ /opt/git-repo/```

客户端使用```git clone git://localhost/repo1.git```检出仓库

## git api

编程方式使用git [libgit2](https://libgit2.org/) [jgit](https://www.eclipse.org/jgit/)

- [c](https://github.com/libgit2/libgit2)
- [c#](https://github.com/libgit2/libgit2sharp)
- [java](https://www.eclipse.org/jgit/)
- [python](https://github.com/libgit2/pygit2)
- [go](https://github.com/libgit2/git2go)

## github api

[github develop api](https://developer.github.com/v3/libraries/)

- [c#](https://github.com/octokit/octokit.net)
- [javascript](https://github.com/octokit/octokit.js)
- [python](https://github.com/PyGithub/PyGithub)
- [go](https://github.com/google/go-github)


## 缓存密码

参考[凭证存储](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%87%AD%E8%AF%81%E5%AD%98%E5%82%A8#r_credential_caching)

```git config --global credential.helper cache```

## git gc

```git gc```命令来优化本地库文件修订版

执行前 .git/objects目录下显示如下,00/  15/  2d/这样的目录为每次提交的sha1值
```
00/  15/  2d/  37/  48/  57/  62/  76/  85/  9a/  b3/  b8/  c8/  da/  info/
02/  1d/  31/  3a/  4f/  5c/  70/  82/  87/  a9/  b4/  bf/  cd/  ea/  pack/
05/  2b/  32/  41/  54/  5f/  72/  83/  88/  b0/  b6/  c4/  d3/  ee/
```
执行```git gc```后.git/objects目录下显示如下
```
info/  pack/
```


## 子模块 submodule

git仓库添加子模块
```
git submodule add https://github.com/ts-zhang/git-module.git
git submodule status                        查看子模块状态
git submodule foreach <git command>         所有子模块中执行命令
git clone --recurse-submodules              克隆带子模块的仓库
git submodule update --init --recursive     初始化带子模块仓库(git clone后)
```

> 在主仓库中pull拉取代码，不能拉取submodule中的修改，需要执行 ```git submodule foreach git pull```

## 文件标注

>显示文件每一行的最后修改时间```git blame -L 10,20 README.md```

显示结果如下
```
323ddd3d (zts 2020-01-13 19:46:02 +0800 10) git add <file> 将工作区文件添加到暂存区
323ddd3d (zts 2020-01-13 19:46:02 +0800 11) git commit -m <message> 暂存区提交到本地库
323ddd3d (zts 2020-01-13 19:46:02 +0800 12) git status 查看当前状态
323ddd3d (zts 2020-01-13 19:46:02 +0800 13) git rm --cached 从暂存区移出
323ddd3d (zts 2020-01-13 19:46:02 +0800 14) git mv README.md README 移动文件
323ddd3d (zts 2020-01-13 19:46:02 +0800 15) git log --pretty=oneline 日志
323ddd3d (zts 2020-01-13 19:46:02 +0800 16) git checkout -- <file>　撤销文件修改
323ddd3d (zts 2020-01-13 19:46:02 +0800 17) git push [remote-name] [branch-name] 推送到某个仓库的某个分支
323ddd3d (zts 2020-01-13 19:46:02 +0800 18) ```
323ddd3d (zts 2020-01-13 19:46:02 +0800 19) 
323ddd3d (zts 2020-01-13 19:46:02 +0800 20) ## 日志查看
```

## 打包

>git仓库打包可以用来离线传输

```
git bundle create repo.bundle HEAD master       打包master分支
git clone repo.bundle repo                      解包为repo仓库
```

## 标签

轻标签  ```git tag <tagname>```

注解标签```git tag -a <tagname>```

查看标签```git tag -n```

删除标签```git tag -d <tagname>```

## 子模块 subtree

git仓库添加子模块

```
git remote add subtree https://github.com/ts-zhang/git-module.git   添加远程仓库
git subtree add -P subtree subtree master                           添加子模块
git subtree add -P subtree subtree master --squash                  添加子模块(子模块的多次提交合并)
```

>submodule和subtree区别 submodule为连接，subtree为拷贝

## 别名

```
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.br branch
git config --global alias.last 'log -1'
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

## 文件忽略

某些工作区中的文件不想提交，比如log，或IDE的配置等．
在Git工作区的根目录下创建一个特殊的.gitignore文件，然后把要忽略的文件名填进去，Git就会自动忽略这些文件．

[gitignore](https://github.com/github/gitignore)

## 客户端工具

Github工具 - [Hub](https://hub.github.com/)

GUI工具 - [Sourcetree](https://www.sourcetreeapp.com/)

官方客户端工具 - [git](https://git-scm.com/downloads)

历史查看工具 - [Git History](https://githistory.xyz/)

