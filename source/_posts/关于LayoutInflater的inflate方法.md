title: 关于LayoutInflater的inflate方法
date: 2014-11-12 10:47:17
tags: [Android]
---
`inflater.inflate(R.layout.sample, null);`方法是我们经常用到的，比如在`adapter`的`getView(int position, Vew convertView, ViewGroup parent)`和`fragment`的`onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState)`方法中。但是这个方法却有个问题，下面我们来做个试验：

*main.xml*

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    	android:id="@+id/container"
    	android:layout_width="match_parent"
    	android:layout_height="match_parent">
    <LinearLayout/>
    
另外一个布局文件*red.xml*

    <?xml version="1.0" encoding="utf-8"?>
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
    	android:layout_width="25dp"
    	android:layout_height="25dp"
    	android:background="#ff0000"
    	android:text="red"/>
    	
测试代码：

    public class InflaterTest extends Activity {
    	private View view;
    	@Override
    	public void onCreate(Bundle savedInstanceState) {
    		super.onCreate(savedInstanceState);
    		setContentView(R.layout.main);
    		ViewGroup parent = (ViewGroup) findViewById(R.id.container);
    		// result: layout_height=wrap_content layout_width=match_parent
    		view = LayoutInflater.from(this).inflate(R.layout.red, null);
    		parent.addView(view);
    		// result: layout_height=100 layout_width=100
    		view = LayoutInflater.from(this).inflate(R.layout.red, null);
    		parent.addView(view, 100, 100);
    		// result: layout_height=25dp layout_width=25dp
    		// view=textView due to attachRoot=false
    		view = LayoutInflater.from(this).inflate(R.layout.red, parent, false);
    		parent.addView(view);
    		// result: layout_height=25dp layout_width=25dp
    		// parent.addView not necessary as this is already done by attachRoot=true
    		// view=root due to parent supplied as hierarchy root and attachRoot=true
    		view = LayoutInflater.from(this).inflate(R.layout.red, parent, true);
    	}
    }
    
由上面的结果可以知道，除了使用最后一种方法，其他方法不额外设置宽高的话，其他的都会忽略掉*red.xml*文件顶层布局宽高信息。还有一种做法就是在*red.xml*中再添加一层父布局，这样也能达到预期的效果，但是这样做对于布局的性能是很受影响的（可以使用强大的工具[hierarchy viewer](http://developer.android.com/guide/developing/tools/hierarchy-viewer.html)来检测），特别是在`adapter`的`getView`中。但是使用`inflate(int resource, ViewGroup root, boolean attachToRoot)`方法的话，不仅可以节省代码量，而且可以很好的减少布局带来的性能消耗。

其他情况：当我们在自定义`AlertDialog`的显示内容时，`AlertDialog`并没有把布局ID作为参数的`setView()`方法，而是直接传入要显示的`View`，这时我们也要用到`LayoutInflater`，这时我们并没有那个`parentView`（事实上也并不存在这个`View`），现在我们只能用`inflate(int resource, ViewGroup</em> root)`方法了，因为`AlertDialog`会擦除传入的`View`的`LayoutParams`信息，并且替换成`match_parent`。

综上，让我们尝试把`inflater.inflate(R.layout.sample, null);`改成`inflater.inflate(R.layout.sample, parent, false);`吧！