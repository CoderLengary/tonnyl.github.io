---
layout: post # needs to be post
title: APK 瘦身实践
summary: APK 瘦身实践
featured-img: work
categories: [Android]
---

在开始对APK进行正式的减肥之前，我们先来了解一下APK包的构成，这样才好对症下药嘛。知己知彼方能百战不殆。

### APK文件内容速揽
我们可以通过 `unzip <apkname>.apk` 命令实现对apk的解压(你也可以直接将`<apkname>.apk`更名为`<apkname>.zip`后直接通过解压软件解压)。解压后，我们就会得到一个文件夹了，里面包含了:

```
AndroidManifest.xml
/assets
classes.dex
classes2.dex
...
/lib
    /arm64-v8a
    /armeabi-v7a
    /mips
    /x86
    /x86_64
/META-INF
    CERT.RSA
    CERT.SF
    MANIFEST.MF
/res
    /anim
    ...
resources.arsc
```

各个文件和目录的作用是什么呢？

文件/目录 | 作用
--- | ---
AndroidManifest.xml | 清单文件，列出了应用的名称，版本，权限等信息。
assets | 配置文件，放置一些本地资源，例如本地html，可以通过使用`AssetManager`检索。
classes.dex, classes2.dex | 编译后的字节码文件，能够被Dalvik/ART理解。
lib | 存放特定平台使用的编译后的.so文件，每一个子文件夹对应一个特定的平台，例如`armeabi`, `armeabi-v7a`, `arm64-v8a`, `x86`, `x86_64`, and `mips`.
META-NIF | 包含 `CERT.RSA`, `CERT.SF`, `MANIFEST.MF`， 存放的签名信息，用于保证系统的安全性和APK文件的完整性。
res | 存放没有被编译到 `resources.arsc`文件中的资源文件。
resouces.arsc | 编译后的二进制资源文件。存放 `res/values/` 目录下的所有XML内容。打包工具对XML内容进行提取，编译为二进制，并进行分类，其中，XML内容包含了不同语言的strings和styles，同时还包含没有被直接打包至 `resources.arsc` 文件的其他文件的路径，例如布局文件和图片文件。

OK，既然我们已经了解了APK的各个组成部分，那么我们就可以针对下面的三个组成部分，采取逐个击破的方式，达到缩减APK体积的目的：

