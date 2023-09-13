### 一、Webview基础知识  

![[Webview_overview.png]]

Injects the supplied Java object into this WebView. The object is injected into all frames of the web page, including all the iframes, using the supplied name. This allows the Java object's methods to be accessed from JavaScript. For applications targeted to API level `[Build.VERSION_CODES.JELLY_BEAN_MR1]` and above, only public methods that are annotated with `JavascriptInterface` can be accessed from JavaScript. For applications targeted to API level `Build.VERSION_CODES.JELLY_BEAN]` or below, all public methods (including the inherited ones) can be accessed, see the important security note below for implications.

Note that injected objects will not appear in JavaScript until the page is next (re)loaded. JavaScript should be enabled before injecting the object. For example:


Webview初始化:

console.log(JSON.stringify(result))
let r = document.getElementById('r')
r.innerHTML = JSON.stringify(result)

第一种方式：`new WebView`对象，最后`setContentView`才能将界面显示出来
```java
    class JsObject {  
        @JavascriptInterface  
        public String getToken() { return "{\"token\":\"1234567890abcdefg\"}"; }  
    }  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState); 
        setContentView(R.layout.activity_main);  
        WebView webView =  new WebView(this);  
        webView.getSettings().setJavaScriptEnabled(true);  
        webView.setWebViewClient(new WebViewClient());  
        webView.setWebChromeClient(new WebChromeClient());   
        webView.addJavascriptInterface(new JsObject(),"myObj");  
        webView.loadUrl("https://www.feymanpaper.asia/");  
        setContentView(webView);  
    }
```

第二种方式：`setContentView`一个界面，然后将Webview绑定到某一个webview组件
```java
    class JsObject {  
        @JavascriptInterface  
        public String getToken() { return "{\"token\":\"1234567890abcdefg\"}"; }  
    }  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);
        WebView webView = findViewById(R.id.mywebview);  
        webView.getSettings().setJavaScriptEnabled(true);  
        webView.setWebViewClient(new WebViewClient());  
        webView.setWebChromeClient(new WebChromeClient());  
        webView.addJavascriptInterface(new JsObject(),"myObj");  
        webView.loadUrl("https://www.feymanpaper.asia/");  
    }
```
Chrome的架构：**1 个浏览器（Browser）主进程、1 个GPU 进程、1 个网络（NetWork）进程、多个渲染进程和多个插件进程**. Glossing over A LOT of details. Chrome desktop has lots of renderers, utility processes for services.
![[chrome_arch_s.png]]

Chrome for Android runs services in the browser process instead of utility processes (low memory devices only).
![[chrome_arch_lowmemory.png]]

Webview的架构(L-N, API 19-25): All in the app’s process! For technical reasons
![[webview_arch_ln.png]]
Webview的架构(O+, API 26+): In-process services (actually, for technical reasons, but also memory) and single renderer (memory considerations)
![[webview_arch_o.png]]

### 二、 Android security checklist: WebView
WebView白名单校验文章
https://blog.oversecured.com/Android-security-checklist-webview/
https://www.cnblogs.com/rebeyond/p/10916076.html
https://www.freebuf.com/articles/endpoint/201407.html

#### shouldOverrideUrlLoading方法-触发时机 
shouldOverrideUrlLoading的官方解释
```dart
  /**
     * Give the host application a chance to take over the control when a new
     * url is about to be loaded in the current WebView. If WebViewClient is not
     * provided, by default WebView will ask Activity Manager to choose the
     * proper handler for the url. If WebViewClient is provided, return true
     * means the host application handles the url, while return false means the
     * current WebView handles the url.
     * This method is not called for requests using the POST "method".
     *
     * @param view The WebView that is initiating the callback.
     * @param url The url to be loaded.
     * @return True if the host application wants to leave the current WebView
     *         and handle the url itself, otherwise return false.
     * @deprecated Use {@link #shouldOverrideUrlLoading(WebView, WebResourceRequest)
     *             shouldOverrideUrlLoading(WebView, WebResourceRequest)} instead.
     */
```
  
- **若没有设置 WebViewClient 则由系统（Activity Manager）处理该 url，通常是使用浏览器打开或弹出浏览器选择对话框。**
-   **若设置 WebViewClient 且该方法返回 true ，则说明由应用的代码处理该 url，WebView 不处理，也就是程序员自己做处理。**
-   **若设置 WebViewClient 且该方法返回 false，则说明由 WebView 处理该 url，即用 WebView 加载该 url。**

  WebView的前进、后退、刷新、以及post请求都不会调用shouldOverrideUrlLoading方法，除去以上行为，还得满足（ ! isLoadUrl || isRedirect） 即 （不是通过webView.loadUrl来加载的 或者 是重定向） 这个条件，才会调用shouldOverrideUrlLoading方法。

####  白名单校验
总结：shouldOverrideUrlLoading方法会在webview后续加载其他url时回调，因此需要在loadUrl之前校验白名单一次，还要在shouldOverrideUrlLoading中再校验白名单.
```
private static boolean checkDomain(String inputUrl) throws URISyntaxException {
if (!inputUrl.startsWith("http://")&&!inputUrl.startsWith("https://")) {
	return false;
}
String[] whiteList=new String[]{"site1.com","site2.com"};
java.net.URI url=new java.net.URI(inputUrl);
String inputDomain=url.getHost(); //提取host，校验Path可以通过url.getPath()获取
for (String whiteDomain:whiteList){
if (inputDomain.endsWith("."+whiteDomain)||inputDomain.equals(whiteDomain))     //www.site1.com app.site2.com
	return true;
}
	return false;
}
```
可以总结为如下几条开发建议： 
* 不要使用indexOf这种模糊匹配的函数；
* 不要自己写正则表达式去匹配； 
* 尽量使用Java封装好的获取域名的方法，比如java.net.URI，不要使用java.net.URL； 
* 不仅要给域名设置白名单，还要给协议设置白名单，一般常用HTTP和HTTPS两种协议，不过强烈建议不要使用HTTP协议，因为移动互联网时代，手机被中间人攻击的门槛很低，搭一个恶意WiFi即可劫持手机网络流量； 
* 权限最小化原则，尽量使用更精确的域名或者路径。 

