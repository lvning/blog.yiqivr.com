title: 使用CardView实现卡片式Google Plus布局
date: 2015-01-14 07:37:11
tags: [Android]
---
经常使用Google APP的人应该对卡片式设计非常熟悉了吧，不管是Google Now、Google Email、Google+等都沿用了卡片式设计的风格，非常清新明了，感觉没有任何冗余的信息。

那么我们在Android开发中如何采用这种设计呢？其实在android.support.v7包中Android已经为我们提供了卡片式设计的视图，名字就叫做CardView。CardView使用起来也是非常的简单，当你想把一个视图作为一个卡片式视图的话，你就用CardView包裹它就行了，因为CardView是从FrameLayout派生来的。

接下来我们写一个小Demo来模仿Google Plus的布局设计，并且给每个卡片添加上从下往上弹出的动画。

再来看看代码如何实现吧：

*card_item.xml*

```
<?xmlversion="1.0"encoding="utf-8"?>
<android.support.v7.widget.CardViewxmlns:android="http://schemas.android.com/apk/res/android"
xmlns:app="http://schemas.android.com/apk/res-auto"
android:layout_height="wrap_content"
app:cardCornerRadius="3dp"
app:cardElevation="3dp">
<RelativeLayout
android:layout_width="match_parent"
android:layout_height="wrap_content">
<ImageView
android:id="@+id/iv"
android:layout_width="match_parent"
android:layout_height="230dp"
android:scaleType="centerCrop"
android:src="@drawable/ic_launcher"/>
<RelativeLayout
android:layout_width="match_parent"
android:layout_height="wrap_content"
android:layout_alignBottom="@id/iv">
<TextView
android:layout_width="match_parent"
android:layout_height="wrap_content"
android:gravity="center_vertical"
android:paddingBottom="3dp"
android:paddingLeft="8dp"
android:paddingRight="8dp"
android:paddingTop="3dp"
android:text="Support android.widget classes to assist with development of applications for android API le
android:textSize="15sp" />
</RelativeLayout>
</RelativeLayout>
</android.support.v7.widget.CardView>
```

注意到最外层的布局了吗，我们使用的正是CardView样式。在这儿我们分别使用了两个属性，分别是cardCornerRadius和cardElevation，分别代表圆角和海拔（设置越大卡片显示的越突出）。还有其他的什么属性能够设置呢，看看源码的attr.xml：

```
<resources>
<declare-styleablename="CardView">
<attrname="cardBackgroundColor"format="color"/>
<attrname="cardCornerRadius"format="dimension"/>
<attrname="cardElevation"format="dimension"/>
<attrname="cardMaxElevation"format="dimension"/>
<attrname="cardUseCompatPadding"format="boolean"/>
<attrname="cardPreventCornerOverlap"format="boolean"/>
<attrname="contentPadding"format="dimension"/>
<attrname="contentPaddingLeft"format="dimension"/>
<attrname="contentPaddingRight"format="dimension"/>
<attrname="contentPaddingTop"format="dimension"/>
<attrname="contentPaddingBottom"format="dimension"/>
</declare-styleable>
</resources>
```

可以看到除了我们用到的两个属性，还可以设置padding等。接下来在看看模仿Google Plus动画该如何实现，这部分应该在每个CardView出现的时候开始动画，也就是说应该在Adapter里面实现：

```
public class CardAdapter extends BaseAdapter {
privatestaticfinallongANIM_DEFAULT_SPEED = 500L;
private Interpolator interpolator;
private SparseBooleanArray positionsMapper;
private ArrayList<? extends Object> items;
privateintheight, previousPostition;
privatelonganimDuration;
private View v;
private Context context;
protected CardAdapter(Context context, ArrayList<? extends Object> items) {
this.context = context;
this.items = items;
previousPostition = -1;
positionsMapper = new SparseBooleanArray(getCount());
WindowManager windowManager = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
height = windowManager.getDefaultDisplay().getHeight() / 2;
interpolator = new AccelerateDecelerateInterpolator();
}
@Override
public int getCount() {
return items.size();
}
@Override
public Object getItem(int position) {
return items.get(position);
}
@Override
public long getItemId(int position) {
return position;
}
@Override
public View getView(int position, View convertView, ViewGroup parent) {
v = LayoutInflater.from(context).inflate(R.layout.card_item, null);
// TODO 填充内容
// Google plus动画
if (v != null && !positionsMapper.get(position) && position > previousPostition) {
animDuration = ANIM_DEFAULT_SPEED;
previousPostition = position;
// 初始化
v.setTranslationX(0.0F);
v.setTranslationY(height);
v.setRotationX(12.0F);
v.setScaleX(0.7F);
v.setScaleY(0.6F);
// 开始动画至正常显示
v.animate().rotationX(0.0F).rotationY(0.0F).translationX(0).translationY(0).setDuration(animDuration)
.scaleX(1.0F).scaleY(1.0F).setInterpolator(interpolator);
// 只执行一次动画
positionsMapper.put(position, true);
}
returnv;
}
}
```

这样我们就实现了类似Google+的动画了。最后就是最简单的调用了：

```
public class MainActivity extends FragmentActivity {
@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);
ListView list = (ListView) findViewById(R.id.listview);
ArrayList<Object> items = new ArrayList<Object>();
for (int i = 0; i < 18; i++) {
items.add(new Object());
}
list.setAdapter(new CardAdapter(this, items));
}
```

好了，一个类似Google+的Demo我们就完成了，当然我们这儿使用的是最简单的ListView，大家可以自己实现其他的效果，比如类似瀑布流等等效果。
