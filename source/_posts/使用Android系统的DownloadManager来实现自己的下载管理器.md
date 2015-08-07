title: 使用Android系统的DownloadManager来实现自己的下载管理器
date: 2014-12-31 02:14:12
tags: [Android]
---
在Android系统中实现下载功能，我们平时都是使用的**HttpURLConnection**或者**HttpClient**来实现，其实在Android2.3（API9）的时候就引入了**DownloadManager**这个工具来帮我们轻松的实现下载功能。

主要用到的类有：

**DownloadManager**：下载管理器

**Request**：代表一个下载请求

**Query**：查询下载数据库中下载状态

接下来我们来看看如何使用：首先我们需要从系统服务中获取这个DownloadManager：

```
DownloadManager downloadManger = (DownloadManager) context.getSystemService(Context.DOWNLOAD_SERVICE);
```

然后实例化一个Request：

```
Request request = new Request(Uri.parse(uriString))
```

最后将这个请求放入下载堆栈中即可：

```
long downloadId = downloadManger.enqueue(request);
```

然后系统就会自动下载你的请求了。

是不是非常简单呢？接下来我们看看完整的代码：

```
DownloadManager downloadManger = (DownloadManager) context.getSystemService(Context.DOWNLOAD_SERVICE);
Request request = new Request(Uri.parse(uriString));
request.setAllowedNetworkTypes(Request.NETWORK_MOBILE | Request.NETWORK_WIFI);
//设置不显示下载通知
request.setShowRunningNotification(false);
request.setVisibleInDownloadsUi(false);
//判断SD卡是否存在
boolean SDCardEnabled = LibIOUtil.getExternalStorageState();
if (SDCardEnabled) {
request.setDestinationInExternalFilesDir(context, AppUtil.getPackageName(context), AsyncTaskLoaderImage.getHash(uriString));
} else {
request.setDestinationUri(null);
}
long downloadId = downloadManger.enqueue(request);
```
那我们怎么知道什么时候这个请求下载完成了呢？其实在下载完成之后系统会给我们发送一个广播：
  
```
BroadcastReceiver receiver = new BroadcastReceiver() {
@Override
public void onReceive(Context context, Intent intent) {
String action = intent.getAction();
if (DownloadManager.ACTION_DOWNLOAD_COMPLETE.equals(action)) {
long downloadId = intent.getLongExtra(DownloadManager.EXTRA_DOWNLOAD_ID, 0);
DownloadManager.Query query = new DownloadManager.Query();
query.setFilterById(downloadId);
Cursor c = dm.query(query);
if (c.moveToFirst()) {
int columnIndex = c.getColumnIndex(DownloadManager.COLUMN_STATUS);
if (DownloadManager.STATUS_SUCCESSFUL == c.getInt(columnIndex)) {
String uriString = 
c.getString(c.getColumnIndex(DownloadManager.COLUMN_LOCAL_URI));
}
}
}
}
};
registerReceiver(receiver, new IntentFilter(DownloadManager.ACTION_DOWNLOAD_COMPLETE));
```
同时我们也可以在任何时候来查询状态，并且查找的时候缩小范围，比如我们只想查找下载完成的：
  
```
query.setFilterByStatus(DownloadManager.STATUS_SUCCESSFUL);
```

或者暂停的、失败的、等待的、正在下载的。

综上，使用此类实现一个自己的下载管理器是十分方便的有木有！不用自己管理数据库等一系列繁琐的操作。

续：不过令人遗憾的是：在测试过程中使用三星GT-I9500的时候使用此API出现了崩溃。解决不了！为什么这么肯定呢？因为我TM点他系统的下载管理APP都崩了好吗？不知道是个别机器的原因还是什么，暂记吧~