title: 在Android中实现阴影效果
date: 2015-03-03 06:52:19
tags: [Android]
---
在Android L推出后，Google提出了全新的设计语言：材质设计。其中很重要的一点就是阴影效果的使用，你可以为每一个View设置一个`elevation`值，相当于除了x、y之外的z值，z值决定了阴影的大小，z值越大表示阴影越大。z值包含两个成分：`elevation`和`translation`。elevation是一个静态的成分，`translation`使用了动画：`Z = elevation + translationZ`。

在layout中设置`elevation`，使用`android:elevation`属性。在代码中设置，使用`View.setElevation()`方法。设置一个View的`translation`，使用`View.setTranslationZ()`方法。`ViewPropertyAnimator.z()`和`ViewPropertyAnimator.translationZ()`能使你更轻易的推动Views的`elevation`。

因此，想要在5.0（API 21）以及以后想要设置阴影非常简单我们只需要设置`elevation`属性就可以了。比如我们想让一张图片显示阴影：

```
<ImageView
   android:layout_width="100dp"
   android:layout_height="100dp"
   android:layout_marginTop="10dp"
   android:background="#fff"
   android:elevation="10dp"
   android:src="@drawable/ic_launcher" />
```

需要注意的一点是：必须要设置`background`并且不能设置是透明背景，这样在真机上才能显示出来，没有设置的话预览能显示，但是真机并没有效果，在`ViewGroup`中也是一样。

那我们想在5.0之前也实现阴影效果怎么办呢？有两个办法：第一种是自定义layer-list，第二种办法是使用nine-patch图片。

先来看看nine-patch方案：大概原理就是使用一张边界是阴影效果的.9图片，然后设置为背景，可以让美工帮忙切一张，也可以使用系统自带的，有个叫`android:background="@android:drawable/dialog_holo_light_frame`”，设置后的效果就是带阴影的效果。

再来看看layer-list方案，如果我们需要一张矩形的阴影效果，则大概应该这么定义：

```
<?xmlversion="1.0"encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Drop Shadow Stack -->
    <item>
        <shape>
            <padding
                android:bottom="2dp"
                android:left="2dp"
                android:right="2dp"
                android:top="2dp" />
            <solid android:color="#10CCCCCC" />
        </shape>
    </item>
    <item>
        <shape>
            <padding
                android:bottom="2dp"
                android:left="2dp"
                android:right="2dp"
                android:top="2dp" />
            <solid android:color="#20CCCCCC" />
        </shape>
    </item>
    <item>
        <shape>
            <padding
                android:bottom="2dp"
                android:left="2dp"
                android:right="2dp"
                android:top="2dp" />
            <solid android:color="#40CCCCCC" />
        </shape>
    </item>
    <item>
        <shape>
            <padding
                android:bottom="2dp"
                android:left="2dp"
                android:right="2dp"
                android:top="2dp" />
            <solid android:color="#50CCCCCC" />
        </shape>
    </item>
    <item>
        <shape>
            <padding
                android:bottom="2dp"
                android:left="2dp"
                android:right="2dp"
                android:top="2dp" />
            <solid android:color="#60CCCCCC" />
        </shape>
    </item>
    <!-- Background -->
    <item>
        <shape>
            <solid android:color="#FFFFFF" />
            <corners android:radius="3dp" />
        </shape>
    </item>
</layer-list>
```

原理就是沿着边界一层一层的绘制，颜色由深到浅。

同样，如果需要实现像FAB（Floating ActionBar）一样的阴影效果，也能这么定义，把矩形换做圆形即可：

