---
layout: post # needs to be post
title: DeepLinkDispatch 的用法
summary: 什么是DeepLink深度链接?
featured-img: sleek
categories: [Android]
---

### 什么是DeepLink深度链接?

先看图:

![DeepLink](https://i.loli.net/2018/03/27/5aba33969fa5e.gif)

一个在Telegram中的dribbble链接, 点击后直接跳转到我的 [`Mango`](https://github.com/TonnyL/Mango)中, 是不是很神奇?

### 为什么要使用DeepLink?
一句话总结便是提升用户体验: 原生App在功能和体验上肯定是要强于网页的.

### DeepLinkDispatch
DeepLinkDispatch提供了一种声明式的, 基于注解的API, 用于定义应用深度链接.

我们可以注册一个`Activity`, 并用`@DeepLink`和一个URI注解,然后她就可以处理特定的深度链接了.没错,就是这么简单. DeepLinkDispatch会对URI进行转换,并将深度链接和URI中特定的参数一起分发给合适的`Activity`.

#### 举个🌰
下面,我们注册一个[`ShotActivity`](https://github.com/TonnyL/Mango/blob/master/app/src/main/java/io/github/tonnyl/mango/ui/shot/ShotActivity.kt),并从像`https://dribbble.com/shots/12345`的链接中获取一个ID. 我们使用`@DeepLink`注解并且标注出将会有一个参数被标识为`id`.

```kotlin
@DeepLink("https://dribbble.com/shots/{id}")
class ShotActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
		  
		 val isDeepLink = intent.getBooleanExtra(DeepLink.IS_DEEP_LINK, false)
		 if (isDeepLink) {
            var id = intent.extras.getString("id")
            // Got the id 😉
        }
    }
}
```

#### 多个深度链接
有时候我们可能需要在一个`Activity`中处理多种链接:

```kotlin
@DeepLink("https://dribbble.com/shots/{id}, https://dribbble.com/anotherDeepLink")
class ShotActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
		  
		 val isDeepLink = intent.getBooleanExtra(DeepLink.IS_DEEP_LINK, false)
		 if (isDeepLink) {
            var id = intent.extras.getString("id")
            // Got the id 😉
        }
    }
}
```

#### 方法注解
我们还可以将`@DeepLink`注解用于任何`public static`方法(在Kotlin中即`companion object`中的方法). DeepLinkDispatch会调用这个方法来创建`Intent`

```kotlin
companion object {
    @DeepLink("foo://example.com/methodDeepLink/{param1}")
    fun intentForDeepLinkMethod(context: Context): Intent {
        return Intent(context, MainActivity::class.java)
                .setAction(ACTION_DEEP_LINK_METHOD)
    }
}
```

如果我们需要`Intent`的extras, 可以直接在方法中添加一个`Bundle`类型的参数,例如:

```kotlin
@DeepLink("foo://example.com/methodDeepLink/{param1}")
fun intentForDeepLinkMethod(context: Context, extras: Bundle): Intent {
    val uri = Uri.parse(extras.getString(DeepLink.URI)).buildUpon()
    return Intent(context, MainActivity::class.java)
            .setData(uri.appendQueryParameter("bar", "baz").build())
            .setAction(ACTION_DEEP_LINK_METHOD)
}
```

如果我们需要定制`Activity`的返回栈, 可以返回一个`TaskStackBuilder`而不是一个`Intent`. DeepLinkDispatch会调用被注解的方法,从`TaskStackBuilder`的最后一个`Intent`创建`Intent`, 当从已经注册的deep link启动`Activity`时使用.

```kotlin
@DeepLink("http://example.com/deepLink/{id}/{name}")
fun intentForTaskStackBuilderMethods(context: Context): TaskStackBuilder {
    val detailsIntent = Intent(context, SecondActivity::class.java).setAction(ACTION_DEEP_LINK_COMPLEX)
    val parentIntent = Intent(context, MainActivity::class.java).setAction(ACTION_DEEP_LINK_COMPLEX)
    val taskStackBuilder = TaskStackBuilder.create(context)
    taskStackBuilder.addNextIntent(parentIntent)
    taskStackBuilder.addNextIntent(detailsIntent)
    return taskStackBuilder
}
```

#### 查询参数
查询参数被自动的转换和传递,并且像其他参数一样是可获取的. 例如, 我们可以获取URI `foo://example.com/deepLink?qp=123` 传递过来的查询参数:

```kotlin
@DeepLink("foo://example.com/deepLink")
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val intent = intent
        if (intent.getBooleanExtra(DeepLink.IS_DEEP_LINK, false)) {
            val parameters = intent.extras
            parameters?.getString("qp")?.let {
                val queryParameter = parameters.getString("qp")
                // Do something with the query parameter...
            }
        }
    }

}
```

#### 回调
我们可以注册`BroadcastReceiver`,当通过深度链接进入我们的应用时, 她将被调用, 当然, 这是可选的. 当处理深度链接时,DeepLinkDispatch通过`LocalBroadcastManager`发送广播并传递成功或者失败的`Intent`. Intent会携带下面的extras被传播.

+ `DeepLinkHandler.EXTRA_URI`: 深度链接的URI.
+ `DeepLinkHandler.EXTRA_SUCCESSFUL`: 深度链接是否成功.
+ `DeepLinkHandler.EXTRA_ERROR_MESSAGE`: 如果出错, 错误信息.

我们可以注册一个接收器,用于接收intent. 下面是一个使用的例子:

```kotlin
class DeepLinkReceiver : BroadcastReceiver() {

    companion object {
        private val TAG = "DeepLinkReceiver"
    }

    override fun onReceive(context: Context, intent: Intent) {
        val deepLinkUri = intent.getStringExtra(DeepLinkHandler.EXTRA_URI)
        if (intent.getBooleanExtra(DeepLinkHandler.EXTRA_SUCCESSFUL, false)) {
            Log.i(TAG, "Success deep linking: " + deepLinkUri)
        } else {
            val errorMessage = intent.getStringExtra(DeepLinkHandler.EXTRA_ERROR_MESSAGE)
            Log.e(TAG, "Error deep linking: $deepLinkUri with error message +$errorMessage")
        }
    }
    
}


class App : Application() {
    override fun onCreate() {
        super.onCreate()
        val intentFilter = IntentFilter(DeepLinkHandler.ACTION)
        LocalBroadcastManager.getInstance(this).registerReceiver(DeepLinkReceiver(), intentFilter)
    }
}
```

#### 自定义注解
我们可以创建自定义注解来减少深度链接的重复. 自定义注解提供公共的前缀, 这些公共前缀会被自动应用到每一个被自定义注解注解的类和方法. 自定义注解一个比较流行的用法便是在web App的深度链接中:

```kotlin
// Prefix all app deep link URIs with "app://airbnb"
@DeepLinkSpec(prefix = arrayOf("app://airbnb"))
annotation class AppDeepLink(vararg val value: String)
```

```kotlin
// Prefix all web deep links with "http://airbnb.com" and "https://airbnb.com"
@DeepLinkSpec(prefix = arrayOf("http://airbnb.com", "https://airbnb.com"))
annotation class WebDeepLink(vararg val value: String)
```

```kotlin
// This activity is gonna hanndle the following deep links:
// "app://airbnb/view_users"
// "http://airbnb.com/users"
// "http://airbnb.com/user/{id}"
// "https://airbnb.com/users"
// "https://airbnb.com/user/{id}"
@AppDeepLink("/view_users")
@WebDeepLink("/users", "/user/{id}")
class CustomPrefixesActivity : AppCompatActivity()//...
```

### 使用方法
将下面的代码添加至`build.gradle`文件中:

```
dependencies {
  compile 'com.airbnb:deeplinkdispatch:3.1.0'
  annotationProcessor 'com.airbnb:deeplinkdispatch-processor:3.1.0'
}
```

创建我们的深度链接module(**DeepLinkDispatch v3**新推出).每一个被`@DeepLinkModule`注解的类, DeepLinkDispatch都会生成一个`Loader`类, 其中包含了所有`@DeepLink`注解的注册信息.

```kotlin
/** This will generate a AppDeepLinkModuleLoader class */
@DeepLinkModule
class AppDeepLinkModule
```

**可选部分**: 如果我们的Android应用包含了多个module(例如独立的Android library工程), 我们需要为应用中的每一个Module都添加一个`@DeepLinkModule`注解类, 只有那样DeepLinkDispatch才能在每一个module中的一个`loader`类收集所有注解.

```kotlin
/** This will generate a LibraryDeepLinkModuleLoader class */
@DeepLinkModule
class LibraryDeepLinkModule
```

创建一个`Activity`(例如`DeepLinkActivity`), 需要带有要处理schema(在`AndroidManifest.xml`文件中注册, 下面的例子中使用`foo`示例):

```xml
<activity
    android:name="com.example.DeepLinkActivity"
    android:theme="@android:style/Theme.NoDisplay">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="foo" />
    </intent-filter>
</activity>
```

用`@DeepLinkHandler`注解我们的`DeepLinkActivity`, 并提供所有被`@DeepLinkModule`注解的类的列表:

```kotlin
@DeepLinkHandler(AppDeepLinkModule::class, LibraryDeepLinkModule::class)
class DeepLinkActivity : Activity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // DeepLinkDelegate, LibraryDeepLinkModuleLoader and AppDeepLinkModuleLoader
        // are generated at compile-time.
        val deepLinkDelegate = DeepLinkDelegate(AppDeepLinkModuleLoader(), LibraryDeepLinkModuleLoader())
        // Delegate the deep link handling to DeepLinkDispatch. 
        // It will start the correct Activity based on the incoming Intent URI
        deepLinkDelegate.dispatchFrom(this)
        // Finish this Activity since the correct one has been just started
        finish()
    }
}
```

#### 小贴士
+ 从 DeepLinkDispatch v3开始, 我们必须 **一直** 提供我们自己的`Activity`类并用`@DeepLinkHandler`注解. 她将不在被注解处理器(Annotation processor or kapt)自动生成.
+ Intent filters只能包含一个URI pattern的一个数据元素(data element). 过滤额外的URI pattern需要创建独立的intent filters.
+ 参照[sample app](https://github.com/airbnb/DeepLinkDispatch/blob/master/sample/src/main/java/com/airbnb/deeplinkdispatch/sample/DeepLinkActivity.java), 示例讲解了DeepLinkDispatch库的用法.

开发版本的Snapshots可以在[Sonatype's `snapshots` repository](https://oss.sonatype.org/content/repositories/snapshots/)获取.

#### 生成深度链接的文档
我们可以告知DeepLinkDispatch生成带有所有深度链接注解的txt文本文档, 我们可以使用文档进行进一步的开发或者作为参考. 为了生成文档, 我们需要在`build.gradle`文件中添加如下代码: 

```
tasks.withType(JavaCompile) {
  options.compilerArgs << "-AdeepLinkDoc.output=${buildDir}/doc/deeplinks.txt"
}
```

文档会以下面的格式生成:

```
* {DeepLink1}\n|#|\n[Description part of javadoc]\n|#|\n{ClassName}#[MethodName]\n|##|\n
* {DeepLink2}\n|#|\n[Description part of javadoc]\n|#|\n{ClassName}#[MethodName]\n|##|\n
```

### 混淆规则
```
-keep class com.airbnb.deeplinkdispatch.** { *; }
-keepclasseswithmembers class * {
     @com.airbnb.deeplinkdispatch.DeepLink <methods>;
}
```

**小贴士**: 不要忘记在混淆规则中包含我们使用过的自定义注解, 例如:

```
-keep @interface your.package.path.deeplink.** { *; }
-keepclasseswithmembers class * {
    @your.package.path.deeplink.* <methods>;
}
```

### 测试示例应用
使用adb加载深度链接(在terminal中输入: `adb shell`).

这将触发一个标准的深度链接. 源注解: `@DeepLink("dld://example.com/deepLink")`

`am start -W -a android.intent.action.VIEW -d "dld://example.com/deepLink" `

`com.airbnb.deeplinkdispatch.sample`

我们可以包含多个路径参数(不需要包含示例应用的包名). 源注解: `@DeepLink("http://example.com/deepLink/{id}/{name}")`

`am start -W -a android.intent.action.VIEW -d "http://example.com/deepLink/123abc/myname" `

需要注意的是有可能直接调用`adb shell am ...` 不过这种方式有时可能会丢失URI, 所以最好是从shell中调用.

### 参考
+ https://github.com/airbnb/DeepLinkDispatch
+ https://github.com/TonnyL/Mango