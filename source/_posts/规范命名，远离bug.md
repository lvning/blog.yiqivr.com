title: 规范命名，远离bug
date: 2014-12-11 07:17:06
tags: [Android]
---
做好命名规范不仅能够帮我们更好的理解代码，而且能够让bug无处可藏。接下来举两个关于命名不规范可能引发的不常见问题：

### 对View设置事件不起作用

假设我们对一个Button设置点击事件，这个Button的id是btn，代码如下：

    findViewById(R.id.btn).setOnClickListener(new OnClickListener() {
    	@Override
    	public void onClick(View v) {
    		// TODO
    	}
    });
    
经过测试之后这个`onClick`打死也不会调用，各位看官会怎么想？我反正第一个感觉就是肯定这个`Button`不是我们想要监听的`Button`，接下来就仔细检查XML布局文件吧，果然，我们监听的这个`Button`在某个不起眼的小角落（比如`include`标签里面的布局），然后在上层布局（或者上上层布局）有个一样的id也叫btn，我们直接find出来的应该是上层的`Button`。解决方案很简单啦，命好名吧，或者用它的上层布局去find它，比如`parentView.findViewById(R.id.btn)`。

### 出现了`java.lang.StackOverflowError`这个问题的出现一般都是有循环调用的情况出现，少数情况是布局实在是太深了，嵌套的太多了。

大家先来看看这个自定义的drawable，名字叫*back_btn.xml*:
    
    <?xmlversion="1.0"encoding="UTF-8"?>
    <selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item
    	android:state_pressed="false"
    	android:drawable="@drawable/back_btn"/>
    <item
    	android:state_pressed="true"
    	android:drawable="@drawable/back_btned"/>
    <item 
    	android:drawable="@drawable/back_btn"/>
    </selector>
    
当我们加载这个*back_btn.xml*的时候，就会出现循环调用的问题，而且问题也不好定位。解决方案当然也是命好你的名了，比如按下的图片可以命名为*back_btn_p*,这个*selector*可以命名为*back_btn_selector.xml*。

通过上面两个范例，大家应该知道命名的规范重要性了吧，那么久从今天开始，就规范命名，远离bug吧！