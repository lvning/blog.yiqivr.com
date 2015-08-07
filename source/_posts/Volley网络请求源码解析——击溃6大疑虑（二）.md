title: Volley网络请求源码解析——击溃6大疑虑（二）
date: 2015-01-08 10:07:42
tags: [Android]
---
### 关于Volley的缓存？

缓存，Volley是通过Response中的Header来设置缓存记录的生命周期的，主要体现在HttpHeaderParser.java文件的parseCacheHeaders()方法中。所以如果后台没有返回cache-control一系列信息的话，我们每次请求其实都是去网络重新请求的，因为Cache.Entry中的ttl（控制缓存存在于磁盘的时间）、softTtl（控制缓存后台刷新时间）属性都是默认的0。如果我们不想要后台来控制怎么办呢？看来我们得需要自己来实现一个类似parseCacheHeaders()的方法，比如：

```
/**
 * Extracts a {@link Cache.Entry} from a {@link NetworkResponse}.
 * Cache-control headers are ignored. SoftTtl == 3 mins, ttl == 24 hours.
 * @param response The network response to parse headers from
 * @return a cache entry for the given response, or null if the response is not cacheable.
 */
 public static Cache.Entry parseIgnoreCacheHeaders(NetworkResponse response) {
 long now = System.currentTimeMillis();
 Map&lt;String, String&gt; headers = response.headers;
 long serverDate = 0;
 String serverEtag = null;
 String headerValue;
 headerValue = headers.get("Date");
 if (headerValue != null) {
 serverDate = HttpHeaderParser.parseDateAsEpoch(headerValue);
 }
 serverEtag = headers.get("ETag");
 final long cacheHitButRefreshed = 3 * 60 * 1000; // in 3 minutes cache will be hit, but also refreshed on background
 final long cacheExpired = 24 * 60 * 60 * 1000; // in 24 hours this cache entry expires completely
 final long softExpire = now + cacheHitButRefreshed;
 final long ttl = now + cacheExpired;
 Cache.Entry entry = new Cache.Entry();
 entry.data = response.data;
 entry.etag = serverEtag;
 entry.softTtl = softExpire;
 entry.ttl = ttl;
 entry.serverDate = serverDate;
 entry.responseHeaders = headers;
 return entry;
 }
 ```
 然后使用的时候：
 
 ```
 public class MyRequest extends com.android.volley.Request<MyResponse> {
 ...
 @Override
 protected Response<MyResponse> parseNetworkResponse(NetworkResponse response) {
 String jsonString = new String(response.data);
 MyResponse MyResponse = gson.fromJson(jsonString, MyResponse.class);
 return Response.success(MyResponse, HttpHeaderParser.parseIgnoreCacheHeaders(response));
 }
 }
 ```
 
这样的话我们就固定了缓存策略，如果在3分钟内获取到缓存，后台会自动刷新缓存，缓存在24小时后将被自动清除。
 

### 如何自动清除Volley特定请求的缓存？
 
首先，要清除Volley的特定的缓存，主要是两个方法：public void invalidate(String key, boolean fullExpire);和public void remove(String key);remove方法代表彻底移除该条缓存记录，invalidate表示将ttl和softy属性都置0，接下来我们还可以得到获取到数据时的时间：Cache.Entry.SeverDate;有了这些条件，我们就可以定义两个方法：

```
public static long getMinutesDifference(long timeStart,long timeStop){
long diff = timeStop - timeStart;
long diffMinutes = diff / (60 * 1000);
return diffMinutes;
}
```
然后调用就OK了：

```
Calendar calendar = Calendar.getInstance();
long serverDate = AppController.getInstance().getRequestQueue().getCache().get(url).serverDate;
if(getMinutesDifference(serverDate, calendar.getTimeInMillis()) >=30){
AppController.getInstance().getRequestQueue().getCache().invalidate(url, true);
}
```

### 如何缓存POST请求的数据？

通过源码我们可以看到，Volley存储缓存的key就是我们请求的URL，那POST怎么办呢？URL可都是一样的啊，想一想，我们可以自己定义一个Request，然后重写 public String getCacheKey()方法不就行了嘛，比如在URL后面加上请求参数的组拼MD5值。

### 如何检查我们得到的数据是来自缓存还是网络？

Volley里面有个MarkerLog，默认是开启的，它能方便的让我们查看一个请求的生命周期，其中就包括从缓存获取数据的信息，因此我们可以取巧得到这个信息，我们继承Request：

```
public class MyReques<T> extends Request&lt;T&gt; {
protected boolean cacheHit;
@Override
public void addMarker(String tag) {
super.addMarker(tag);
cacheHit = false;
if (tag.equals("cache-hit")){
cacheHit = true;
}
}
}
```
我们添加一个cacheHit变量，通过源码得知：当Volley从缓存获取到数据之后会添加cache-hit信息，相应的，我们就知道他从缓存中获取到了数据。

### 如何同步的执行一个请求？

Volley默认请求都是异步的，请求网络共有4条线程组成的线程池，还有一条缓存线程，关于请求，Volley实现了一个叫做RequestFuture的类，我们可以利用他来实现同步请求，比如接下来我们创建一个同步的JSON HTTP GET：

```
RequestFuture<JSONObject> future = RequestFuture.newFuture();
JsonObjectRequest request = new JsonObjectRequest(URL, null, future, future);
requestQueue.add(request);
try {
JSONObject response = future.get(); // this will block
} catch (InterruptedException e) {
// exception handling
} catch (ExecutionException e) {
// exception handling
}
```

### 如何上传文件？

默认的，Volley并没有为我们实现上传文件的功能，但是我们可以自定义一个Request来实现：

```
public MultipartRequest(String url, Response.ErrorListener errorListener, Response.Listener<String> listener, File file, String stringPart)
{
super(Method.POST, url, errorListener);
mListener = listener;
mFilePart = file;
mStringPart = stringPart;
buildMultipartEntity();
}
private void buildMultipartEntity()
{
entity.addPart(FILE_PART_NAME, new FileBody(mFilePart));
try
{
entity.addPart(STRING_PART_NAME, new StringBody(mStringPart));
}
catch (UnsupportedEncodingException e)
{
VolleyLog.e("UnsupportedEncodingException");
}
}@Override
public String getBodyContentType()
{
return entity.getContentType().getValue();
}
@Override
public byte[] getBody() throws AuthFailureError
{
ByteArrayOutputStream bos = new ByteArrayOutputStream();
try
{
entity.writeTo(bos);
}
catch (IOException e)
{
VolleyLog.e("IOException writing to ByteArrayOutputStream");
}
return bos.toByteArray();
}
@Override
protected Response<String> parseNetworkResponse(NetworkResponse response)
{
return Response.success("Uploaded", getCacheEntry());
}
@Override
protected void deliverResponse(String response)
{
mListener.onResponse(response);
}
```