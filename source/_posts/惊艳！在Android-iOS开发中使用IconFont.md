title: 惊艳！在Android/iOS开发中使用IconFont
date: 2015-01-21 03:19:52
tags: [Android,iOS]
---

### 引言

本人是图标控、应用控。图标好看的、应用设计漂亮的，即使不用我也不会卸载它，给它在我的机机上留上一席之地，偶尔打开过过瘾。之前使用过一款叫做Radium的iOS APP，是一款收音机应用，给我留下了十足深刻的印象，不为别的，就是因为设计的非常漂亮，以致我现在也在Mac平台上继续使用它。多说无用，先给大家看看几张截图：

![](http://pic.yupoo.com/lvning10086/EQTTjHv3/FAyy5.jpg)

这三张图都是同一个界面，列表背景色是专辑封面的主色，列表标题色是专辑封面第二主色，按下列表后列表背景色和标题色互换。这还不本文的重点，大家注意到列表的图标了吗？进入不同的专辑，界面中的每个图标的颜色居然也是不一样的，也是根据专辑的封面主色而定的！真是惊艳！

接下来我们就要思考了：

- 获取专辑封面主色、第二主色这是如何实现的呢？
- 如何动态的改变页面中图标的颜色？

关于第一个问题，比较简单，我想大多数朋友都和我想的一样，就是获取到bitmap的宽高，然后两个for循环取每个像素点的色值存到集合中。最后集合中最多的就是主色，第二多的就是第二主色，第三、第四依次类推。

主要是第二个问题，我想到了5种可能（头脑风暴，不管能不能实现，先列举出来）：

1. 使用paint（iOS中使用Quartz2D）来一个一个绘制，然后留出改变颜色的接口来动态改变。
2. 使用HTML5，我们在很多H5的网页中看到的图标也是能动态改变颜色的。
3. 使用svg。
4. 使用多套图方案。
5. 使用IconFont。

关于第一种设想，非常麻烦，一个稍微复杂点的图标绘制出来可能都要花上半天的时间，说白了都是体力活。

第二种可能，Radium页面中又许多流畅而且华丽的动画，应该不是H5开发的，并且我们现在考虑的是如何在本地应用中实现，so，out！

第三种，要使用svg，首先得保证你的Android设备得高于于2.3，因为Android 2.3不支持SVG格式，并且目前2.3还是有10%左右份额的。

第四种，我们是摸不清下一个专辑封面是张什么样的啊，太繁琐了而且大大增加了客户端的体积。

看来只剩下第五种可能了...

### 什么是iconfont

利用字体工具把我们平时 Web 上用的图形图标（icons）转换成 web fonts，就成了 icon fonts，它可以借助 CSS 的 @font-face 嵌入到网页里，用以显示 icons。因为字体是矢量化图形，它天生具有「分辨率无关」的特性，在任何分辨率和PPI下面，都可以做到完美缩放，不会像传统位图，如：png，jpeg，放大后有锯齿或模糊现象。

### 它有什么优点

- **文件小**：相比图片几十几百KB的容量，icon fonts 几乎是羽翼级轻量。
- **加载性能好**：因为图标都被打包进一套字体内，http request 减少。这如同我们常用的 css sprites 技术。
- **支持CSS样式**：和普通字体一样，你可以利用CSS来定义大小、颜色、阴影、hover状态、透明度、渐变等等...
- **兼容性好**：web fonts 起源很早，别说主流浏览器，连IE6/7都能良好支持。除了一些老的移动端浏览器，如Android 2.1以下的初代浏览器，Opera mini 这类自限型浏览器。


**缺点：只支持纯色icon。**

为了能有个直观的印象，大家可以去[IconFont测试](http://css-tricks.com/examples/IconFont/)简单拖动滑块测试下改变每个属性（比如颜色、阴影等）的时候图标会变成什么样。

### 素材

在我们国内，比较好的矢量图标库有：[阿里巴巴矢量图标库](http://www.iconfont.cn/)。

在国外有[Font Awesome](http://fortawesome.github.io/Font-Awesome/)。

### 实现

github上已经有大牛实现了，[时光隧道](https://github.com/JoanZapata/android-iconify)。