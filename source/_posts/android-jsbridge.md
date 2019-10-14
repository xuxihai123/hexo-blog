---
title: 基于android混合开发的JsBridge技术学习
---

## 无JsBridge
 -  js如果要调用java(native、也说原生)方法:则java必须实现js接口供挂载在window对象上供js来执行。这里简单的可以只调用，调用后java端什么也不做。复杂的调用可以是js调用了java端，然后java用loadUrl("javascript:")继续调用js，这里的调用可以是上一个js调用java的回调，也可以是一个纯js，也可以是下一个js调用java。
 -  java调用js就简单多了,都是用的loadUrl("javascript:XXX")。这里也可以在js串加入调用java作为回调。
 
## 基于JsBridge
 -  java(native)调用js,方法还是loadUrl,但是jsbridge在这个基础上封装了下,封装成了callHandler(String handlerName, String data, CallBackFunction callBack) 
 -  js调用java就不用接口了，而是使用一个隐藏的iframe。通过变更src路径，java端webview上拦截这个变更来实现js调用java.
 -  src:https://github.com/lzyzsd/JsBridge
 
JsBridge example:
 ```java
webView.callHandler("functionInJs", "data from Java",
    new CallBackFunction() {

        @Override
        public void onCallBack(String data) {
            // TODO Auto-generated method stub
            Log.i(TAG, "reponse data from js " + data);
        }

    });
```

这里三个参数，第三个是作为回调的，成功后会调用这个对象里的onCallBack方法。

```java
public void callHandler(String handlerName, String data, CallBackFunction callBack) {
    doSend(handlerName, data, callBack);
}
```
直接跳到doSend方法
 ```java
private void doSend(String handlerName, String data, CallBackFunction responseCallback) {
        Message m = new Message();
        if (!TextUtils.isEmpty(data)) {
            m.setData(data);
        }
        if (responseCallback != null) {
            String callbackStr = String.format(BridgeUtil.CALLBACK_ID_FORMAT, ++uniqueId + (BridgeUtil.UNDERLINE_STR + SystemClock.currentThreadTimeMillis()));
            responseCallbacks.put(callbackStr, responseCallback); //responseCallbacks数组存回调对象
            m.setCallbackId(callbackStr);  //数据塞在message对象中
        }
        if (!TextUtils.isEmpty(handlerName)) {  //handler不为空就把他设置给message对象
            m.setHandlerName(handlerName);
        }
        queueMessage(m);
    }
```
创建message对象并queueMessage处理,   回调对象入responseCallbacks(Map)

```java
private void queueMessage(Message m) {
    if (startupMessage != null) { //startupMessage不为空的时候
        startupMessage.add(m);//message对象加入列表
    } else {
        dispatchMessage(m); //这里派遣(发出)信息--向js
    }
}
```
这里startupMessage是否为空呢？看这里:

```java
@Override
public void onPageFinished(WebView view, String url) {
    super.onPageFinished(view, url);

    if (toLoadJs != null) {
        BridgeUtil.webViewLoadLocalJs(view, toLoadJs);
    }

    //
    if (startupMessage != null) {
        for (Message m : startupMessage) {
            dispatchMessage(m);
        }
        startupMessage = null;
    }
}
```
这个方法是在webview加载完后执行，必然会把startupMessage置为null,那么到了
dispatchMessage这个方法...   英文意思是派遣消息,是？

```java
private void dispatchMessage(Message m) {
    String messageJson = m.toJson();
    //escape special characters for json string
    messageJson = messageJson.replaceAll("(\\\\)([^utrn])", "\\\\\\\\$1$2");
    messageJson = messageJson.replaceAll("(?<=[^\\\\])(\")", "\\\\\"");
   // JS_HANDLE_MESSAGE_FROM_JAVA=="javascript:WebViewJavascriptBridge._handleMessageFromNative('%s');"
   //javascriptCommand==javascript:WebViewJavascriptBridge._handleMessageFromNative('{\"data\":\"data from Java\",\"handlerName\":\"functionInJs\",\"callbackId\":\"JAVA_CB_3_2640\"}');
    String javascriptCommand = String.format(BridgeUtil.JS_HANDLE_MESSAGE_FROM_JAVA, messageJson);
    if (Thread.currentThread() == Looper.getMainLooper().getThread()) {
        this.loadUrl(javascriptCommand);  //java这里主线程执行js
    }
}
```

