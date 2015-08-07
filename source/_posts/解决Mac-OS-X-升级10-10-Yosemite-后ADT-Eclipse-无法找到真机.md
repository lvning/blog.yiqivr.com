title: 解决Mac OS X 升级10.10(Yosemite)后ADT(Eclipse)无法找到真机
date: 2014-10-24 08:07:26
tags: [Android]
---
升级Yosemite之后继续开发Android，发现当编译版本高于设备版本的时候设备选择器找不到真机了，WTF！难道要改低版本SDK开发不成？臣妾做不到啊！

后来捣鼓了一阵，发现解决方案有两个：

- 弹出设备选择器后，拖动第一行分割线（也就是Serial Number/AVD Name...那行）来改变宽度，我们的设备又神奇的出现了，看来是布局出了点问题。见图：

![](http://pic.yupoo.com/lvning10086/E9Fom1w2/sOlUp.png)

- 弹出设备选择器后，拔掉真机，然后插上真机，设备又现身了。

希望能帮助遇到同样困扰的童鞋~