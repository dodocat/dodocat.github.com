---
layout: post
title: Android Volley 实践
date: 2014-03-12 19:46
comments: true
categories: Android
tags: android volley
---

# Volley
> 这是一篇在团队内部分享演示的讲稿的整理.

> Volley is a library that makes networking for Android apps easier and most importantly, faster.

## Volley 的使用场景
大量的信息流, 大量的图片.
> 从网络上读取文本, 属性, 图片, 然后把他们放到 UI 上去.
Advantages of using Volley:


## Advantages of using Volley:
* 自动管理所有网络请求.
* 透明的换内存和存储缓存
* 队列易于维护, 所有请求可以随时取消.
* 自定义
* debug

## ABC Volley
1. 建立一个 Request Quene.
2. 创建一个 Request.
3. 在 RequestListner 里处理返回结果.
4. 把 Request 加入队列.
> __演示__



## 如何使用 volley

本文中会尝试介绍以下内容, 最后应该能够大致明白怎么在程序中使用 volley


* 在工程中使用 volley
* 队列 Queue
* 请求 Request
* Making asynchronous JSON and String HTTP requests
* 取消请求
* 请求失败重试
* Header 信息
* 错误处理


### Installing and using Volley as a library project


``` bash
$ git clone https://android.googlesource.com/platform/frameworks/volley
```


### Using Request Queue

Volley 中所有的请求在发出之前要先进入一个队列.


``` java
RequestQueue mRequestQueue = Volley.newRequestQueue(this); // 'this' is Context
```

实际应用中通常在 Application 中维护这个队列


``` java

public class ApplicationController extends Application {

    /**
     * Log or request TAG
     */
    public static final String TAG = "VolleyPatterns";

    /**
     * Global request queue for Volley
     */
    private RequestQueue mRequestQueue;

    /**
     * A singleton instance of the application class for easy access in other places
     */
    private static ApplicationController sInstance;

    @Override
    public void onCreate() {
        super.onCreate();

        // initialize the singleton
        sInstance = this;
    }

    /**
     * @return ApplicationController singleton instance
     */
    public static synchronized ApplicationController getInstance() {
        return sInstance;
    }

    /**
     * @return The Volley Request queue, the queue will be created if it is null
     */
    public RequestQueue getRequestQueue() {
        // lazy initialize the request queue, the queue instance will be
        // created when it is accessed for the first time
        if (mRequestQueue == null) {
            mRequestQueue = Volley.newRequestQueue(getApplicationContext());
        }

        return mRequestQueue;
    }

    /**
     * Adds the specified request to the global queue, if tag is specified
     * then it is used else Default TAG is used.
     *
     * @param req
     * @param tag
     */
    public <T> void addToRequestQueue(Request<T> req, String tag) {
        // set the default tag if tag is empty
        req.setTag(TextUtils.isEmpty(tag) ? TAG : tag);

        VolleyLog.d("Adding request to queue: %s", req.getUrl());

        getRequestQueue().add(req);
    }

    /**
     * Adds the specified request to the global queue using the Default TAG.
     *
     * @param req
     * @param tag
     */
    public <T> void addToRequestQueue(Request<T> req) {
        // set the default tag if tag is empty
        req.setTag(TAG);

        getRequestQueue().add(req);
    }

    /**
     * Cancels all pending requests by the specified TAG, it is important
     * to specify a TAG so that the pending/ongoing requests can be cancelled.
     *
     * @param tag
     */
    public void cancelPendingRequests(Object tag) {
        if (mRequestQueue != null) {
            mRequestQueue.cancelAll(tag);
        }
    }
}

```

### Making asynchronous HTTP requests

volley 提供了下面几种 Request. 实际应用中需要根据自己需求定义 Request.

