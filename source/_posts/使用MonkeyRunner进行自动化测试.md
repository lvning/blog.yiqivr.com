title: 使用MonkeyRunner进行自动化测试
date: 2015-01-08 05:38:55
tags: [Android]
---
### 什么是 MonkeyRunner

MonkeyRunner是Google Android SDK下的一个工具，用于自动化测试Android程序。

Monkeyrunner工具提供了一套API，使用这些api写出的Python程序可以在黑盒地控制Android设置和模拟器。有了Monkeyrunner，我们可以通过python语句控制apk包的安装和卸载、启动app、向app发送各种动作指令、截取图片并保存。

Monkeyrunner和Monkey不同，Monkey是直接运行在adb shell中的命令，它随机的生成用户或者系统的各种随机事件（暴力测试，虐机必备），而MonkeyRunner则可以通过api来产生特定命令和事件来控制设备和模拟器。

MonkeyRunner在android测试中有下面特点：

- 支持多设备： API可以跨多个设备或模拟器实施测试套件。您可以在同一时间接上所有的设备或一次启动全部模拟器（或统统一起），依据程序依次连接到每一个，然后运行一个或多个测试。您也可以用程序启动一个配置好的模拟器，运行一个或多个测试，然后关闭模拟器。

- 功能测试： monkeyrunner可以为一个应用自动贯彻一次功能测试。您提供按键或触摸事件的输入数值，然后观察输出结果的截屏。

- 回归测试：monkeyrunner可以运行某个应用，并将其结果截屏与既定已知正确的结果截屏相比较，以此测试应用的稳定性。

- 可扩展的自动化：由于monkeyrunner是一个API工具包，您可以基于Python模块和程序开发一整套系统，以此来控制Android设备。除了使用monkeyrunner API之外，您还可以使用标准的Python os和subprocess模块来调用如adb这样的Android工具。

- monkeyrunner工具使用Jython（使用Java编程语言的一种Python实现）。Jython允许monkeyrunnerAPI与Android框架轻松的进行交互。使用Jython，您可以使用Python语法来获取API中的常量、类以及方法。


### MonkeyRunner使用范例

接下来例举一个基本操作的Python脚本monkey_flow.py：

```
#coding=utf-8
# python引入monkeyRunner模块
from com.android.monkeyrunner import MonkeyRunner, MonkeyDevice
# 连接设备, 获得一个MonkeyDevice对象
device = MonkeyRunner.waitForConnection()
# 安装apk包. 返回值是boolean，可以判断是否安装成功
#device.installPackage('myproject/bin/MyApplication.apk')
# 设置要测试的包名
package = 'superdp.o20.dns.com'
# 设置启动页面
activity = 'com.dns.o2o.superdp.ui.activity.SplashScreenActivity'
# 设置component
runComponent = package + '/' + activity
# 启动应用程序
device.startActivity(component=runComponent)
# 休眠5秒
MonkeyRunner.sleep(5)
# 截图
#result = device.takeSnapshot()
# 保存截图
#result.writeToFile('shot.png','png')
# 点击坐标
device.touch(150,685,'DOWN_AND_UP')
MonkeyRunner.sleep(3)
# 从(550,1250)拖拽到(550,850)
device.drag((550,1250),(550,850),2)
MonkeyRunner.sleep(3)
# 进入一个child为fragment的viewpager，滑到第二页
device.drag((850,1150),(150,1150),1)
MonkeyRunner.sleep(3)
# 返回第一页
device.drag((150,1150),(850,1150),1)
MonkeyRunner.sleep(3)
# 点击列表其中一项
device.touch(555,480,'DOWN_AND_UP')
MonkeyRunner.sleep(3)
# 进入详情，点击按钮提示未登录进入登录界面
device.touch(800,1680,'DOWN_AND_UP')
MonkeyRunner.sleep(3)
# 点击用户名输入框
device.touch(450,421,'DOWN_AND_UP')
# 输入username
device.type('username')
# 点击密码输入框
device.touch(427,582,'DOWN_AND_UP')
# 输入password
device.type('password')
# 点击登录
device.touch(395,911,'DOWN_AND_UP')
MonkeyRunner.sleep(3)
# 用户名密码错误登录失败，一直返回退出应用
device.press('KEYCODE_BACK', 'DOWN_AND_UP')
device.press('KEYCODE_BACK', 'DOWN_AND_UP')
device.press('KEYCODE_BACK', 'DOWN_AND_UP')
device.press('KEYCODE_BACK', 'DOWN_AND_UP')
device.press('KEYCODE_BACK', 'DOWN_AND_UP')
```

