---
layout: post # needs to be post
title: 老司机带你吃牛轧糖 - 探索 Android 7.1 Nougat 新特性
summary: What’s new in Android 7.1 Nougat?
featured-img: sleek #optional - if you want you can include hero image
categories: [Android]
---

## What’s new in Android 7.1 Nougat?
Android 7.1 Nougat 已经推出有一段时间，相信大多数人和我一样，并没有用上最新的系统，但是，总有一群走在时代的前列线上的Geek们，勇于尝鲜，艰苦奋斗，为刷新版本号贡献自己的力量。好吧，实际上就是我还没有用上7.1，有些眼馋了。那么，和开发者息息相关的有哪些新特性呢？

本次主要介绍3个新特性: `App Shortcuts`, `Round Icon Resource` 和 `Image Keyboard Support` 。所有的新特性可以访问谷歌开发者中文博客的文章 [欢迎使用Android 7.1.1 Nougat](http://developers.googleblog.cn/2016/12/android-711-nougat_20.html)。

## App Shortcuts
作为一个密切关注 Android 发展的伪 Geek ，在7.1正式版未发布之前，通过网上的一些爆料文章，我就了解到了这一新功能。实际上，这个功能刚开始出现时，我还以为 Google Pixel 要上压感屏了呢，事实证明，的确是我想多了。

App Shortcuts 允许用户直接在启动器中显示一些操作，让用户立即执行应用的深层次的功能。触发这一功能的操作就是「长按」。这一功能类似于iOS中的「3D Touch」。

下面通过一张GIF，直观的感受一下 App Shortcuts 是怎样的。（由于我的一加3并没有升级到最新的7.1，还只是7.0，所以我安装了 Nova Launcher 来体验。）

![App shortcuts.gif](https://i.loli.net/2018/03/27/5ab9d9068bb72.gif)

长按图标，收到震动后松手，如果能够看到图标上弹出了支持的跳转操作，说明成功的呼出了 Shortcuts 功能，如果不支持这一功能，在 Nova Launcher 上弹出的就是卸载或者移除操作，在 Pixel Launcher 上不会出现弹出菜单，显示的是常见的长按操作。长按弹出的操作，可以将这个操作已快捷方式图标的形式直接放置在主屏上。如果长按主图标不松手，就可以调整位置了。

目前，一个应用最多可以支持 **5** 个 Shortcut,可以通过 [getMaxShortcutCountPerActivity](https://developer.android.com/reference/android/content/pm/ShortcutManager.html#getMaxShortcutCountPerActivity()) 查看Launcher最多支持 Shortcut 的数量。每一个 Shortcut 都对应着一个或者多个intent，当用户选择某一个 Shortcut 时，应该做出特定的动作。下面是一些将一些特定的动作作为 Shortcuts 的例子：

+ 在地图APP中，指引用户至最常用的位置

+ 在聊天APP中，发送信息至某个好友

+ 在多媒体APP中，播放下一个电视节目

+ 在游戏APP中，加载至上次保存的地方

App Shortcut 可以分为两种不同的类型: Static Shortcuts（静态快捷方式） 和 Dynamic Shortcuts（动态快捷方式）。

+ Static Shortcuts：在打包到apk的资源文件中定义，所以，直到下一次更新版本时才能改变静态快捷方式的详细说明。
+ Dynamic Shortcuts：通过ShortcutManager API在运行时发布，在运行时，应用可以发布，升级和移除快捷方式。

### Using Static Shortcuts
创建 Static Shortcuts 分为以下几步：

1. 在工程的 manifest 文件 (AndroidManifest.xml)下，找到 intent filter 设置为 **android.intent.action.MAIN** 和 **android.intent.category.LAUNCHER** 的Activity。
	
2. 在此 Activity 下添加标签，引用定义 shortcuts 的资源文件。
	
	```xml
	<activity
        android:name=".homepage.MainActivity"
        android:configChanges="orientation|keyboardHidden|screenSize|screenLayout"
        android:label="@string/app_name"
        android:theme="@style/AppTheme.NoActionBar">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
        <meta-data
            android:name="android.app.shortcuts"
            android:resource="@xml/shortcuts" />
    </activity>
	```

3. 创建新的资源文件

	```xml
	<?xml version="1.0" encoding="utf-8"?>
<shortcuts xmlns:android="http://schemas.android.com/apk/res/android">

	    <shortcut
	        android:enabled="true"
	        android:icon="@drawable/ic_search_circle"
	        android:shortcutId="search_bookmarks"
	        android:shortcutShortLabel="@string/search_bookmarks"
	        android:shortcutLongLabel="@string/search_bookmarks">
	        <intent
	            android:action="android.intent.action.VIEW"
	            android:targetPackage="com.marktony.zhihudaily"
	            android:targetClass="com.marktony.zhihudaily.search.SearchActivity" />
			<!--如果你的一个shortcut关联着多个intent，你可以在这里继续添
				加。最后一个intent决定着用户在加载这个shortcut时会看到什么-->
	        <categories android:name="android.shortcut.conversation" />
	    </shortcut>
	    
		<!--在这里添加更多的shortcut-->
		
	</shortcuts>
	```
	
	`shortcut` 下标签的含义：
	
	+ enabled:见名知意，shortcut是否可用。如果你决定让这个static shortcut不在可用的话，可直接将其设置为 false ，或者直接从 shortcuts 标签中移除。

	+ icon:显示在左边的图标，可用使用Vector drawable。

	+ shortcutDisabledMessage:当禁用此shortcut后，它仍然会出现在用户长按应用图标后的快捷方式列表里，也可以被拖动并固定到桌面上，但是它会呈现灰色并且用户点击时会弹出Toast这个标签所定义的内容。

	+ shortcutLongLabel:当启动器有足够多的空间时，会显示这个标签所定义的内容。

	+ shortcutShortLabel:shortcut的简要说明，是必需字段。当shortcut被添加到桌面上时，显示的也是这个字段。

	+ intent:shortcut关联的一个或者多个intent，当用户点击shortcut时被打开。

	+ shortcutId:shortcut的唯一标示id，若存在具有相同shortcutId的shortcut，则只显示一个。

	到这里，最简单的shortcut就添加成功了。运行包含上面的文件的项目，点击shortcut就可以直接进入 SearchActivity，当按下back键时，直接就退出了应用。如果希望不退出应用，而是进入 **MainActivity** 时，应该怎么办呢？不用着急，在shortcut继续添加intent就可以了。
	
	```xml
	<?xml version="1.0" encoding="utf-8"?>
<shortcuts xmlns:android="http://schemas.android.com/apk/res/android">
	
	    <shortcut
	        android:enabled="true"
	        android:icon="@drawable/ic_search_circle"
	        android:shortcutId="search_bookmarks"
	        android:shortcutShortLabel="@string/search_bookmarks"
	        android:shortcutLongLabel="@string/search_bookmarks">
		
			<intent
	    			android:action="android.intent.action.MAIN"
	    			android:targetClass="com.marktony.zhihudaily.homepage.MainActivity"
			    android:targetPackage="com.marktony.zhihudaily" />
		
	        <intent
	            android:action="android.intent.action.VIEW"
	            android:targetPackage="com.marktony.zhihudaily"
	            android:targetClass="com.marktony.zhihudaily.search.SearchActivity" />
		
	        <categories android:name="android.shortcut.conversation" />
		
	    </shortcut>
		
	  	<!--在这里添加更多的shortcut-->
		
	 </shortcuts>
	```

### Using Dynamic Shortcuts
动态快捷方式应该和应用内的特定的、上下文敏感的 action 链接。这些 action 应该可以在用户的几次使用之间、甚至是在应用运行过程中被改变。好的候选 action 包括打电话给特定的人、导航至特定的地方、或者展示当前游戏的分数。

[ShortcutManager API](https://developer.android.com/reference/android/content/pm/ShortcutManager.html) 允许我们在动态快捷方式上完成下面的操作：

+ 发布: 使用 `setDynamicShortcuts()` 重新定义整个动态快捷方式列表，或者是使用 `addDynamicShortcuts()` 向已存在的动态快捷方式列表中添加快捷方式。

+ 更新: 使用 `updateShortcuts()` 方法。

+ 移除: 使用 `removeDynamicShortcuts()` 方法移除特定动态快捷方式或者使用 `removeAllDynamicShortcuts()` 移除所有动态快捷方式。

下面是在 MainActivity 的 `onCreate()` 中创建动态快捷方式的例子：

```java
protected void onCreate(Bundle savedInstanceState) {
    // ...
    ShortcutManager shortcutManager = getSystemService(ShortcutManager.class);
    ShortcutInfo webShortcut = new ShortcutInfo.Builder(this, "shortcut_web")
            .setShortLabel("github")
            .setLongLabel("Open Tonny's github web site")
            .setIcon(Icon.createWithResource(this, R.drawable.ic_dynamic_shortcut))
            .setIntent(new Intent(Intent.ACTION_VIEW, Uri.parse("https://TonnyL.github.io")))
            .build();
    shortcutManager.setDynamicShortcuts(Collections.singletonList(webShortcut));
}
```

也可以为动态快捷方式创建返回栈:

```java
protected void onCreate(Bundle savedInstanceState) {
    // ...
    ShortcutInfo dynamicShortcut = new ShortcutInfo.Builder(this, "shortcut_dynamic")
            .setShortLabel("Dynamic")
            .setLongLabel("Open dynamic shortcut")
            .setIcon(Icon.createWithResource(this, R.drawable.ic_dynamic_shortcut_2))
            .setIntents(
                    new Intent[]{
                            new Intent(Intent.ACTION_MAIN, Uri.EMPTY, this, MainActivity.class).setFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK),
                            new Intent(DynamicShortcutActivity.ACTION)
                    })
            .build();
    shortcutManager.setDynamicShortcuts(Arrays.asList(webShortcut, dynamicShortcut));
}
```

创建一个新的空的 Activity ，名字叫做 DynamicShortcutActivity ，在 manifest 文件中注册。

```xml
<activity  
      android:name=".DynamicShortcutActivity"
      android:label="Dynamic shortcut activity">
      <intent-filter>
        <action android:name="com.marktony.zhihudaily.OPEN_DYNAMIC_SHORTCUT" />
        <category android:name="android.intent.category.DEFAULT" />
      </intent-filter>
</activity>
```

通过清除 array 中的排序过的 intents ，当我们通过创建好的 shortcut 进入 DynamicShortcutActivity 之后，按下 back 键，MainActivity 就会被加载。

需要注意的是，在动态创建快捷方式之前，最好是检查一下是否超过了所允许的最大值。否则会抛出相应的异常。

### Extra Bits
当 static shortcut 和 dynamic shortcut 一起展示时，其出现的顺序是怎样定制呢？

在 **ShortcutInfo.Builder** 中有一个专门的方法 `setRank(int)` ,通过设置不同的等级，我们就可以控制动态快捷方式的出现顺序，等级越高，出现在快捷方式列表中的位置就越高。

我们还可以设置动态快捷方式的 shortLabel 的字体颜色。

```java
ForegroundColorSpan colorSpan = new ForegroundColorSpan(getResources().getColor(android.R.color.holo_red_dark, getTheme()));

String label = "github";
SpannableStringBuilder colouredLabel = new SpannableStringBuilder(label);
colouredLabel.setSpan(colorSpan, 0, label.length(), Spanned.SPAN_INCLUSIVE_INCLUSIVE);
ShortcutInfo webShortcut = new ShortcutInfo.Builder(MainActivity.this, "shortcut_web")
       .setShortLabel(colouredLabel)
       .setRank(1)
       .build();
```

### App Shortcuts Best Practices
当设计和创建应用的 shortcuts 时，应该遵守下面的指导建议：

+ 遵循设计规范: 为了保持我们的应用和系统应用的快捷方式在视觉上一致性，应该遵守 [App Shortcuts Design Guidelines](https://developer.android.com/shareables/design/app-shortcuts-design-guidelines.pdf)。

+ 发布4个不同的快捷方式:尽管现在的 API 支持静态和动态总共5个快捷方式，但是为了提高 shortcut 的视觉效果，建议只添加4个不同的快捷方式。

+ 限制快捷方式描述的文本长度:在 Launcher 中，显示快捷方式时，空间长度受到了限制。如果可能的话，应该将「short description」的文字长度控制在10个字母以内，将「long discription」的长度限制在25个字母以内。

+ 保存 shortcut 和 action 的历史记录: 创建的每一个 shortcut，应该考虑到用户能够通过不同的方式完成相同的任务。在这种情况下，记得调用 `reportShortcutUsed()` 方法，这样，launcher 就可以提高 shortcut 对应的 actions 的反应速度。

+ 只有在 shortcuts 的意义存在时更新：当改变动态快捷方式时，只有在 shortcut 仍然保持着它的含义时，调用 `updateShortcuts()` 方法改变它的信息。否则，应该使用 `addDynamicShortcuts()` 或者 `setDynamicShortcuts()` 创建一个具有新含义的ID的快捷方式。

举个例子，如果我们已经创建了导航到一个超市的快捷方式，如果超市的名称改变了但是位置并没有变化时，只更新信息是合适的。但是如果用户开始在一个不同位置的超市购物时，最好是创建一个全新的快捷方式(而不仅仅是更新信息了)。

+ 在备份和恢复时，动态 shortcuts 不应该被保存: 正是因为这个原因，推荐我们在需要 APP 启动和重新发布动态快捷方式时，检查 `getDynamicShortcuts()` 的对象的数量。可以参考 [Backup and Restore](https://developer.android.com/guide/topics/ui/shortcuts.html#backup-and-restore) 部分的代码片段。

## Round Icon Resources
在Android 7.1上，Google推出了一个部分用户可能不太喜欢的特性–圆形图标。圆形图标长什么样，可以看看下面的图。

![Round Icon Resources.png](https://i.loli.net/2018/03/27/5ab9dc9d03c5e.png)

> 应用程序现在可以定义圆形启动器图标以用于特定的移动设备之上。当启动器请求应用程序图标时，程序框架应返回 android：icon 或 android：roundIcon，视设备具体要求而定。因此，应用程序在开发时应该确保同时定义 android：icon和 android：roundIcon 两个变量。您可以使用 Image Asset Studio 来设计圆形图标。
> 
> 您应该确保在支持新的圆形图标的设备上测试您的应用程序，以确保应用程序图标的外观无虞和实际效果。测试您的资源的一种方法是在 Google Pixel 设备上安装您的应用。您还可以通过运行 Android 模拟器并使用 Google API 模拟器系统（目标 API 等级为 25）测试您的图标。

我们可以通过 **Android Studio** 自带的 **Image Asset Studio** 设计图标。在项目的 `res` 目录下点击鼠标右键，选择 `new –> Image Asset` 即可设计图标。

![Image Asset Studio.png](https://i.loli.net/2018/03/27/5ab9dd227d4bc.png)

更多关于设计应用图标的信息，可以参考 [Material Design guidelines](https://material.google.com/style/icons.html#icons-product-icons) 。

## Image Keyboard Support
在较早版本的 Android 系统中，软键盘(例如我们所熟知的 Input Method Editors ，或者说 IME )，只能够给应用发送 unicode 编码的 emoji ，对于 rich content ，应用只能通过使用自建的私有的 API 实现发送图片的功能。而在 Android 7.1中，SDK 包含了一个全新的 Commit Content API ，输入法应用不仅可以调用此 API 实现发送图片和其他rich content，一些通讯应用（比如 Google Messenger）也可以通过此 API 来更好地处理这些来自输入法的图片、网页信息和 GIF 内容。

![Image Keyboard Support.png](https://i.loli.net/2018/03/27/5ab9dd8f30f30.png)


### How it works
1. 当用户点击 EditText 时，editor 会发送一个它所能接受的  `EditorInfo.contentMimeTypes` MIME 内容类型的列表。

2. IME 读取这个在软键盘中支持类型和展示内容的列表。

3. 当用户选择一张图片后，IME调用 `commitContent()` 并向 editor 发送一个 `InputContentInfo` 。 `commitContent()` 方法是一个类似于 `commitText()` 的方法，但是是 rich content 的。 `InputContentInfo` 包含着一个表示 content provider 中内容的 URI 。然后我们的应用就可以请求相应的权限并读取 URI 中的内容。

![How Image Keyboard Support Works.png](https://i.loli.net/2018/03/27/5ab9de2750fca.png)

### Adding Image Support to Apps
为了接收来自 IME 的 rich content ，应用必须告诉 IME 它所能接收的内容类型并之指定当接收到内容后的回调方法。下面是一个怎样创建一个能够接收 PNG 图片的 EditText 的演示代码。

```java
EditText editText = new EditText(this) {
    
	@Override
    public InputConnection onCreateInputConnection(EditorInfo editorInfo) {
        final InputConnection ic = super.onCreateInputConnection(editorInfo);
        EditorInfoCompat.setContentMimeTypes(editorInfo,
                new String [] {"image/png"});
        final InputConnectionCompat.OnCommitContentListener callback =
            new InputConnectionCompat.OnCommitContentListener() {
                
				@Override
                public boolean onCommitContent(InputContentInfoCompat inputContentInfo,
                        int flags, Bundle opts) {
                    // read and display inputContentInfo asynchronously
                    if (BuildCompat.isAtLeastNMR1() && (flags &
                        InputConnectionCompat.INPUT_CONTENT_GRANT_READ_URI_PERMISSION) != 0) {
                        try {
                            inputContentInfo.requestPermission();
                        }
                        catch (Exception e) {
                            return false; // return false if failed
                        }
                    }
                    // read and display inputContentInfo asynchronously.
                    // call inputContentInfo.releasePermission() as needed.
                    return true;  // return true if succeeded
                }
            };
        return InputConnectionCompat.createWrapper(ic, editorInfo, callback);
    }
};
```

代码还是蛮多的，解释一下。

+ 例子使用了 support library，并且引用的是 `android.support.v13.view.inputmethod` 而不是 `android.view.inputmethod` 。

+ 例子创建了一个 EditText 并复写了它改变 `InputConnection` 的 `onCreateInputConnection(EditorInfo)` 方法. InputConnection 是 IME 和正在接收输入的沟通管道。

+ 调用 `super.onCreateInputConnection()` 保留了内建的行为(包括发送和接收文本)，并提供给我们一个 InputConnection 的引用。

+ `setContentMimeTypes()` 向 EditorInfo 添加了一个所支持的MIME类型的列表。 需要保证在 `setContentMimeTypes()` 之前调用 `super.onCreateInputConnection()` 。

+ 回调在 IME 提交内容是被执行。 `onCommitContent()` 方法有一个对包含了内容URI的 InputContentInfoCompat 的引用。

	- 当我们的应用运行在 API Level 25或者更高并且 IME 设置了 `INPUT_CONTENT_GRANT_READ_URI_PERMISSION` flag 时，我们应该请求并且释放权限。否则，我们应该在此之前就拥有 content URI 的访问权限，一是因为权限是由 IME 授权的，二是 content provider 不对访问进行约束。更多的信息可以访问 [Adding Image Support to IMEs](https://developer.android.com/guide/topics/text/image-keyboard.html#adding_image_support_to_imes).

下面是一些实践小技巧。

+ 不支持 rich content的Editor不应该调用 `setContentTypes()` 并把 `EditorInfo.contentMimeTypes` 设置为 null。

+ Editor 应该忽略掉在 `InputConnectionInfo` 中指定的 MIME 类型和所接收类型不通的内容。

+ rich content不影响也不被文本指针的位置所影响。editor 在进行内容处理是可以直接忽略掉光标的位置。

+ 在 editor 的 `OnCommitContentListener.onCommitContent()` 方法中，我们可以异步的返回 true，甚至是在加载内容之前。

+ 不同于文本内容在被提交之前可以在 IME 中被编辑，rich content 会被立即提交。需要注意特性，如果想要提供编辑或者删除内容的能力，我们需要自己提供处理逻辑。

为了测试 APP，需要确保你的设备或者虚拟机的键盘能够发送 rich content 。你可以在Android 7.1 或者更高的系统中使用 Google Keyboard ，或者是安装 [CommitContent IME sample](https://developer.android.com/samples/CommitContentSampleIME/index.html).

你可以在 [CommitContent App sample](https://developer.android.com/samples/CommitContentSampleApp/index.html) 获取到完整的示例代码。

### Adding Image Support to IMEs
想要 IME 支持发送 rich content ，需要引入下面所展示的 Commit Content API 。

+ 复写 `onStartInput()` 或者 `onStartInputView()` ，并读取来自目标 editor 的支持内容类型列表。

	```java
	public void onStartInputView(EditorInfo info, boolean restarting) {
	    String[] mimeTypes = EditorInfoCompat.getContentMimeTypes(editorInfo);
	    boolean gifSupported = false;
	    for (String mimeType : mimeTypes) {
	        if (ClipDescription.compareMimeTypes(mimeType, "image/gif")) {
	            gifSupported = true;
	        }
	    }
	    if (gifSupported) {
	        // the target editor supports GIFs. enable corresponding content
	    } else {
	        // the target editor does not support GIFs. disable corresponding content
	    }
	}
	```

+ 当用户选择了一张图片时，将内容提交给 APP 。当 IME 有正在编辑的文本时，应该避免调用 `commitContent()` ，因为这样可能导致 editor 失去焦点。下面的代码片段展示了怎样提交一张 GIF 图片。

	```java
	/**
	 * Commits a GIF image
	 *
	 * @param contentUri Content URI of the GIF image to be sent
	 * @param imageDescription Description of the GIF image to be sent
	 */
	public static void commitGifImage(Uri contentUri, String imageDescription) {
	    InputContentInfoCompat inputContentInfo = new InputContentInfoCompat(
	            contentUri,
	            new ClipDescription(imageDescription, new String[]{"image/gif"}));
	    InputConnection inputConnection = getCurrentInputConnection();
	    EditorInfo editorInfo = getCurrentInputEditorInfo();
	    Int flags = 0;
	    If (android.os.Build.VERSION.SDK_INT >= 25) {
	        flags |= InputConnectionCompat.INPUT_CONTENT_GRANT_READ_URI_PERMISSION;
	    }
	    InputConnectionCompat.commitContent(
	            inputConnection, editorInfo, inputContentInfo, flags, opts);
	}
	```
	
+ 作为一个 IME 开发者，有很大可能你需要引入你自己的 content provider 来响应 content URI 请求。如果你的 IME 支持来自像 `MediaStore` 这样已经存在的 content provider 倒是可以例外。关于创建 content provider 的更多信息，可以参见 [CommitContent IME sample](https://developer.android.com/samples/CommitContentSampleIME/index.html), [Content Provider](https://developer.android.com/guide%20/topics/providers/content-providers.html) 文档, [File Provider](https://developer.android.com/training/secure-file-sharing/setup-sharing.html) 文档。

+ 如果正在创建自己的 content provider ，建议不要 export (将 `android:export` 设置为 false )。通过设置 `android:grandUriPermission` 为true 允许在 provider 内部进行权限授予替代。然后，你的 IME 在内容提交时可以授予访问 content URI 的权限。有两种实现的方法：

	- 在 Android 7.1(API Level 25) 或更高的系统中，当调用 `commitContent` 方法时，将 flag 参数设置为 `INPUT_CONTENT_GRANT_READ_URI_PERMISSION` 。然后，APP 收到的 `InputContentInfo` 对象可以通过调用 `requestPermission()` 方法和 `releasePermission()` 请求和释放临时访问权限。

	- 在 Android 7.0(API Level 24)或者更低的系统中， `INPUT_CONTENT_GRANT_READ_URI_PERMISSION` 直接被忽略，所以我们需要手动的授予内容访问权限。方法就是 `grantUriPermission()` ,但是我们也可以引入满足自己要求的机制。

权限授予的例子，我们可以在 [CommitContent IME sample](https://developer.android.com/samples/CommitContentSampleIME/index.html) 中的 `doCommitContent()` 方法。

为了测试 IME，确保我们的设备或者模拟器拥有接收 rich content 的应用。我们可以在 Android 7.1 或者更高的系统中使用 Google Messenger 应用或者安装 [CommitContent App Sample](https://developer.android.com/samples/CommitContentSampleApp/index.html) 。

获取完整的示例代码，可以访问 [CommitContent IME Sample](https://developer.android.com/samples/CommitContentSampleIME/index.html)。

## Summary
Google 在刷新版本号的路上简直是在策马奔腾了，嘚儿驾。我们也能够看到 Google 的努力， Android 也在变的越来越好，加油吧，小机器人。

本次 Shortcuts 部分的代码可以在我的 GitHub 仓库 [PaperPlane](https://github.com/TonnyL/PaperPlane)中看到。欢迎 star 哟。