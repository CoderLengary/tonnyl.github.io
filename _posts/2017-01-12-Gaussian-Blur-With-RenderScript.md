---
layout: post # needs to be post
title: 使用 RenderScript 实现高斯模糊(毛玻璃/磨砂)效果
summary: 使用 RenderScript 实现高斯模糊(毛玻璃/磨砂)效果
featured-img: sleek
categories: [Android]
---


## 前言
逛 Instagram 的时候，偶然发现，Instagram 的对话框设计的很有意思，如下图：

![Instagram](https://i.loli.net/2018/03/27/5ab9f38463432.jpeg)

它的 dialog 的背景竟然是毛玻璃效果的，看起来很漂亮，恩，我是说对话框和迪丽热巴都漂亮😂。看到这么好的效果，当然就要开始搞事情了，自己动手实现差不多的效果。最终的实现效果如下图：

![Success - 1](https://i.loli.net/2018/03/27/5ab9f3c2b797c.png)

![Success - 2](https://i.loli.net/2018/03/27/5ab9f3e8275b4.jpeg)

分别实现了对话框背景的虚化和手动调节虚化程度。

## 实现方法对比
最开始想要实现毛玻璃效果时，我是一脸懵逼的，不知道如何下手。幸亏，有万能的 Google。搜索之后发现常见的实现方法有4种，分别是：

+ RenderScript
+ Java算法
+ NDK算法
+ openGL

处理一整张图片这么大计算量的工作，openGL 的性能最好，而用 java 实现肯定是最差的了。而 RenderScrip t和 NDK 的性能相当，但是你懂得，NDK 和 openGL 我无可奈何，综合考虑，RenderScript 应该是最适合的。

但并不是说 RenderScript 就是完全没有问题的：

+ 模糊半径(radius)越大，性能要求越高，模糊半径不能超过25，所以并不能得到模糊度非常高的图片。
+ ScriptIntrinsicBlur 在API 17时才被引入，如果需要在 Android 4.2 以下的设备上实现，就需要引入 RenderScript Support Library ，当然，安装包体积会相应的增大。

## RenderScript 实现
首先在 `app` 目录下 `build.gradle` 文件中添加如下代码：

```gradle
defaultConfig {
    applicationId "io.github.marktony.gaussianblur"
    minSdkVersion 19
    targetSdkVersion 25
    versionCode 1
    versionName "1.0"
    renderscriptTargetApi 19
    renderscriptSupportModeEnabled true
}
```

RenderScriptIntrinsics 提供了一些可以帮助我们快速实现各种图片处理的操作类，例如，`ScriptIntrinsicBlur`，可以简单高效实现 高斯模糊效果。

```java
public class RenderScriptGaussianBlur {
    private RenderScript renderScript;
    
    public RenderScriptGaussianBlur(@NonNull Context context) {
        this.renderScript = RenderScript.create(context);
    }
    
    public Bitmap gaussianBlur(@IntRange(from = 1, to = 25) int radius, Bitmap original) {
        Allocation input = Allocation.createFromBitmap(renderScript, original);
        Allocation output = Allocation.createTyped(renderScript, input.getType());
        ScriptIntrinsicBlur scriptIntrinsicBlur = ScriptIntrinsicBlur.create(renderScript, Element.U8_4(renderScript));
        scriptIntrinsicBlur.setRadius(radius);
        scriptIntrinsicBlur.setInput(input);
        scriptIntrinsicBlur.forEach(output);
        output.copyTo(original);
        return original;
    }
}
```

然后就可以直接使用 RenderScriptGaussianBlur ，愉快地根据 SeekBar 的值，实现不同程度的模糊了。

```java
public class MainActivity extends AppCompatActivity {
    private ImageView imageView;
    private ImageView container;
    private LinearLayout layout;
    private TextView textViewProgress;
    private RenderScriptGaussianBlur blur;
    
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        imageView = (ImageView) findViewById(R.id.imageView);
        container = (ImageView) findViewById(R.id.container);
        container.setVisibility(View.GONE);
        layout = (LinearLayout) findViewById(R.id.layout);
        layout.setVisibility(View.VISIBLE);
        SeekBar seekBar = (SeekBar) findViewById(R.id.seekBar);
        textViewProgress = (TextView) findViewById(R.id.textViewProgress);
        TextView textViewDialog = (TextView) findViewById(R.id.textViewDialog);
        blur = new RenderScriptGaussianBlur(MainActivity.this);
        
        seekBar.setMax(25);
        seekBar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            
			@Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                textViewProgress.setText(String.valueOf(progress));
            }
            
			@Override
            public void onStartTrackingTouch(SeekBar seekBar) {
            }
            
			@Override
            public void onStopTrackingTouch(SeekBar seekBar) {
                int radius = seekBar.getProgress();
                if (radius < 1) {
                    radius = 1;
                }
                Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.image);
                imageView.setImageBitmap(blur.gaussianBlur(radius, bitmap));
            }
        });
        textViewDialog.setOnClickListener(new View.OnClickListener() {
            
			@Override
            public void onClick(View v) {
                container.setVisibility(View.VISIBLE);
                layout.setDrawingCacheEnabled(true);
                layout.setDrawingCacheQuality(View.DRAWING_CACHE_QUALITY_LOW);
                Bitmap bitmap = layout.getDrawingCache();
                container.setImageBitmap(blur.gaussianBlur(25, bitmap));
                layout.setVisibility(View.INVISIBLE);
                
                AlertDialog dialog = new AlertDialog.Builder(MainActivity.this).create();
                dialog.setTitle("Title");
                dialog.setMessage("Message");
                dialog.setButton(DialogInterface.BUTTON_POSITIVE, "OK", new DialogInterface.OnClickListener() {
                    
					@Override
                    public void onClick(DialogInterface dialog, int which) {
                        dialog.dismiss();
                    }
                });
                dialog.setButton(DialogInterface.BUTTON_NEGATIVE, "Cancel", new DialogInterface.OnClickListener() {
                    
					@Override
                    public void onClick(DialogInterface dialog, int which) {
                    }
                });
                dialog.setOnCancelListener(new DialogInterface.OnCancelListener() {
                    
					@Override
                    public void onCancel(DialogInterface dialog) {
                    }
                });
                dialog.setOnCancelListener(new DialogInterface.OnCancelListener() {
                    
					@Override
                    public void onCancel(DialogInterface dialog) {
                        container.setVisibility(View.GONE);
                        layout.setVisibility(View.VISIBLE);
                    }
                });
                dialog.show();
            }
        });
    }
}
```

在代码里做了一些 view 的可见性的操作，比较简单，相信你能看懂的。和 Instagram 中dialog 的实现有一点不同的是，我没有截取整个页面的 bitmap ，只是截取了 actionbar 下的内容，如果一定要实现一样的效果，调整一下页面的布局就可以了。这里不多说了。

## 轮子
除了 RenderScript 外，还有一些优秀的轮子：

+ [500px-android-blur](https://github.com/500px/500px-android-blur)
+ [Blurry](https://github.com/wasabeef/Blurry)
+ [android-stackblur](https://github.com/kikoso/android-stackblur)
+ [FastBlur](https://github.com/paveldudka/blurring/blob/master/app/src/main/java/com/paveldudka/util/FastBlur.java):Java算法实现

[BlurTestAndroid](https://github.com/patrickfav/BlurTestAndroid) 对不同类库的实现方式、采取的算法和所耗费的时间做了统计和比较，你也可以下载它的 demo app，自行测试。

![BlurTestAndroid](https://i.loli.net/2018/03/27/5ab9f5fe1b84b.png)

示例代码在这里：[GaussianBlur](https://github.com/TonnyL/GaussianBlur)