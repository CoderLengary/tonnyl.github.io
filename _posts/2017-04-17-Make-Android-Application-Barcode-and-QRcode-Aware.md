---
layout: post # needs to be post
title: Make Android Application Barcode and QRcode Aware
summary: Android实现条形码与二维码扫描
featured-img: shane-rounce-205187
categories: [Android]
---

# Android实现条形码与二维码扫描

![ZXing](https://i.loli.net/2018/03/27/5aba2e743ea4f.jpg)

在本次教程中，我们将使用ZXing库让我们的app实现条形码和二维码的扫描。

## 为什么选择ZXing
要实现条形码和二维码扫描功能，常用的有下面几种方法：

+ 使用Google Play Service和[Mobile Vision APIs](https://developers.google.com/vision/)。
+ 使用[barcodescanner](https://github.com/dm77/barcodescanner)库。
+ 使用[ZXing](https://github.com/zxing/zxing)库。

我们来仔细分析以上三种方法各自的优缺点。
+ 第一种，使用Google Play Service和Mobile Vision APIs。不合适，因为不是所有的用户设备都有安装Google Play Service，特别是在中国。
+ 第二种，使用barcodescanner。也不合适，尽管barcodescanner的导入非常简单，识别也很快，但是UI的调整比较困难。特别是当UI非常复杂时，你就需要拉取整个项目的源码然后自己进行定制。
+ 现在就只剩下ZXing了。ZXing库由谷歌出品，所以在短期内应该不会停止维护。很多应用，报错barcodescanner，都是基于ZXingk开发的。使用ZXing我们不用担心用户没有安装Google Play Service，而聚焦于开发和处理结果本身。

## ZXing是什么
> Zxing是Google提供的关于条码（一维码、二维码）的解析工具，提供了二维码的生成与解析的方法。

## 设置项目
### 添加依赖
在你的`build.gradle`文件(**application级别**)中，添加`ZXing core`的库。

```groovy
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    // 其他的库在这里...
    compile 'com.google.zxing:core:3.3.0'
    testCompile 'junit:junit:4.12'
}
```

### 添加必要的权限
在你的`AndroidManifest.xml`文件中，添加下面的代码：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="io.github.marktony.espresso">

    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.VIBRATE" />

     <application ...
```

### 导入必要的类和资源文件
将[整个包](https://github.com/TonnyL/Espresso/tree/master/mobile/src/main/java/io/github/marktony/espresso/zxing)复制到你的项目中。

![ZXing package](https://i.loli.net/2018/03/27/5aba2eac9fb41.jpg)

**注意**:上面的包中的代码我做了简化，如果你需要ZXing示例项目此部分的完整的代码，请访问[这里](https://github.com/zxing/zxing/tree/master/android)，并下载相应的代码。

代码文件导入完成后，我们还需要导入下面的资源文件：

+ [`res/layout/activity_scan`](https://github.com/TonnyL/Espresso/blob/master/mobile/src/main/res/layout/activity_scan.xml) -  [CaptureActivity](https://github.com/TonnyL/Espresso/blob/master/mobile/src/main/java/io/github/marktony/espresso/zxing/CaptureActivity.java)的布局文件.
+ [`res/values/zxing_ids.xml`](https://github.com/TonnyL/Espresso/blob/master/mobile/src/main/res/values/zxing_ids.xml) - zxing id的集合.
+ [`res/raw/beep.ogg`](https://github.com/TonnyL/Espresso/tree/master/mobile/src/main/res/raw) - 当扫描成功后播放的声音文件.
+ [`res/drawable/...`](https://github.com/TonnyL/Espresso/tree/master/mobile/src/main/res/drawable) - 共3个文件: [scan_line.png](https://github.com/TonnyL/Espresso/blob/master/mobile/src/main/res/drawable/scan_line.png), [qr_code_bg.9.png](https://github.com/TonnyL/Espresso/blob/master/mobile/src/main/res/drawable/qr_code_bg.9.png), [shadow.png](https://github.com/TonnyL/Espresso/blob/master/mobile/src/main/res/drawable/shadow.png).

## 开动吧
现在我们从`MainActivity`启动`CaptureActivity`并期待获取相应的结果:

```java
public class MainActivity extends AppCompatActivity {

    public final static int SCANNING_REQUEST_CODE = 1;

    private TextView textView;

    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        textView = (TextView) findViewById(R.id.textView);

        textView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(MainActivity.this, CaptureActivity.class);
                intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
                startActivityForResult(intent, SCANNING_REQUEST_CODE);
            }
        });
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        switch (requestCode) {
            case SCANNING_REQUEST_CODE:
                if (resultCode == RESULT_OK) {
                    final Bundle bundle = data.getExtras();
                    Handler handler = new Handler(Looper.getMainLooper());
                    handler.post(new Runnable() {
                        @Override
                        public void run() {
                            textView.setText(bundle.getString("result"));
                        }
                    });
                }
                break;
            default:
                break;
        }
    }
}

