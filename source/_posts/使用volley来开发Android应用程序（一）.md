title: 使用volley来开发Android应用程序（一）
date: 2014-11-16 07:49:14
tags: [Android]
---
### 什么是volley

volley是Google IO 2003年官方推荐的一个能让HTTP请求更快速、更方便的开源库。volley能让我们写更少更精炼的代码实现强壮的代码，它执行任务默认都是异步的，所以我们不必担心主线程阻塞等问题了。volley有很多实用的功能，比如请求缓存、优化的请求队列和取消任务功能、图片双缓存、还非常方便我们针对自己的项目需求进行拓展。目前该库托管在[Google git](https://android.googlesource.com/platform/frameworks/volley)上。

### 向项目中添加volley

我们可以通过项目引入源码库或者jar包的方式来使用volley，现在简要说明一下第二种方式：

- 安装git和ant（推荐使用brew）

```
$ brew install git
$ brew install ant
```

在安装好brew的前提下分别执行来进行安装。当然，也可以通过安装git客户端和ant包来实现。

- 下载volley source code

```
$ git clone https://android.googlesource.com/platform/frameworks/volley
```

执行上面的命令来克隆一份最新的代码到自己本地，然后再执行

```
$ android update project -p .
```

来生成我们编译需要的文件，然后执行

```
ant jar
```
成功之后就能再bin目录下看到我们的volley.jar了。（关于编译方式我们也可以选择Android Studio默认的Gradle方式，在此就不说明了）。

### 如何使用volley

接下来将volley.jar添加到我们的项目之后我们来看看怎么使用，这里先介绍下我们即将用到的几个对象：`RequestQueue`、`Request`（`StringRequest`、`ImageRequest`、`JsonRequest`（`JsonObjectRequest`、`JsonArrayRequest`））。

`RequestQueue`是请求队列对象，这个队列并不需要我们来维护，我们只需要将`Request`添加到这个队列里面去就行了，非常简单。`Request`包含常见的几种请求：字符串、图片、以及JSON，分别对应相应的对象。

接下来我们再看看volley内部的执行流程是怎么样的呢？

![](http://pic.yupoo.com/lvning10086/EdacCEc1/Y4efi.png)

从上面的图看以看到分别有三种线程模型，分别是主线程、缓存线程和网络请求线程。当我们添加一个请求到队列中的时候，volley会先检查是否存在缓存，存在的话就直接返回给主线程，如果不存在的话就交给请求线程来处理，在请求队列中成功或失败后再返回给主线程处理。

了解了内部流程之后，我们先来简单的看看怎么发送一个简单的请求：

    final TextView mTextView = (TextView) findViewById(R.id.text);
    ...
    // Instantiate the RequestQueue.
    RequestQueue queue = Volley.newRequestQueue(this);
    String url ="http://www.google.com";
    // Request a string response from the provided URL.
    StringRequest stringRequest = new StringRequest(Request.Method.GET, url,
    new Response.Listener(){
    	@Override
    	public void onResponse(String response){
    		// Display the first 500 characters of the response string.
    		mTextView.setText("Response is: "+ response.substring(0,500));
    	}
    }, new Response.ErrorListener(){
    	@Override
        public void onErrorResponse(VolleyError error){
        	mTextView.setText("That didn't work!");
        	}
        });
    // Add the request to the RequestQueue.
    queue.add(stringRequest);
    
可以看到，我们通过`Volley.newRequestQueue(this);`方法实例化一个默认的请求队列，然后再传入参数实例化一个请求，最后将这个请求对象添加到我们的队列里面去，然后就在回调函数里面处理请求结果就行了。

但是如何取消一个任务执行呢？volley使用`Cancel`方法来取消一个请求，不过在这之前，我们得先给这个请求设置一个Tag才行，然后再在想要取消任务的时候调用`cancel`即可。简要代码如下：

    public static final String TAG = "MyTag";
    StringRequest stringRequest; // Assume this exists.
    RequestQueue mRequestQueue; // Assume this exists.
    // Set the tag on the request.
    stringRequest.setTag(TAG);
    // Add the request to the RequestQueue.
    mRequestQueue.add(stringRequest);
    
然后再取消（比如在`onStop()`中）：

    @Override
    protected void onStop (){
    	super.onStop();
    	if(mRequestQueue != null){
    		mRequestQueue.cancelAll(TAG);
    	}
    }
    
取消任务也非常简单吧。在下一篇文章我们再解释如果自己初始化一个请求队列，而不是使用`Volley.newRequestQueue(this);`方法生成的默认队列。