有了脚本之后，直接在命令行进入脚本所在文件夹然后运行：

```
$ monkey runner monkey_flow.py
```

就能进行自动测试了。

接下来再来看一个实战范例：

有个需求，公司一个APP一共有几十套模板，一个个安装查看太麻烦了，我们就能用monkeyrunner来帮我们完成这些繁琐的操作：

```
#coding=utf-8
import time
import sys
import os
from com.android.monkeyrunner import MonkeyRunner, MonkeyDevice, MonkeyImage
packageName = 'superdp.o20.dns.com_package13'
activity = 'com.dns.o2o.superdp.ui.activity.SplashScreenActivity'
componentName = packageName + '/' + activity
initTime = 5
name = sys.argv[0].split('\\\\\')
filename = name[len(name) - 1]
rootpath = os.path.split(os.path.realpath(sys.argv[0]))[0]
dir = rootpath + '/apk/'
screenPath = rootpath + '/screenShot/'
countApk = len(os.listdir(dir))
print('Connecting...')
device = MonkeyRunner.waitForConnection()
print('Removing...')
device.removePackage(packageName)
print('Removing package successful!')
MonkeyRunner.sleep(2)
for i in os.listdir(dir):
 print(\'Installing...&lt;%s&gt;\'%i)
 path = dir + '//' + i
 device.installPackage(path)
 print('Install Successful!')
 print('Starting activity...')
 device.startActivity(component=componentName)
 MonkeyRunner.sleep(initTime)
 result = device.takeSnapshot()
 print('Tack ScreenShot...')
 result.writeToFile(screenPath + i + '.png', 'png')
 print('Removing...')
 device.removePackage(packageName)
 print('Remove apk successful!')
 MonkeyRunner.sleep(2)
```
         
脚本会依此安装apk文件夹里面的APK，然后启动程序之后截取首页图片保存在screenShot文件夹里面，接下来卸载这个APK，再安装下一个，直到APK循环完成。

### 更多关于MonkeyRunner

我们还能使用MonkeyRunner提供的API来对截图进行基线图比对，比如如果80%相似就通过。

还能使用monkey_recorder.py和monkey_playback.py文件来对APP操作进行录制：首先使用monkeyrunner monkey_recorder.py来对操作进行录制，录制成功之后另存为mr文件，比如叫result.mr，然后使用monkeyrunner monkey_playback.py result.mr命令来进行回放。

### 遇到的问题

SyntaxError: Non-ASCII character in file

查了下Python的默认编码文件是用的ASCII码，将文件存成了UTF-8也没用，解决办法很简单

只要在文件开头加入 # -*- coding: UTF-8 -*-    或者 #coding=utf-8 就行了。

### 总结

MonkeyRunner提供的操作有限，适合重复性、短路径的测试，不适合连续性的操作，由于type和drag方法的不稳定，不容易适配，很容易偏离我们的预期。MonkeyRunner操作依赖于控件坐标（也可以通过id，比较耗时），一旦UI发生改变，脚本无法使用。

Monkeyrunner的等待机制`MonkeyRunner.sleep()`，无法更加友好地等待；MonkeyRunner不提供结果断言，我们只能通过截图比较，无法摆脱人工干预。