```

我们可以在`onActivityResult()`方法中得到相应的结果。

注意: 如果你的app的target api level是23或者更高, **不要**
忘记适配运行时权限因为我们的app需要用到相机。

## 测试
现在我们可以测试app是否成功运行了。访问 [qr-code-generator.com](http://www.qr-code-generator.com/)生成你想要的二维码，然后进行愉快的测试吧。我们可以期待获得下面的结果:

![Test](https://i.loli.net/2018/03/27/5aba2ee00f0c1.jpg)

## 结论
在这篇教程中，我们使用ZXing库实现了条形码和二维码的扫描。在你自己的app中，你可能需要对扫描的结果进行进一步的处理，例如加载URL或者是从第三方数据源中获取数据等等。

你可以在访问[Espresso](https://github.com/TonnyL/Espresso)项目主页下载全部源代码。Espresso是一个基于MVP架构，采用RxJava2, Retrofit2, Realm3.0, ZXing开发的快递查询App。

## 参考
[ZXing](https://github.com/zxing/zxing) - ZXing("Zebra Crossing")项目官方主页。

## 联系我
如果你对这篇文章或者项目有任何问题，可以通过以下方式联系我：

GitHub: https://github.com/TonnyL

Zhihu: https://www.zhihu.com/people/kirin-momo/

Slack: https://androiddevsslack.slack.com

Weibo: http://weibo.com/u/5313690193

Medium: https://medium.com/@TonnyL

Twitter: https://twitter.com/TonnyLZTL


# Make android application barcode and QR code aware

In this tutorial, we will carry out the barcode and QR code scanning within our app using ZXing library.

## Why ZXing
Here are the several ways to carry out this function below:

+ Using Google Play Service and the [Mobile Vision APIs](https://developers.google.com/vision/).
+ Using [barcodescanner](https://github.com/dm77/barcodescanner) library.
+ Using [ZXing](https://github.com/zxing/zxing) library.

Let's analyze the positive and negative points of the 3 ways above.
+ The first one, using Google Play Service and the Mobile Vision APIs is not suitable, because not every user has installed the Google Play Service on his devices especially in China.
+ The second one, using barcodescanner library is not suitable either. Though the integration of it is simple and it is fast, the adjustment of UI is not easy. Particularly when the UI is much complex, you need to pull down the source code and custom it by yourself.
+ Then only the ZXing left. The ZXing library is produced by Google, so it will not be abandoned in a short time. Many apps including the barcodescanner are based on it. Using ZXing library, we do not need to worry about users without Google Play Service installed, and we can make user scans easier and faster, and focus our development on handling the results.

## What is ZXing
> ZXing ("zebra crossing") is an open-source, multi-format 1D/2D barcode image processing library implemented in Java, with ports to other languages.

## Set up the project
### Add the dependences
In your build.gradle file(**App module level**), add the `ZXing core` library to it.

```groovy
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    // Other dependencies here...
    compile 'com.google.zxing:core:3.3.0'
    testCompile 'junit:junit:4.12'
}
```

### Ask the required permissions
In your `AndroidManifest.xml` file, add the code below.

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="io.github.marktony.espresso">

    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.VIBRATE" />

     <application ...
```