* [JsonObjectRequest](http://goo.gl/CRMvRj) -- To send and receive JSON Object from the Server
* [JsonArrayRequest](http://goo.gl/F02Ew3) -- To receive JSON Array from the Server
* [StringRequest](http://goo.gl/c5DB8p) -- To retrieve response body as String (ideally if you intend to parse the response by yourself)



#### JsonObjectRequest
用于发送和接受 JSON 对象.


Using HTTP GET method:

``` java
final String URL = "/volley/resource/12";
// pass second argument as "null" for GET requests
JsonObjectRequest req = new JsonObjectRequest(URL, null,
         new Response.Listener<JSONObject>() {
	         @Override
	         public void onResponse(JSONObject response) {
	             try {
	                 VolleyLog.v("Response:%n %s", response.toString(4));
	             } catch (JSONException e) {
	                 e.printStackTrace();
	             }
	         }
	     }, new Response.ErrorListener() {
	         @Override
	         public void onErrorResponse(VolleyError error) {
	             VolleyLog.e("Error: ", error.getMessage());
	         }
	     });

// add the request object to the queue to be executed
ApplicationController.getInstance().addToRequestQueue(req);
```

Using HTTP POST method:

``` java
final String URL = "/volley/resource/12";
// Post params to be sent to the server
HashMap<String, String> params = new HashMap<String, String>();
params.put("token", "AbCdEfGh123456");

JsonObjectRequest req = new JsonObjectRequest(URL, new JSONObject(params),
	     new Response.Listener<JSONObject>() {
	         @Override
	         public void onResponse(JSONObject response) {
	             try {
	                 VolleyLog.v("Response:%n %s", response.toString(4));
	             } catch (JSONException e) {
	                 e.printStackTrace();
	             }
	         }
	     }, new Response.ErrorListener() {
	         @Override
	         public void onErrorResponse(VolleyError error) {
	             VolleyLog.e("Error: ", error.getMessage());
	         }
	     });

// add the request object to the queue to be executed
ApplicationController.getInstance().addToRequestQueue(req);
```





#### StringRequest

这是简单的接收返回 字符串的 Request. 往往用于处理非标准固定格式的数据.

``` java
final String URL = "/volley/resource/recent.xml";
StringRequest req = new StringRequest(URL, new Response.Listener<String>() {
    @Override
    public void onResponse(String response) {
        VolleyLog.v("Response:%n %s", response);
    }
}, new Response.ErrorListener() {
    @Override
    public void onErrorResponse(VolleyError error) {
        VolleyLog.e("Error: ", error.getMessage());
    }
});

// add the request object to the queue to be executed
ApplicationController.getInstance().addToRequestQueue(req);
```

### 设置优先级
* ```Priority.LOW```
* ```Priority.NORMAL```
* ```Priority.HIGH```
* ```Priority.IMMEDIATE```

### Cancelling requests

给每个 Request 添加一个相关的 tag, 一个比较好的实践是添加相关的 activity.

``` java
request.setTag("My Tag");
```

``` java
ApplicationController.getInstance().addToRequestQueue(request, "My Tag");
```
使用 tag 取消请求
``` java
mRequestQueue.cancelAll("My Tag");
```


### 失败请求重试和自定义超时时间
* 超时时间
* 重试次数
* exponential backoff
``` java
request.setRetryPolicy(new DefaultRetryPolicy(20 * 1000, 1, 1.0f));
```


### 设置 Headers (HTTP headers)


### 错误处理

As you have seen in the above code examples when you create a request object in Volley you need to specify  an error listener, Volley invokes the ```onErrorResponse``` callback method of that listener passing an instance of the ```VolleyError``` object when there is an error while performing the request.

The following is the list of exceptions in Volley:

* **AuthFailureError** -- If you are trying to do Http Basic authentication then this error is most likely to come.
* **NetworkError** -- Socket disconnection, server down, DNS issues might result in this error.
* **NoConnectionError** -- Similar to NetworkError, but fires when device does not have internet connection, your error handling logic can club ```NetworkError``` and ```NoConnectionError``` together and treat them similarly.
* **ParseError** -- While using ```JsonObjectRequest``` or ```JsonArrayRequest``` if the received JSON is malformed then this exception will be generated. If you get this error then it is a problem that should be fixed instead of being handled.
* **ServerError** -- The server responded with an error, most likely with _4xx_ or _5xx_ HTTP status codes.
* **TimeoutError** -- Socket timeout, either server is too busy to handle the request or there is some network latency issue. By default Volley times out the request after **2.5 seconds**, use a RetryPolicy if you are consistently getting this error.


下面是一个帮助处理错误的工具

``` java
public class VolleyErrorHelper {
     /**
     * Returns appropriate message which is to be displayed to the user
     * against the specified error object.
     *
     * @param error
     * @param context
     * @return
     */
	public static String getMessage(Object error, Context context) {
		if (error instanceof TimeoutError) {
			return context.getResources().getString(R.string.generic_server_down);
		}
		else if (isServerProblem(error)) {
			return handleServerError(error, context);
		}
		else if (isNetworkProblem(error)) {
			return context.getResources().getString(R.string.no_internet);
		}
		return context.getResources().getString(R.string.generic_error);
	}

	/**
	 * Determines whether the error is related to network
	 * @param error
	 * @return
	 */
	private static boolean isNetworkProblem(Object error) {
	    return (error instanceof NetworkError) || (error instanceof NoConnectionError);
	}
	/**
	 * Determines whether the error is related to server
	 * @param error
	 * @return
	 */
	private static boolean isServerProblem(Object error) {
	    return (error instanceof ServerError) || (error instanceof AuthFailureError);
	}
	/**
	 * Handles the server error, tries to determine whether to show a stock message or to
	 * show a message retrieved from the server.
	 *
	 * @param err
	 * @param context
	 * @return
	 */
	private static String handleServerError(Object err, Context context) {
	    VolleyError error = (VolleyError) err;

	    NetworkResponse response = error.networkResponse;

	    if (response != null) {
	        switch (response.statusCode) {
            case 404:
            case 422:
            case 401:
                try {
                	  // server might return error like this { "error": "Some error occured" }
                	  // Use "Gson" to parse the result
                    HashMap<String, String> result = new Gson().fromJson(new String(response.data),
                            new TypeToken<Map<String, String>>() {
                            }.getType());

                    if (result != null && result.containsKey("error")) {
                        return result.get("error");
                    }

                } catch (Exception e) {
                    e.printStackTrace();
                }
                // invalid request
                return error.getMessage();

            default:
                return context.getResources().getString(R.string.generic_server_down);
            }
	    }
        return context.getResources().getString(R.string.generic_error);
	}
}
```


## References

* NetworkOnMainThreadException -- http://developer.android.com/reference/android/os/NetworkOnMainThreadException.html
* Application -- http://developer.android.com/reference/android/app/Application.html