```
<?xmlversion="1.0"encoding="utf-8"?>
<selectorxmlns:android="http://schemas.android.com/apk/res/android">
    <itemandroid:state_pressed="true">
        <layer-list>
            <!-- Shadow -->
            <itemandroid:top="1dp"android:right="1dp">
                <layer-list>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#08000000"/>
                            <padding
                                android:bottom="3px"
                                android:left="3px"
                                android:right="3px"
                                android:top="3px"
                                />
                        </shape>
                    </item>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#09000000"/>
                            <padding
                                android:bottom="2px"
                                android:left="2px"
                                android:right="2px"
                                android:top="2px"
                                />
                        </shape>
                    </item>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#10000000"/>
                            <padding
                                android:bottom="2px"
                                android:left="2px"
                                android:right="2px"
                                android:top="2px"
                                />
                        </shape>
                    </item>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#11000000"/>
                            <padding
                                android:bottom="1px"
                                android:left="1px"
                                android:right="1px"
                                android:top="1px"
                                />
                        </shape>
                    </item>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#12000000"/>
                            <padding
                                android:bottom="1px"
                                android:left="1px"
                                android:right="1px"
                                android:top="1px"
                                />
                        </shape>
                    </item>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#13000000"/>
                            <padding
                                android:bottom="1px"
                                android:left="1px"
                                android:right="1px"
                                android:top="1px"
                                />
                        </shape>
                    </item>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#14000000"/>
                            <padding
                                android:bottom="1px"
                                android:left="1px"
                                android:right="1px"
                                android:top="1px"
                                />
                        </shape>
                    </item>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#15000000"/>
                            <padding
                                android:bottom="1px"
                                android:left="1px"
                                android:right="1px"
                                android:top="1px"
                                />
                        </shape>
                    </item>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#16000000"/>
                            <padding
                                android:bottom="1px"
                                android:left="1px"
                                android:right="1px"
                                android:top="1px"
                                />
                        </shape>
                    </item>
                </layer-list>
            </item>
            <!-- Blue button pressed -->
            <item>
                <shape android:shape="oval">
                    <solid android:color="#90CAF9"/>
                </shape>
            </item>
        </layer-list>
    </item>
    <item android:state_enabled="true">
        <layer-list>
            <!-- Shadow -->
            <item android:top="2dp"android:right="1dp">
                <layer-list>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#08000000"/>
                            <padding
                                android:bottom="4px"
                                android:left="4px"
                                android:right="4px"
                                android:top="4px"
                                />
                        </shape>
                    </item>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#09000000"/>
                            <padding
                                android:bottom="2px"
                                android:left="2px"
                                android:right="2px"
                                android:top="2px"
                                />
                        </shape>
                    </item>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#10000000"/>
                            <padding
                                android:bottom="2px"
                                android:left="2px"
                                android:right="2px"
                                android:top="2px"
                                />
                        </shape>
                    </item>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#11000000"/>
                            <padding
                                android:bottom="1px"
                                android:left="1px"
                                android:right="1px"
                                android:top="1px"
                                />
                        </shape>
                    </item>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#12000000"/>
                            <padding
                                android:bottom="1px"
                                android:left="1px"
                                android:right="1px"
                                android:top="1px"
                                />
                        </shape>
                    </item>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#13000000"/>
                            <padding
                                android:bottom="1px"
                                android:left="1px"
                                android:right="1px"
                                android:top="1px"
                                />
                        </shape>
                    </item>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#14000000"/>
                            <padding
                                android:bottom="1px"
                                android:left="1px"
                                android:right="1px"
                                android:top="1px"
                                />
                        </shape>
                    </item>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#15000000"/>
                            <padding
                                android:bottom="1px"
                                android:left="1px"
                                android:right="1px"
                                android:top="1px"
                                />
                        </shape>
                    </item>
                    <item>
                        <shape android:shape="oval">
                            <solid android:color="#16000000"/>
                            <padding
                                android:bottom="1px"
                                android:left="1px"
                                android:right="1px"
                                android:top="1px"
                                />
                        </shape>
                    </item>
                </layer-list>
            </item>
            <!-- Blue button -->
            <item>
                <shape android:shape="oval">
                    <solid android:color="#03A9F4"/>
                </shape>
            </item>
        </layer-list>
    </item>
</selector>
```

这样就能实现通用平台的阴影效果了，还有一点不要被迷惑了，support-v4里面有个`ViewCompat.setElevation(iv, 2.0f);`方法，试验证明是没用的。

更多的学习大家可以参考API DEMO的`ShadowCardDrag`和`ShadowCardStack`两个Demo。

最后来看看实现的效果图：

![](http://pic.yupoo.com/lvning10086/EQUyYk71/SsTq7.png)

第一个是使用的.9背景图片，也就是之前提到的属性，第二个和第三个是使用的layer-list，最后一个是使用的elevation属性。