把message对象转为json处理，BridgeUtil里面声明了final static String JS_HANDLE_MESSAGE_FROM_JAVA = "javascript:WebViewJavascriptBridge._handleMessageFromNative('%s');"; 最终处理后需要loadUrl的字符串如上，我们可以知道最终会调用_handleMessageFromNative方法处理，并带上json串：{\"data\":\"data from Java\",\"handlerName\":\"functionInJs\",\"callbackId\":\"JAVA_CB_3_2640\"}
现在来看这个js方法：// 提供给native调用,receiveMessageQueue 在会在页面加载完后赋值为null,所以

```js
function _handleMessageFromNative(messageJSON) {
    if (receiveMessageQueue) {
        receiveMessageQueue.push(messageJSON);
    } else {
        _dispatchMessageFromNative(messageJSON);
    }
}
```
 作者提供的代码说了，这个跟startMessage一样,receiveMessageQueue会在页面加载完后置为null.继续_dispatchMessageFromNative
// 提供给native使用,

```js
function _dispatchMessageFromNative(messageJSON) {  //native -doSend..后loadUrl(js这个方法)-获取json数据
        setTimeout(function() {
            var message = JSON.parse(messageJSON);
            var responseCallback;
            // java call finished, now need to call js callback function
            if (message.responseId) {  //没有responseId to else
                responseCallback = responseCallbacks[message.responseId];
                if (!responseCallback) {
                    return;
                }
                responseCallback(message.responseData);
                delete responseCallbacks[message.responseId];
            } else {
                // 直接发送
                if (message.callbackId) {
                    var callbackResponseId = message.callbackId;
                    responseCallback = function(responseData) {
                        _doSend({
                            responseId : callbackResponseId, //响应javaId:JAVA_CB_3_2640
                            responseData : responseData
                        });
                    };
                }

                var handler = WebViewJavascriptBridge._messageHandler;
                if (message.handlerName) {
                    handler = messageHandlers[message.handlerName];//handler名是functionJs
                }
                // 查找指定handler
                try {
                    handler(message.data, responseCallback);
                } catch (exception) {
                    if (typeof console != 'undefined') {
                        console
                                .log(
                                        "WebViewJavascriptBridge: WARNING: javascript handler threw.",
                                        message, exception);
                    }
                }
            }
        });
    }
```

上面的加了注释不难理解，有2句解释下：

handler = messageHandlers[message.handlerName];//handler名是functionJs

这里jsbridge在启动的时候有这段:

```js
bridge.registerHandler("functionInJs", function (data, responseCallback) {
    document.getElementById("show").innerHTML = ("data from Java: = " + data);
    var responseData = "Javascript Says Right back aka!";
    responseCallback(responseData);
});
```
这里会注册这个functioinJs方法:所以上面获得的是这里定义的方法.所以后面的

handler(message.data, responseCallback);

data就是前面java里面封装的data,responseCallBack是上面的

```js
responseCallback = function(responseData) {
    _doSend({
        responseId : callbackResponseId, //响应javaId:JAVA_CB_3_2640
        responseData : responseData
    });
};
```

他是作为数据在id为show的div上显示数据后要回调的方法。现在数据显示出来了，我们知道了，这里的responseId,responseData是上面的一个

```js
 var responseData = "Javascript Says Right back aka!";
```

转到_doSend方法:

```js
// sendMessage add message, 触发native处理 sendMessage
function _doSend(message, responseCallback) {
    if (responseCallback) {
        var callbackId = 'cb_' + (uniqueId++) + '_' + new Date().getTime();
        responseCallbacks[callbackId] = responseCallback;//这里在js端存储JAVA_CB_3_2640作为js  callbackId的属性
        message.callbackId = callbackId;//待发送的信息里面存储了一个刚才生成的js  callbackId
    }

    sendMessageQueue.push(message); //把这个message放入sendMessageQueue数组中.
    messagingIframe.src = CUSTOM_PROTOCOL_SCHEME + '://'
            + QUEUE_HAS_MESSAGE;   //这里变更iframe的src属性,这样会触发Java端WebViewClient的shouldOverrideUrlLoading方法执行
}
```

下面转到shouldOverrideUrlLoading方法.

```java
public boolean shouldOverrideUrlLoading(WebView view, String url) {
        try {
            url = URLDecoder.decode(url, "UTF-8");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        if (url.startsWith(BridgeUtil.YY_RETURN_DATA)) { // 如果是返回数据 _fetchQueue
            handlerReturnData(url);
            return true;
        } else if (url.startsWith(BridgeUtil.YY_OVERRIDE_SCHEMA)) { //第一次 _doSend/////
            flushMessageQueue();
            return true;
        } else {
            return super.shouldOverrideUrlLoading(view, url);
        }
    }
```
....