### Import the needed class and resource files
Copy [the whole package](https://github.com/TonnyL/Espresso/tree/master/mobile/src/main/java/io/github/marktony/espresso/zxing) to your project.

![ZXing package](https://i.loli.net/2018/03/27/5aba2eac9fb41.jpg)

**Notice**: I have simplified the classes and code in the package above, if you need the complete package, visit the [ZXing samples](https://github.com/zxing/zxing/tree/master/android) here and download the corresponding code.

After import the code, we need to add the resource files below.

+ [`res/layout/activity_scan`](https://github.com/TonnyL/Espresso/blob/master/mobile/src/main/res/layout/activity_scan.xml) - The layout file of [CaptureActivity](https://github.com/TonnyL/Espresso/blob/master/mobile/src/main/java/io/github/marktony/espresso/zxing/CaptureActivity.java).
+ [`res/values/zxing_ids.xml`](https://github.com/TonnyL/Espresso/blob/master/mobile/src/main/res/values/zxing_ids.xml) - The collection of zxing ids.
+ [`res/raw/beep.ogg`](https://github.com/TonnyL/Espresso/tree/master/mobile/src/main/res/raw) - The sound that app will make when recognize the code successfully.
+ [`res/drawable/...`](https://github.com/TonnyL/Espresso/tree/master/mobile/src/main/res/drawable) - Three files in total: [scan_line.png](https://github.com/TonnyL/Espresso/blob/master/mobile/src/main/res/drawable/scan_line.png), [qr_code_bg.9.png](https://github.com/TonnyL/Espresso/blob/master/mobile/src/main/res/drawable/qr_code_bg.9.png), [shadow.png](https://github.com/TonnyL/Espresso/blob/master/mobile/src/main/res/drawable/shadow.png).

## Wrap it up
Now we start the `CaptureActivity` from `MainActivity` and expect a result by:

```java
public class MainActivity extends AppCompatActivity {

    public final static int SCANNING_REQUEST_CODE = 1;

    private TextView textView;

    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        textView = (TextView) findViewById(R.id.textView);

        textView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(MainActivity.this, CaptureActivity.class);
                intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
                startActivityForResult(intent, SCANNING_REQUEST_CODE);
            }
        });
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        switch (requestCode) {
            case SCANNING_REQUEST_CODE:
                if (resultCode == RESULT_OK) {
                    final Bundle bundle = data.getExtras();
                    Handler handler = new Handler(Looper.getMainLooper());
                    handler.post(new Runnable() {
                        @Override
                        public void run() {
                            textView.setText(bundle.getString("result"));
                        }
                    });
                }
                break;
            default:
                break;
        }
    }
}

```

We take care of the result in function `onActivityResult()`.

Notice: If your target api level is 23 or higher, **DO NOT** forget adapt the runtime permissions because our app need to use the camera.
Tip: As you can see in my source code, I did not launch the `CaptureActivity` in `AddPackageFragment` instead of `MainActivity`, so it is a little bit different to the code above.

## Test
Now we can test our app if it works well. Go to [qr-code-generator.com](http://www.qr-code-generator.com/) and test it. We should expect something just this:

![Test](https://i.loli.net/2018/03/27/5aba2ee00f0c1.jpg)

## Conclusion
In this tutorial, we have run through the process of facilitating barcode and QR code scanning within Android apps using the ZXing library. In your own apps, you might want to carry out further processing on the retrieved scan results, such as loading URLs or looking the data up in a third party data source.

You can download the complete source code of [Espresso](https://github.com/TonnyL/Espresso), it is an express delivery tracking app designed with Material Design style, built on MVP(Model-View-Presenter) architecture with RxJava2, Retrofit2, Realm database and ZXing.

## Reference
[ZXing](https://github.com/zxing/zxing) - Official ZXing ("Zebra Crossing") project home.

## Contact
If you have any problem about this article or the Espresso project, contact me on:

GitHub: https://github.com/TonnyL

Zhihu: https://www.zhihu.com/people/kirin-momo/

Slack: https://androiddevsslack.slack.com

Weibo: http://weibo.com/u/5313690193

Medium: https://medium.com/@TonnyL

Twitter: https://twitter.com/TonnyLZTL