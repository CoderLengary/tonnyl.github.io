---
layout: post # needs to be post
title: 提升体验 - 支持 Chrome Custom Tabs
summary: 使用Chrome Custom Tabs加载网页可以有效的提升用户的体验.
featured-img: shane-rounce-205187
categories: [Android]
---

## 前言
文章比较长，先放项目地址: [PaperPlane](https://github.com/TonnyL/PaperPlane). 实际效果:

![PaperPlane-1.gif](https://i.loli.net/2018/03/27/5ab9e66d1f7db.gif)
![PaperPlane-2.gif](https://i.loli.net/2018/03/27/5ab9e66d2a23d.gif)

## 什么是Custom Tabs?
所有的APP开发者都面临这样一个选择，当用户点击一个URL时，是应该用浏览器打开还是应该用应用内置的WebView打开呢？

两个选项都面临着一些问题。通过浏览器打开是一个非常重的上下文切换，并且是无法定制的。而WebView不能和浏览器共享数据并且需要需要手动去处理更多的场景。

Chrome Custom Tabs让APP在进行网页浏览时更多的控制权限，在不采用WebView的情况下，这既能保证Native APP和网页之间流畅的切换，又允许APP定制Chrome的外观和操作。可定义的内容如下：

+ toolbar的颜色
+ 进场和退场动画
+ 给Chrome的toolbar、overflow menu和bottom toolbar添加自定义操作

并且，Chrome Custom Tabs允许开发者对Chrome进行预启动和网页内容的预加载，以此提升加载的速度。

![Custom-Tabs.gif](https://i.loli.net/2018/03/27/5ab9e6b607a7b.gif)

## Chrome Custom Tabs VS WebView, 我应该什么时候用？
如果页面的内容是由我们自己控制的，可以和Android组件进行交互，那么,WebView是一个好的选择，如果我们的应用需要打开外部的网站，那么推荐使用Chrome Custom Tabs，原因如下：

+ 导入非常简单。不需要编写额外的代码来管理请求，授予权限或者存储cookie
+ 定制UI：
	- Toolbar 颜色
	- 动作按钮 (Action Button)
	- 定制菜单项
	- 定制进场退场动画
	- Bottom Toolbar 
+ 导航感知：浏览器通知回调接口通知应用网页的导航情况
+ 安全性：浏览器使用了Google’s Safe Browsing，用于保护用户和设备免受危险网站的侵害
+ 性能优化:
	+ 浏览器会在后台进行预热，避免了应用占用大量资源的情况
	+ 提前向浏览器发送可能的URL，提高了页面加载速度
+ 生命周期管理：在用户与Custom Tabs进行交互时，浏览器会将应用标示为前台应用，避免了应用被系统所回收
+ 共享cookie数据和权限，这样，用户在已经授权过的网站，就不需要重新登录或者授权权限了
+ 如果用户开启了数据节省功能，在这里仍然可以从中受益
+ 同步的自动补全功能
+ 仅仅需要点击一下左上角按钮就可以直接返回原应用
+ 想要在Lollipop之前的设备上最新引入的浏览器(Auto updating WebView),而不是旧版本的WebView

## 什么时候可用？
从Chrome 45版本开始，所有的Chrome用户都可以使用这项功能，目前仅支持Android系统。

## 开发指南
完整的示例可以查看 https://github.com/GoogleChrome/custom-tabs-client 。包含了定制 UI 、连接后台服务、处理应用和 Custom Tab Activity 生命周期的可复用的类。

第一步当然是将 [Custom Tabs Support Library](http://developer.android.com/tools/support-library/features.html#custom-tabs) 添加到工程中来。打开 `build.gradle` 文件，添加 support library 的依赖。

```gradle
dependencies {
    ...
    compile 'com.android.support:customtabs:23.3.0'
}
```

一旦Support Library添加项目成功了，我们就有两种可能的定制操作了：

+ 定制UI和与Chrome Custom Tabs的交互
+ 使页面加载更快速，保持应用激活

UI的定制是通过使用 [CustomTabsIntent](http://developer.android.com/reference/android/support/customtabs/CustomTabsIntent.html) 和 [CustomTabsIntent.Builder](http://developer.android.com/reference/android/support/customtabs/CustomTabsIntent.Builder.html) 类完成的；而速度的提升则是通过使用 CustomTabsClient 链接 Custom Tabs 服务，预热 Chrome 和让 Chrome 知晓将要打开的 URL 实现的。

### 打开一个Chrome Custom Tab
```java
// 使用CustomTabsIntent.Builder配置CustomTabsIntent
// 准备完成后，调用CustomTabsIntent.Builder.build()方法创建一个CustomTabsIntent
// 并通过CustomTabsIntent.launchUrl()方法加载希望加载的url
String url = ¨https://github.com/TonnyL¨;
CustomTabsIntent.Builder builder = new CustomTabsIntent.Builder();
CustomTabsIntent customTabsIntent = builder.build();
customTabsIntent.launchUrl(this, Uri.parse(url));
```

### 配置地址栏的颜色
Chrome Custom Tabs 一个很重要的功能就是我们能够改变地址栏的颜色，使之和我们应用的颜色协调。

```java
// 改变toolbar的背景色。colorInt就是想要指定的int值
builder.setToolbarColor(colorInt);
```

### 配置定制化的 action button
作为应用的开发者，我们对呈现在用户眼前的 Chrome Custom Tab 内的 Action Button 拥有完全的控制权。

在大部分的情况下，用户会执行最基础的操作像分享，或者是其他公共的 Activity。

Action Button 被表示为一个 action button 的图标和用户点击 action button 之后 Chrome 将要执行的 pendingIntent 。图标的高度一般为 24dp，宽度一般为 24-48dp。

![Action Button](https://i.loli.net/2018/03/27/5ab9e851998f8.png)

```java
// 向toolbar添加一个Action Button
// ‘icon’是一张位图(Bitmap)，作为action button的图片资源使用
// 'description'是一个字符串，作为按钮的无障碍描述所使用
// 'pendingIntent' 是一个PendingIntent，当action button或者菜单项被点击时调用。
// 在url作为data被添加之后，Chrome 会调用PendingIntent#send()方法。
// 客户端应用会通过调用Intent#getDataString()获取到URL
// 'tint'是一个布尔值，定义了Action Button是否应该被着色
builder.setActionButton(icon, description, pendingIntent, tint);
```

### 配置定制化菜单
Chrome 浏览器拥有非常全面的 action 菜单，用户在浏览器内操作非常顺畅。然而，对于我们自己的应用，可能就不适合了。

Chrome Custom Tabs 顶部有三个横向排列的图标，分别是“前进”、”页面信息”和”刷新“。在菜单的底部分别是”查找页面”和“在浏览器中打开”。

作为开发者，我们最多可以在顶部横向图标和底部菜单之间添加 **5** 个自定义菜单选项。

菜单项通过调用 [CustomTabsIntent.Builder#addMenuItem](http://developer.android.com/reference/android/support/customtabs/CustomTabsIntent.Builder.html#addMenuItem(java.lang.String,%20android.app.PendingIntent)) 添加，title 和用户点击菜单选项后 Chrome 调用的 pendingIntent 需要作为参数被传入。

![Action Menu](https://i.loli.net/2018/03/27/5ab9e8c5eb14a.png)

```java
builder.addMenuItem(menuItemTitle, menuItemPendingIntent);
```

### 配置进场和退场动画
许多的 Android 应用都会在 Activity 之间切换时使用自定义的视图进入和退出动画。Chrome Custom Tabs 也一样，我们可以改变进入和退出动画，以此保持 Chrome Custom Tabs 和应用其他内容的协调性和一致性。

```java
builder.setStartAnimations(this, R.anim.slide_in_right, R.anim.slide_out_left);
builder.setExitAnimations(this, R.anim.slide_in_left, R.anim.slide_out_right);
```

### 预热Chrome，提高页面加载速度
默认情况下，当 [CustomTabsIntent#launchUrl](http://developer.android.com/reference/android/support/customtabs/CustomTabsIntent.html#launchUrl(android.app.Activity,%20android.net.Uri)) 被调用时会激活 Chrome，加载 URL。这会花费我们宝贵的时间并且影响流畅度。

Chrome 团队了解用户对于流畅体验的渴望，所以他们在 Chrome 中提供了一个 Service 使我们的 APP 能够连接并且预热浏览器和原生组件。他们也把这种能力分享给了我们普通开发者，开发者能够告知 Chrome 用户访问页面的可能性。然后，Chrome 就能完成如下的操作：

+ 主域名的DNS预解析
+ 最有可能加载的资源的DNS预解析
+ 包括HTTPS/TLS验证在内的预连接

预热Chrome的步骤如下：

+ 使用 [CustomTabsClient#bindCustomTabsService](http://developer.android.com/reference/android/support/customtabs/CustomTabsClient.html#bindCustomTabsService(android.content.Context,%20java.lang.String,%20android.support.customtabs.CustomTabsServiceConnection)) 连接 service
+ 一旦 service 连接成功，后台调用 [CustomTabsClient#warmup](http://developer.android.com/reference/android/support/customtabs/CustomTabsClient.html#warmup(long)) 启动 Chrome
+ 调用 [CustomTabsClient#newSession](http://developer.android.com/reference/android/support/customtabs/CustomTabsClient.html#newSession(android.support.customtabs.CustomTabsCallback)) 创建一个新的 session . 这个 session 被用作所有的 API 请求
+ 我们可以在创建 session 时选择性的添加一个 [CustomTabsCallback](http://developer.android.com/reference/android/support/customtabs/CustomTabsCallback.html) 作为参数，这样我们就能知道页面是否被加载完成
+ 通过 [CustomTabsSession#mayLaunchUrl](http://developer.android.com/reference/android/support/customtabs/CustomTabsSession.html#mayLaunchUrl(android.net.Uri,%20android.os.Bundle,%20java.util.List%3Candroid.os.Bundle)) 告知 Chrome 用户最有可能加载的页面
+ 调用 [CustomTabsIntent.Builder](http://developer.android.com/reference/android/support/customtabs/CustomTabsIntent.Builder.html) 构造方法，并传入已经创建好的 CustomTabsSession 作为参数传入

### 连接Chrome服务
[CustomTabsClient#bindCustomTabsService](http://developer.android.com/reference/android/support/customtabs/CustomTabsClient.html#bindCustomTabsService) 方法简化了连接 Custom Tabs 服务的过程。

创建一个继承自 `CustomTabsServiceConnection` 的类并使用[onCustomTabsServiceConnected](http://developer.android.com/reference/android/support/customtabs/CustomTabsServiceConnection.html#onCustomTabsServiceConnected(android.content.ComponentName,%20android.support.customtabs.CustomTabsClient)) 方法获取 [CustomTabsClient](http://developer.android.com/reference/android/support/customtabs/CustomTabsClient.html) 的实例。在下一步中会用到此实例：

```java
// 官方示例
// 客户端需要连接的Chrome的包名，取决于channel的名称
// Stable(发行版) = com.android.chrome
// Beta(测试版) = com.chrome.beta
// Dev(开发版) = com.chrome.dev
public static final String CUSTOM_TAB_PACKAGE_NAME = "com.android.chrome";  // Change when in stable
CustomTabsServiceConnection connection = new CustomTabsServiceConnection() {
    
@Override
    public void onCustomTabsServiceConnected(ComponentName name, CustomTabsClient client) {
        mCustomTabsClient = client;
    }
    
@Override
    public void onServiceDisconnected(ComponentName name) {
    }
};
boolean ok = CustomTabsClient.bindCustomTabsService(this, mPackageNameToBind, connection);
```

```java
// 我的示例
package com.marktony.zhihudaily.customtabs;
import android.support.customtabs.CustomTabsServiceConnection;
import android.content.ComponentName;
import android.support.customtabs.CustomTabsClient;
import java.lang.ref.WeakReference;

/**
 * Created by Lizhaotailang on 2016/9/4.
 * Implementation for the CustomTabsServiceConnection that avoids leaking the
 * ServiceConnectionCallback
 */
public class ServiceConnection extends CustomTabsServiceConnection {
    // A weak reference to the ServiceConnectionCallback to avoid leaking it.
    private WeakReference<ServiceConnectionCallback> mConnectionCallback;
    public ServiceConnection(ServiceConnectionCallback connectionCallback) {
        mConnectionCallback = new WeakReference<>(connectionCallback);
    }
    
	@Override
    public void onCustomTabsServiceConnected(ComponentName name, CustomTabsClient client) {
        ServiceConnectionCallback connectionCallback = mConnectionCallback.get();
        if (connectionCallback != null) connectionCallback.onServiceConnected(client);
    }
    
	@Override
    public void onServiceDisconnected(ComponentName name) {
        ServiceConnectionCallback connectionCallback = mConnectionCallback.get();
        if (connectionCallback != null) connectionCallback.onServiceDisconnected();
    }
}
```

### 预热浏览器
[boolean warmup](http://developer.android.com/reference/android/support/customtabs/CustomTabsClient.html#warmup(long))

预热浏览器进程并加重原生库文件。预热是异步进行的，返回值表示请求是否被接收。多个成功的请求都会返回 true 。

true 代表着成功。

### 创建新的 tab sessioin
[boolean newSession](http://developer.android.com/reference/android/support/customtabs/CustomTabsClient.html#newSession(android.support.customtabs.CustomTabsCallback))

session 用于在连续请求中链接 mayLaunchUrl 方法。CustomTabsIntent 和 tab 互相联系。这里所提供的回调和已经创建成功的 session 相关。通过这个回调，任何关于已经成功创建的 session 的更新都会被接收到。返回 session 是否被成功创建。多个具有相同 CustomTabsCallback 或者 null 值的请求都会返回 false 。

### 告知Chrome用户可能打开的链接
[boolean mayLaunchUrl](http://developer.android.com/reference/android/support/customtabs/CustomTabsSession.html#mayLaunchUrl(android.net.Uri,%20android.os.Bundle,%20java.util.List%3Candroid.os.Bundle))

CustomTabsSession 方法告知浏览器未来可能导航到的 url 。`warmup()` 方法应该先被调用。最有可能的 url 应该最先被指出。也可以选择性的提供可能加载的 url 的列表。列表中的数据被认为被加载的可能性小于最初的那一个，而且必须按照优先级降序排列。这些额外的 url 可能被忽略掉。所有之前对于这个方法的调用都会被去优先化。返回操作是否成功完成。

### Custom Tabs连接回调
[void onNavigationEvent](http://developer.android.com/reference/android/support/customtabs/CustomTabsCallback.html#onNavigationEvent(int,%20android.os.Bundle))

在 custom tab 中，导航事件发生时被调用。`navigationEvent int` 是关于页面内的6个状态值之一。6个状态值定义如下：

```java
/**
* 页面开始加载时被发送
*/
public static final int NAVIGATION_STARTED = 1;
/**
* 页面完成加载时被发送
*/
public static final int NAVIGATION_FINISHED = 2;
/**
* 由于错误tab不能完成加载时被发送
*/
public static final int NAVIGATION_FAILED = 3;
/**
* 在加载完成之前，加载因为用户动作例如点击了另外一个链接或者刷新页面
* 加载被中止时被发送
*/
public static final int NAVIGATION_ABORTED = 4;
/**
* tab状态变为可见时发送
*/
public static final int TAB_SHOWN = 5;
/**
* tab状态变为不可见时发送
*/
public static final int TAB_HIDDEN = 6;
```

### 如果用户没有安装最新版本的Chrome，会发生什么呢？
Custom Tabs 通过带有 key Extras 的 ACTION_VIEW Intent 来定制 UI 。这就意味着将要打开的页面会通过系统浏览器或者用户默认浏览器打开。

如果用户已经安装了 Chrome 并且是默认浏览器，它会自动的获取 EXTRAS 的值并提供一个定制化的 UI 。其他的浏览器使用 Intent extras 提供相同的定制UI也是有可能的。

### 怎样检测Chrome是否支持Chrome Custom Tabs?
所有支持 Chrome Custom Tabs 的 Chrome 浏览器都暴露了一个 service 。为了检测是否支持 Chrome Custom Tabs ，可以尝试着绑定 service ，如果成功的话，那么 Customs Tabs 可以成功的使用。

## 最佳实践
启用 Chrome Custom Tabs 后,我们看到了各种不同质量界别的实现效果。这里介绍一组实现优秀集成的最佳实践。

### 连接 Custome Tabs Service 并发起预热
连接到 Custom Tabs Service 并预加载 Chrome 之后，通过 Custom Tabs 打开链接 最多可以节省 700ms 。

在我们打算启用 Custom Tabs 的 Activity 的 `onStart()` 方法中连接 Custom Tabs service 。连接成功后，调用 `warmup()` 方法。

Custom Tabs 作为一个非常低优先级的进程，这也就意味着 它不会对我们的应用不会有任何的负面的影响，但是当加载链接时，会获得非常好的启动性能。

### 内容预渲染
预渲染让内容打开非常迅速。所以，如果用户 **至少有 50% 的可能性** 打开某个链接，调用 `mayLaunchUrl()` 方法能使 Custom Tabs 预获取主页面所支持的内容并预渲染。这会最大程度的加快页面的加载速度。但是会不可避免的有 **一点流量和电量的消耗**。

### 当 Custom Tabs 没有安装时，提供备选方案
尽管 Custom Tabs 对于大多数用户都是适用的，仍然有一些场景不适用，例如设备上没有安装支持 Custom Tabs 的浏览器或者是设备上的浏览器版本不支持 Custom Tabs 。

确保提供了备选方案以提供好的应用体验。打开默认浏览器或者引入 WebView 都是不错的选择。

### 将我们的应用作为 referrer (引荐来源)
通常，对于网站而言，追用访问的来源非常地重要。当加载了 Custom Tabs 时，通过设置 referrer ，让他们知晓我们正在给他们提高访问量。

```java
intent.putExtra(Intent.EXTRA_REFERRER, 
        Uri.parse(Intent.URI_ANDROID_APP_SCHEME + "//" + context.getPackageName()));
```

### 添加定制动画
定制的动画能够使我们的应用切换到网页内容时更加地顺畅。 确保进场动画和出厂动画是反向的，这样能够帮助用户理解跳转的关系。

```java
//设置定制的进入/退出动画
CustomTabsIntent.Builder intentBuilder = new CustomTabsIntent.Builder();
intentBuilder.setStartAnimations(this, R.anim.slide_in_right, R.anim.slide_out_left);
intentBuilder.setExitAnimations(this, android.R.anim.slide_in_left,
 android.R.anim.slide_out_right);
//打开Custom Tab        
intentBuilder.build().launchUrl(context, Uri.parse("https://github.com/TonnyL"));
```

### 为 Action Button 选择一个 icon
添加一个 Action Button 能够使用户更加理解 APP 的功能。但是，如果没有好的 icon 代表 Action Button 将要执行的操作，有必要创建一个带操作文字描述的位图。

牢记位图的最大尺寸为高度 24dp，宽度 48dp。

```java
String shareLabel = getString(R.string.label_action_share);
Bitmap icon = BitmapFactory.decodeResource(getResources(),
      android.R.drawable.ic_menu_share);
// 为我们的BroadCastReceiver创建一个PendingIntent
Intent actionIntent = new Intent(
      this.getApplicationContext(), ShareBroadcastReceiver.class);
PendingIntent pendingIntent = 
      PendingIntent.getBroadcast(getApplicationContext(), 0, actionIntent, 0);	        
// 设置pendingIntent作为按钮被点击后将要执行的操作    
intentBuilder.setActionButton(icon, shareLabel, pendingIntent);
```

### 为其他浏览器做准备
牢记用户安装的浏览器中，支持 Custom Tabs 的数量可能不止一个。如果有不止一个浏览器支持 Custom Tabs ，并且没有任何一个浏览器被设置为偏好浏览器，需要询问用户如何打开链接:

```java
/**
 * 返回支持Custom Tabs的应用的包名
 */	
public static ArrayList getCustomTabsPackages(Context context) {
    PackageManager pm = context.getPackageManager();
    // Get default VIEW intent handler.
    Intent activityIntent = new Intent(Intent.ACTION_VIEW, Uri.parse("http://www.example.com"));
    // 获取所有能够处理VIEW intents的应用
    List resolvedActivityList = pm.queryIntentActivities(activityIntent, 0);
    ArrayList packagesSupportingCustomTabs = new ArrayList<>();
    for (ResolveInfo info : resolvedActivityList) {
        Intent serviceIntent = new Intent();
        serviceIntent.setAction(ACTION_CUSTOM_TABS_CONNECTION);
        serviceIntent.setPackage(info.activityInfo.packageName);
        // Check if this package also resolves the Custom Tabs service.
        if (pm.resolveService(serviceIntent, 0) != null) {
            packagesSupportingCustomTabs.add(info);
        }
    }
    return packagesSupportingCustomTabs;
}
```

### 允许用户不使用 Custom Tabs
为应用添加一个设置选项，允许用户通过默认浏览器而不是 Custom Tab 打开链接。如果我们的应用在添加 Custom Tabs 之前，都是通过默认浏览器打开链接显得尤为重要。

### 尽量让 Native 应用处理 URL
Native 应用可以处理一些 url 。如果用户安装了 Twitter APP，在点击 tweet 内的链接时，她更加希望 Twitter 应用能够处理这些链接。

在应用内打开链接之前，检查手机里有没有其他 APP 能够处理这些 url。

### 定制Toolbar的颜色
如果想要让用户感觉网页内容是我们应用的一部分，将 toolbar 的颜色设置为 primaryColor 。

如果想要让用户清楚的了解到已经离开了我们的应用，那就完全不要定义 toolbar 的颜色。

```java
// 设置自定义的toolbar的颜色
CustomTabsIntent.Builder intentBuilder = new CustomTabsIntent.Builder();
intentBuilder.setToolbarColor(Color.BLUE);
```

### 增加分享按钮
确保在 overflow 菜单中添加了一个分享的操作，在大多数的情况下，用户希望能够分享当前所见网页内容的链接，Custom Tabs 默认没有添加分享的按钮。

```java
// 在BroadCastReceiver中分享来自CustomTabs的内容
public void onReceive(Context context, Intent intent) {
    String url = intent.getDataString();
    if (url != null) {
        Intent shareIntent = new Intent(Intent.ACTION_SEND);
        shareIntent.setType("text/plain");
        shareIntent.putExtra(Intent.EXTRA_TEXT, url);
        Intent chooserIntent = Intent.createChooser(shareIntent, "Share url");
        chooserIntent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(chooserIntent);
    }
}
```

### 定制关闭按钮
自定义关闭按钮使 Custom Tabs 看起来像应用的一部分。

如果希望 CustomTabs 在用户看来像一个 Dialog, 使用 `x` (叉叉)按钮。如果希望 
 Custom Tabs 是用户的一部分，使用返回箭头。
 
 ```java
 //设置自定义的关闭按钮
CustomTabsIntent.Builder intentBuilder = new CustomTabsIntent.Builder();
intentBuilder.setCloseButtonIcon(BitmapFactory.decodeResource(
    getResources(), R.drawable.ic_arrow_back));
 ```
 
### 处理内部链接
当监听到链接是由 [android:autoLink](http://developer.android.com/reference/android/widget/TextView.html#attr_android:autoLink) 生成的或者在 WebView 中复写了 click 方法，确保我们的应用处理了这些内容的链接，让 Custom Tabs 处理外部链接。
 
```java
WebView webView = (WebView)findViewById(R.id.webview);
webView.setWebViewClient(new WebViewClient() {
    
@Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        return true;
    }
    
@Override
    public void onLoadResource(WebView view, String url) {
        if (url.startsWith("http://www.example.com")) {
            //Handle Internal Link...
        } else {
            //Open Link in a Custom Tab
            Uri uri = Uri.parse(url);
            CustomTabsIntent.Builder intentBuilder =
                    new CustomTabsIntent.Builder(mCustomTabActivityHelper.getSession());
           //Open the Custom Tab        
            intentBuilder.build().launchUrl(context, url));                            
        }
    }
});
```

### 处理连击
如果希望在用户点击链接和打开 Custom Tabs 之间做一些准备工作，确保所花费的时间不超过 100ms。否则用户会认为 APP 没有响应，可能是试着点击链接多次。

如果不能避免延迟，确保我们的应用对可能的情况做好准备，当用户点击相同的链接多次时，不要多次打开 Custom Tabs。

## 低版本 API
尽管整合 Custom Tabs 的推荐方式是使用 Custom Tabs Support Library，低 API 版本的系统也是可以使用的。

完整的 Support Library 的导入方法可以参见 [GitHub](https://github.com/GoogleChrome/custom-tabs-client/tree/master/customtabs), 并可以做为一个起点。连接 service 的 AIDL 文件也被包含在其中，Chromium 仓库中也包含了这些文件，而这些文件在 Android Studio 中是不能直接被使用的。

### 在低版本 API 中使用 Custom Tabs 的基本方法
```java
String url = ¨https://github.com/TonnyL¨;
Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url)); 
private static final String EXTRA_CUSTOM_TABS_SESSION = "android.support.customtabs.extra.SESSION";
Bundle extras = new Bundle;
extras.putBinder(EXTRA_CUSTOM_TABS_SESSION, 
   sessionICustomTabsCallback.asBinder() /* 不需要session时设置为null */);
intent.putExtras(extras);
```

### 定制 UI
UI 定制是通过向 ACTION_VIEW Intent 添加 Extras 实现的。用于定制 UI 的完整的extras keys的列表可以在 [CustomTabsIntent docs](http://developer.android.com/reference/android/support/customtabs/CustomTabsIntent.html) 找到。下面是添加自定义的 toolbar 的颜色的示例：

```java
private static final String EXTRA_CUSTOM_TABS_TOOLBAR_COLOR = "android.support.customtabs.extra.TOOLBAR_COLOR";
intent.putExtra(EXTRA_CUSTOM_TABS_TOOLBAR_COLOR, colorInt);
```

### 连接 Custom Tabs Service
Custom Tabs service 和其他 Android Service 的使用方法相同。接口通过 AIDL 创建并且代理 service 类也会自动创建。

```java
// 客户端需要连接的Chrome的包名，取决于channel的名称
// Stable(发行版) = com.android.chrome
// Beta(测试版) = com.chrome.beta
// Dev(开发版) = com.chrome.dev
public static final String CUSTOM_TAB_PACKAGE_NAME = "com.chrome.dev";  // Change when in stable
public static final String ACTION_CUSTOM_TABS_CONNECTION =
       "android.support.customtabs.action.CustomTabsService";
Intent serviceIntent = new Intent(ACTION_CUSTOM_TABS_CONNECTION);
serviceIntent.setPackage(CUSTOM_TAB_PACKAGE_NAME);
context.bindService(serviceIntent, mServiceConnection,
                    Context.BIND_AUTO_CREATE | Context.BIND_WAIVE_PRIORITY);
```

## 一些有用的网址
+ [Github Demo](https://github.com/GoogleChrome/custom-tabs-client)
+ [Chrome Custom Tabs on StackOverflow](http://stackoverflow.com/questions/tagged/chrome-custom-tabs)
+ 我的项目地址: [PaperPlane](https://github.com/TonnyL/PaperPlane)

## 总结
如果我们的应用是面向国外用户的，那理所当然的，应该加入 Chrome Custom Tabs 的支持，这在很大程度上能够提升用户的体验。如果我们的应用只是面向国内用户，我的建议还是应该加上这项功能，毕竟，还是有部分用户安装了 Chrome 浏览器，当用户浏览到 Custom Tabs 页面，应该也会像我一样，感觉到眼前一亮吧。

文章比较长，感谢阅读。