+ [Java代码](#Java代码)
+ [资源文件](#资源文件)
+ [Native Code](#navtive-code)
+ [其他](#其他)

### Java代码
我们可以使用Proguard，在编译时对Java代码进行混淆，优化和压缩。Proguard对代码进行遍历，然后剔除其中未被使用的冗余的代码，并对类，属性，接口等进行重命名，从而达到瘦身的目的。

我们可以在 `build.gradle` 文件中配置Proguard。

```groovy
buildTypes {

        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }

    }
```

在未使用Proguard之前，我们的APK体积达到了13.9M。

![](https://i.loli.net/2018/03/27/5aba2fa673225.jpg)

开启Proguard后，APK的体积缩减到了10.1M。可见，效果还是很明显的。

![](https://i.loli.net/2018/03/27/5aba2fed61969.jpg)

除此之外，微信的工程师还开源了一个项目，可以直接对APK进行处理，缩小体积。具体的使用方法：https://github.com/shwenzhang/AndResGuard。

### 资源文件
#### Lint
由于Proguard只对Java代码起作用，对于资源文件，它就无能为力了。这个时候，`Lint`就派上用场了。Lint是一个静态的代码分析器，我们可以通过在 `Android Studio` -> `Analyze` -> `Inspect code...` ， 选择范围即可开始就检查。

![](https://i.loli.net/2018/03/27/5aba300cb98bb.jpg)

Lint在检查完成后，会提供一份详细的资源文件清单，并且将没有用到的资源在 `UnusedResources:Unused resources` 区域。只要我们没有通过反射使用这些资源，就可以放心的删掉它们了。

![](https://i.loli.net/2018/03/27/5aba302c54fb1.jpg)

下面是我根据lint的提示，剔除了部分无用资源后，APK的体积：

![](https://i.loli.net/2018/03/27/5aba304b72142.jpg)

Notice：需要注意的是，Lint并不会扫描 `assets` 目录，所以最好还是手动的检查一下 `assets` 目录下是否有未被使用的文件吧。

我们还可以通过配置 `shrinkResources` 来移除未使用的资源。不过，使用 `shrinkResources` 必须开启代码混淆。在处理过程中，`ProGuard` 会移除未被使用的代码，但是不会移除资源。而开启 `shrinkResources` 后，Gradle就会移除资源了。

下面为开启 `shrinkResources` 后APK体积的变化:

![](http://upload-images.jianshu.io/upload_images/2440049-707f600064af7ef3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

又缩小了一点点。

#### 图片压缩
对于资源文件的优化，其实占大头的还是图片。Android打包本身会对png进行无损压缩，在不那么影响图片显示效果的情况下，对图片进行有损压缩对apk体积的减小还是非常可观的。

我们先使用[TinyPng](https://tinypng.com/)对项目中四张比较大的图片进行压缩，可以看出，效果还是非常不错的。

![](https://i.loli.net/2018/03/27/5aba30a010793.jpg)

使用TinyPng后，我们还可以将图片转换为 `WebP` 格式，进一步缩小图片的体积。不过需要注意的是，WebP格式仅适用于Android 4.0+版本，如果你的应用需要兼容Android 2.3，那么需要额外的引入 `.so` 文件，apk的体积自然也会增加。所以，根据你的需求，权衡利弊吧。

在 `Android Studio` 中，选定需要转换的图片，然后右键鼠标，即可在菜单底部看到 `Convert to WebP` 的选项了，然后就可以进行转换的操作。

![](https://i.loli.net/2018/03/27/5aba30be718c4.jpg)

经过上面的处理，我们的App体积来到了9.6M。

![](https://i.loli.net/2018/03/27/5aba30e7335f1.jpg)

#### 矢量图
如果你的项目中使用 `appcompat` 兼容包，并且版本在23.2以上，那么，使用矢量图或许是个不错的选择。矢量图和分辨率无关，理论上支持任何级别的缩放。以一张常用的 点赞 图标为例，在 Android中使用 `Vector` 矢量图，大小为：

![](https://i.loli.net/2018/03/27/5aba3111543ea.jpg)

而它对应的xxxhdpi的png格式，大小为：

![](https://i.loli.net/2018/03/27/5aba3145e1533.jpg)

效果显而易见，但是这也并不是意味着，我们就可以将所有的图片替换为 矢量图 了 。我们来看一看 `ic_favorite_black_24dp.xml` 的源代码:

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
        android:width="24dp"
        android:height="24dp"
        android:viewportWidth="24.0"
        android:viewportHeight="24.0">
    <path
        android:fillColor="#FF000000"
        android:pathData="M12,21.35l-1.45,-1.32C5.4,15.36 2,12.28 2,8.5 2,5.42 4.42,3 7.5,3c1.74,0 3.41,0.81 4.5,2.09C13.09,3.81 14.76,3 16.5,3 19.58,3 22,5.42 22,8.5c0,3.78 -3.4,6.86 -8.55,11.54L12,21.35z"/>
</vector>
```

正如你所见，vector的颜色是单一的颜色，所以，vector 适用于图标等颜色单一的图片，如果是颜色比较复杂，那么vector很明显就不合适了。

#### 使用JPG
对于非透明的大图，JPG格式将会比PNG格式的大小有显著的优势，虽然不是绝对的，但是通常会减小到一半都不止。在启动页，活动页等之类的大图展示区采用JPG将是非常明智的选择。

对于图片的使用，Google的建议，简单来说就是：VD->WebP->Png->JPG

+ 如果是纯色的icon，那么用svg
+ 如果是两种以上颜色的icon，用webp
+ 如果webp无法达到效果，选择png
+ 如果图片没有alpha通道，可以考虑jpg

#### 复用图片
项目中我们还可能遇到图片内容相同，仅仅是颜色不同的情况。这个时候我们就可以使用Android提供的着色来完成，而不用提供好几套图片。例如在Android 5.0+上我们可以使用 `android:tint` 和 `android:tintMode`，在低版本中可以使用 `ColorFilter`。

对于那些内容颜色等都相同，只是方向不同的图片，我们可以只保留一中方向的，其他方向的图片通过代码实现。例如我们可以对上面提到的 `ic_favorite_black_24dp` 进行翻转。创建一个 `drawable`：

```xml
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/ic_favorite_black_24dp"
    android:fromDegrees="180"
    android:pivotX="50%"
    android:pivotY="50%"
    android:toDegrees="180" />
```

虽然Android设备的分辨率非常的多，但这并不代表着我们需要为每一种分辨率都准备一套资源。在显示差异不大的情况下，我们可以尽量复用一套图片资源，一套布局，然后再考虑特定屏幕密度。

#### 语言资源
说完了图片，我们还要来说说语言资源。对于大多数的应用，并不需要支持几十种的国际化。使用Gradle，对语言资源进行配置，也可以达到应用瘦身的目的。例如，我们的应用如果只需要支持中文和英语:

```groovy
android {
    defaultConfig {
        ...
        resConfigs "en", "zh"
        // 支持分辨率
        resConfigs "nodpi", "hdpi", "xhdpi", "xxhdpi", "xxxhdpi"
    }
}
```

### Native Code
如果你的App使用到了Native code,在不影响功能的前提下，可以考虑去除部分平台对应的代码，例如移除对 `armeabi、mips`的支持，

```groovy
android {
    ... 
    splits {
        abi {
            enable true
            reset()
            include 'armeabi-v7a', 'arm64-v8a', 'x86', 'x86_64'
            universalApk true
        }
    }

 }
```

当然，如果是特别大的原生库，我们还可以通过网络，从云端获取，而不直接打包在APK中。

### 其他
+ 别忘了最简单，也最容易忽视的，去除重复的依赖，或者是引用更加轻量级的库，也可以达到apk瘦身的效果。
+ 在Release版本中，去除那些只会在debug时才会出现的代码。
+ 对于那些使用频率很小的文件或者是图片，可以存放到云端后，通过网络加载。

### 结语
相对小的体积能够在用户安装前就给用户留下不错的印象，但是，这不并代表着我们可以为了追求APK体积的小巧而过度的牺牲用户体验。权衡利弊后，选择合适的，才是最重要的。

项目完整地址: [Espresso](https://github.com/TonnyL/Espresso) - Espresso is an express delivery tracking app designed with Material Design style, built on MVP(Model-View-Presenter) architecture with RxJava2, Retrofit2, Realm database and ZXing.

如果你觉得文章不错的话，请点个赞吧。有其他问题，可以通过以下方式联系我:

Zhihu: [https://www.zhihu.com/people/kirin-momo/](https://www.zhihu.com/people/kirin-momo/)

Slack: [https://androiddevsslack.slack.com](https://androiddevsslack.slack.com)

Weibo: [http://weibo.com/u/5313690193](http://weibo.com/u/5313690193)

Medium: [https://medium.com/@TonnyL](https://medium.com/@TonnyL)

Twitter: [https://twitter.com/TonnyLZTL](https://twitter.com/TonnyLZTL)