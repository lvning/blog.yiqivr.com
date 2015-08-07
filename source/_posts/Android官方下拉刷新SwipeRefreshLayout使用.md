title: Android官方下拉刷新SwipeRefreshLayout使用
date: 2014-12-15 06:39:18
tags: [Android]
---
说到下拉刷新，大家肯定都用过，最常见的莫过于ListView的下拉刷新了，那大家用的是开源的第三方框架PullToRefresh还是其他的呢？或者是自己实现的呢？可能大部分都是继承ListView实现的吧？那如果我想要实现WebView的下拉刷新呢？或者GridView的呢？是不是又得重新去实现呢？好像是挺麻烦的。

今天介绍的是下拉刷新组件叫**SwipeRefreshLayout**，大家可以在最新的*android.support.v4*或者*android.support.v13*里面找到它，利用它，大家可以轻松实现你自己想要的下拉刷新，因为**SwipeRefreshLayout**继承自ViewGroup，而非特定的ListView或其他的View。我们使用它的时候只需要将想要实现下拉的View包裹在一个可以滚动的View里面既可（比如ScrollView、ListView）。

主要方法：

- **isRefreshing** 返回现在是否正在刷新状态。
- **setRefreshing(boolean refreshing)** 主动显示刷新状态可以设置为true，当刷新操作完成的时候需要调用此方法设置为false。
- **setOnRefreshListener(SwipeRefreshLayout.OnRefreshListener listener)** 主要方法，设置刷新操作回调，当用户出发下拉刷新操作后会主动回调改方法。

接下来看看如何实现的吧：

    <?xml version="1.0" encoding="utf-8"?>
    <android.support.v4.widget.SwipeRefreshLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/srl"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    	<ScrollView
    	android:layout_width="match_parent"
    	android:layout_height="match_parent">
    		<LinearLayout
    			android:layout_width="match_parent"
    			android:layout_height="match_parent">
    			...
    		</LinearLayout>
    	</ScrollView>
    </android.support.v4.widget.SwipeRefreshLayout>
    
我们将想要实现下拉刷新的视图包裹在了一个ScrollView里面。


    SwipeRefreshLayout srl = (SwipeRefreshLayout) findViewById(R.id.srl);
    rl.setOnRefreshListener(new OnRefreshListener() {
    	@Override
    	public void onRefresh() {
    		srl.postDelayed(new Runnable() {
    		@Override 
    		public void  run() {
    			srl.setRefreshing(false);
    		}, 3000);
    });

我们先实例化**SwipeRefreshLayout**，然后设置刷新回调就行了，3秒钟后设置刷新完成（当然实战中是网络操作或是其他想要的需求），是不是很简单啊。