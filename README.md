# git操作教程

![git 结构图](img/git.png "git结构图")


## 版本跳转 - [git reset命令](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E7%BD%AE%E6%8F%AD%E5%AF%86)

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

```
git init　初始化仓库
git add <file>　添加到暂存区
git commit -m <message> 提交到本地库
git status 查看当前状态
git rm --cached 从暂存区移出
git mv README.md README 移动文件
git log --pretty=oneline 日志
git checkout -- <file>　撤销文件修改
git push [remote-name] [branch-name] 推送到某个仓库的某个分支
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

