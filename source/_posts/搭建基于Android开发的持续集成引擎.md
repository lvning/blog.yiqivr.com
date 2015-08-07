title: 搭建基于Android开发的持续集成引擎
date: 2014-11-28 07:50:53
tags: [Android]
---
### 前言

[《持续集成:软件质量改进和风险降低之道》](http://book.douban.com/subject/10769596/)</a>全面深入地讨论持续集成的各个方面，介绍了一种增加项目可见性、降低项目失败风险的有效实践。那我们在移动开发中要怎么应用呢？ 持续集成（Continuous integration）简称CI，是一种软件开发的实践，可以让团队在持续集成的基础上收到反馈并加以改进，不必等到开发的后期才寻找和修复缺陷。在平时的开发中，项目一般会经历这几个步骤：

1. 程序员从源代码仓库下载最新程序
2. li>程序员编写代码、测试用例，并提交更新结果给版本控制仓库
3. CI服务器根据触发条件，从版本控制仓库提取最新代码，交给构建工具的工作空间
4. 构建工具对代码进行编译、测试，并进行打包。如有必要，实现产品部署、发布
5. 通过构建工具与版本控制工具的配合，实现产品版本控制与管理
6. 建立、管理项目开发的工作网站

有了持续集成，我们可以大大减少集成的问题，让团队能够更快的开发内聚的软件。 接下来我们看看怎么搭建基于**Jenkins+ant(maven/gradle)+github**的集成系统。

### 什么是Jenkins?

Jenkins是一个开源项目，提供了一种易于使用的持续集成系统，使开发者从繁杂的集成中解脱出来，专注于更为重要的业务逻辑实现上。同时 Jenkins 能实施监控集成中存在的错误，提供详细的日志文件和提醒功能，还能用图表的形式形象地展示项目构建的趋势和稳定性。Jenkins 的前身是Hudson是一个可扩展的持续集成引擎。

### Jenkins的安装与配置

首先我们在jenkins的[官网](http://jenkins-ci.org/)下载war安装包，下载完成后放在tomcatwebapps目录下，然后启动tomcat在浏览器访问http://localhost:8080/jenkins，就可以成功安装。 接下来我们配置Jenkins,安装好了以后，可以看到如下界面：

![](http://pic.yupoo.com/lvning10086/Ef9dQoxL/czF6X.png)

主页显示了我们构建的项目列表，当然现在大家看到的应该是空的。接下来我们安装一些必要的插件，点击系统管理----》管理插件看见以下界面：

![](http://pic.yupoo.com/lvning10086/Ef9edUBm/9zgu4.png)

可以看见Jenkins已经为我们安装了一些必要厂要的插件，比如Ant Plugin、JUnit Plugin等等。然后点击可选插件，勾选Git Plugin、Github Plugin。（上面有个过滤，大家别慢慢滑着找啊，手会酸的，Jenkins有非常丰富的插件，以后大家还可以自行配置XCode Plugin，配合Command Tool使用。）然后点击直接安装。 安装好了插件之后，再来配置一些系统，点击系统管理----》系统配置看到如下界面：

![](http://pic.yupoo.com/lvning10086/Ef9p5VP0/medium.jpg)

系统消息可以填写一些消息，这个信息会显示在首页顶部， 用来向用户发布一些系统范围的通知或公告，兼容HTML标签格式。

![](http://pic.yupoo.com/lvning10086/Ef9prQC1/medium.jpg)

然后滑到Git Plugin，配置好自己的姓名和邮箱。当然，没有安装Git和ant的小伙伴们需要自己添加下。

![](http://pic.yupoo.com/lvning10086/Ef9q6if8/medium.jpg)

如果需要邮件自动提醒功能的话，还可以配置上SMTP邮件，配置完成后可以发送测试邮件测试配置。

### Jenkins的使用

配置好了以后，我们来新建一个项目：点击新建，输入名称，选择构建一个自由风格的软件项目。可以看到如下的界面：

![](http://pic.yupoo.com/lvning10086/Ef9eIEGx/medium.jpg)

![](http://pic.yupoo.com/lvning10086/Ef9d3Cyq/medium.jpg)

接下来在GitHub project一栏输入你的项目Git地址，在源码管理一栏选择Git，然后在Repository URL输入Git仓库地址，然后在Credentials添加一个你的账号。

![](http://pic.yupoo.com/lvning10086/Ef9dCyT8/medium.jpg)

接下来我们可以选择构建触发器，构建触发器可以在我们往Git仓库push的时候自动pull代码然后自动编译打包，我们勾选Build when a change is pushed to GitHub和Poll SCM，然后出现了日程表一栏，在这儿我们输入H/5 * * * * ，表示每隔5分钟获取一次更新信息，大家可以自行配置。

接下来到了构建了，我们增加构建步骤，选择Invoke Ant，在Ant Version一栏选择我们在系统配置中添加的Ant别名，然后Target大家看着办的，如果是发布的话就填release，测试包可以天debug，跟大家平时使用Ant一样，还可以点击高级，有个Properties等，这些也和Ant配置一样，可以填写sdk.dir=\"xxx\"等等信息。

然后点击增加构建后操作步骤，我们可以选择E-mail Notification，这样可以在构建成功以后给相应的人员发送邮件，比如张三最后一次提交，然后构建出问题了，就会发邮件给张三让他滚回公司来改bug。最后我们点击保存就完成一个项目的创建了。

### 开始使用Jenkins构建项目

我们回到项目主页，入下图所示：

![](http://pic.yupoo.com/lvning10086/Ef9eWQgr/medium.jpg)

接下来点击立即构建，见证奇迹的时候到了，构建历史栏目会显示你当前的构建进度，最后构建成功之后会显示蓝色小球，失败会显示红色小球，如果失败，可以点击红色小球查看控制台输出，然后对应解决bug。

![](http://pic.yupoo.com/lvning10086/Ef9feXDK/zQald.png)

### 最后

最后？其实Jenkins还有许多强大的功能，比如用户权限管理等等，大家自己折腾吧！最后最后就是享受CI给我们带来的极大便捷吧！HAVE FUN！