### 四、Navigation Confused Attack
http://mohamoha.club/2021/05/30/JavascriptInterface_once_more/
https://www.youtube.com/watch?v=rT41G6FNLb4&list=PLmv8T5-GONwS8G-6lKgRdtPo6wGX6xrd1
Bridge between html and native resources:
- Navigation callback
- Javascript Interface
- Javascript Event Handler
- H5 API

#### Navigation callback
Developers have the option of controlling navigation within WebView. Whenever there is a navigation on a WebView, the developer can intercept this or get notification: `shouldOverrideUrlLoading`, `onPageFinished`, `onPageStarted`, `shouldInterceptRequest`. 
![[navigation_callback.png]]

#### Javascript Interface
![[javascript_interface_core.png]]

#### Js Event Handler
The WebView API allows developers to handle the alert, prompt and confirm Javascript events, by registering the `onJsAlert()`, `onJsPrompt()`, `onJsConfirm()` Java callback methods. Whenever the JavaScript side calls any of these event methods, their respective handler will be called, if it is overridden. The developer is free to implement any logic in these event handlers.

#### H5 API
The rise of HTML5 has brought in a set of APIs that can give web applications the ability to access device hardware via Javascript.
E.g. Geolocation and getUserMedia, which enable access to GPS and to media devices such as camera and microphone.
Developer needs to make use of onGeolocationShowPrompt(for geolocation), and onPermissionRequest(for media devices) to grant or deny permission to the requests.

#### Risks on bridges
- CVE-201206336, CVE-2014-1939, CVE-2014-7224
- App Clone Attack
- H5 API Abuse
- Javascript Interface Abuse
- Enforcement On Bridges

#### Enforcement On Bridges
1. Lifecycle based access control
	- Proved to be unsafe, can be bypassed with Time-delay attack
![[lifecycle_based_access_control_overview.png]]
![[lifecycle_based_access_control_attack.png]]
![[lifecycle_based_access_control_thread_status.png]]
2. "real-trime" access control
	- In mose case, safe
![[realtime_access_control_overview.png]]
![[realtime_access_control_attack.png]]

然而"real-trime" access control也会引入新的攻击模式: **Navigation Confused Attack**
比较完善的一个 JsBridge 白名单校验方案,在 JsBridge 被调用时，通过 `WebView.getUrl` 实时的获取当前 WebView 的 URL，并以此做白名单校验。这套方案本身比较完善，可以防御很多此类的攻击。但是由于开发人员对 WebView 的本质理解不到位，即使是这套方案，在实际使用时也存在被绕过的风险

![[check_whitelist.png]]
根据上文的描述，我们可以知道目前的白名单校验机制**完全依赖**于 `WebView.getUrl`，开发者通常会完全信任这个函数返回的结果，而依据此做白名单判断。
但是事实上 **`WebView.getUrl` 返回的值并不总是当前正在执行的 URL** ！查看 WebView 中 getUrl 的定义，注释中明确指出该函数获取的只是 `visible URL`。有可能不是当前正在运行的 url，有可能是一个正在加载的 url。

Tagged getUrl: During different types of navigation, WebView.getUrl will return different value.
Browser initiative navigation: return pending entry
	- Omnibox, bookmarks, context menus, etc.
Render initiative navigation: return last committed entry
	- Links, forms, scripts
	- **Less trustworthy**: bad web pages can try to send you places, but not internal pages
![[browser_vs_render_initiated_navigation.png]]
![[browser_vs_hybrid_app.png]]
**In Hybrid app browser-initiate-navigation can alse be invoked by render model with Bridges.**
	- Render can invoke Browser-initiated-navigation by JavascriptInterface

三种漏洞模式：
- Direct navigation confused vulnerability：应用的开发者直接将 `WebWiew.loadUrl` 暴露给不可信的 js 代码，攻击者就可以通过 js 直接触发 browser-initiate-navigation，修改 WebView 中的 `pending_entry_` ，使 `WebWiew.getUrl` 返回错误的值，绕过白名单校验
-  ReDirect navigation confused vulnerability：在 WebView 的 shouldOverrideUrlLoading、 onJsPrmote 等生命周期回调函数中，以直接或者间接的形式提供 `WebWiew.loadUrl` 的调用路径。 攻击者就可以通过 js 触发生命周期回调的方式触发 browser-initiate-navigation，修改 WebView 中的 `pending_entry_` ，使 `WebWiew.getUrl` 返回错误的值，绕过白名单校验。
-  Shared Navigation Confused Attack：WebView 常用的加载页面方式只有 `WebWiew.loadUrl` 一种，因此通常情况下业务都是使用这个方式加载页面。 如果加载页面的 WebView 存在复用的场景，也会触发这个问题。举例来说，业务通过 scheme 暴露了一个外部页面加载能力，外部通过链接 `huaweilalala://toWebVIew/?url=http:xxxxxxxxx` 就可以拉起组件 `WebviewLoadActivity`，并渲染页面 `http:xxxxxxxxx`

Mitigation:
	- NoFrak
	- Draco



