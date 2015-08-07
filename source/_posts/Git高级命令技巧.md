title: Git高级命令技巧
date: 2014-11-27 15:27:24
tags: [Android]
---
毫无疑问，Git与Github已经逐渐成为开源世界的协作首选了，一些基本的Git入门知识与命令这儿就不再概述了（在此推荐经典入门教程[Pro Git中文在线版](http://git-scm.com/book/zh/v1)），接下来列举一些高级技巧：

### 显示漂亮的Git log日志

一般情况下我们使用`git log`命令来显示log日志，效果如下图：

![](http://pic.yupoo.com/lvning10086/EeSZZs40/iDQBj.png)

`git log`看起来就是稍微死板了点儿，从中获取不了太多的信息，接下来我们使用下面的命令来试试：

```
$ git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --
```

再来看看效果如何：

![](http://pic.yupoo.com/lvning10086/EeT0h1qe/FroXE.png)

是不是秒变白富美啊！其实我们还可以更简化一下，使用别名：

```
$ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --"
```

在这之后，我们使用`git lg`命令就可以了，是不是方便了许多啊。