---
layout: post # needs to be post
title: 利用 ViewPager 实现 Material Design 风格炫酷引导页
summary: 当我们第一次打开一个App，首先映入眼帘的就是引导页了。
featured-img: raindrop_glass
categories: [Android]
---

当我们第一次打开一个App，首先映入眼帘的就是引导页了。

引导页向我们展示了一些可以滑动的页面，并在页面中以图片方式展示了这个App的一些亮点。要实现这个效果，网上更多的是通过引入第三方库的形式，但是，这哪里有我们自己动手撸一个来的痛快呢？所以，今天我将手 ~~摸~~ 把手的带大家实现一个符合Material Design标准的炫酷引导页。

### 效果预览
我们首先来看看Google Docs是实现的效果:

![Google Docs](https://i.loli.net/2018/03/27/5aba2cd1cd020.gif)

下面是我们将要实现的效果:

![Espresso](https://i.loli.net/2018/03/27/5aba2cf086716.gif)

### 设计规范
在Material Design官方文档中，关于引导页的部分是这样说明的：

> Onboarding is a virtual unboxing experience that helps users get started with an app’s UI. It should be brief and enhance the use of the app.

简单来说，引导应该是一个简短的过程，能够帮助用户理解App的使用。

那么，我们应该什么时候使用引导呢？按照规范，当我们的应用添加了新的特色功能或者有重大改变时，应该使用引导页面。对于那些用户本来就期待的，很普通的功能，就不要滥用引导页了。

更多关于设计规范的内容，请访问: https://material.io/guidelines/growth-communications/onboarding.html.

### 开始吧
本次的教程主要分为以下三步：

1. 使用ViewPager创建引导页
2. 当页面滑动时，控制背景色的渐变
3. 当应用第一次加载时，显示引导页

现在就开始吧。

首先当然是要添加依赖了。在`build.gradle`文件中，添加支持库的依赖。

```groovy
dependencies {
    compile 'com.android.support:appcompat-v7:25.3.1'
    compile 'com.android.support:design:25.3.1'
    // Other dependencies here.
}
```

然后我们就可以利用`Android Studio`自带的模板，创建一个`Tabbed Activity`: 在我们的Java包点击鼠标右键 -> `New` -> `Activity` -> `Tabbed Activity`，然后选择`Navigation Style`为** `Swipe Views (ViewPager) ` ** .

![](https://i.loli.net/2018/03/27/5aba2d1714073.jpg)

![](https://i.loli.net/2018/03/27/5aba2d3e6a5f6.jpg)

#### Tabbed Activity Template
首先，我们来看看我们现有有了些什么：

1. Activity(在我的工程中为`OnboardingActivity`);
2. `SectionsPagerAdapter`, ViewPager的适配器;
3. `PlaceholderFragment`, 用于相应每一个滑动页面的布局。

现在我们就可以根据需要对代码进行改造了。打开`OnboardingActivity`对应的布局文件`activity_onboarding.xml`，完成下面的工作：

1. 移除`Toolbar`;
2. 添加底部导航按钮部分(用于控制ViewPager)。

下面就是底部导航按钮：

![](https://i.loli.net/2018/03/27/5aba2d67d9b69.jpg)

#### 布局
底部导航按钮的类型包括：

1. 圆形的页面位置指示器;
2. 上一页/下一页按钮;
3. 完成按钮(理应包括跳过按钮，后面我会讲解为什么我没有做跳过按钮)。

下面是`activity_onboarding.xml`文件的源码：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main_content"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/colorPrimary"
    tools:context=".ui.onboarding.OnboardingActivity">

    <android.support.v4.view.ViewPager
        android:id="@+id/container"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:paddingBottom="?attr/actionBarSize"
        app:layout_behavior="@string/appbar_scrolling_view_behavior" />

    <!--分割线-->
    <View
        android:layout_width="match_parent"
        android:layout_height="1dp"
        android:layout_gravity="bottom"
        android:layout_marginBottom="?attr/actionBarSize"
        android:alpha="0.12"
        android:background="@android:color/white" />

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:layout_gravity="bottom"
        android:paddingEnd="@dimen/activity_horizontal_margin"
        android:paddingLeft="@dimen/activity_horizontal_margin"
        android:paddingRight="@dimen/activity_horizontal_margin"
        android:paddingStart="@dimen/activity_horizontal_margin">


        <ImageButton
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/imageButtonPre"
            style="@style/Widget.AppCompat.Button.Borderless"
            android:contentDescription="@string/onboarding_pre_button_description"
            android:layout_gravity="start|center"
            android:padding="@dimen/activity_horizontal_margin"
            android:src="@drawable/ic_chevron_left_white_24dp"
            android:visibility="gone"
            android:tint="@android:color/white" />

        <!--圆形指示器-->
        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:orientation="horizontal">

            <ImageView
                android:id="@+id/imageViewIndicator0"
                android:layout_width="8dp"
                android:layout_height="8dp"
                android:background="@drawable/onboarding_indicator_selected" />

            <ImageView
                android:id="@+id/imageViewIndicator1"
                android:layout_width="8dp"
                android:layout_height="8dp"
                android:layout_marginEnd="@dimen/activity_margin_half"
                android:layout_marginRight="@dimen/activity_margin_half"
                android:layout_marginLeft="@dimen/activity_margin_half"
                android:layout_marginStart="@dimen/activity_margin_half"
                android:background="@drawable/onboarding_indicator_unselected" />

            <ImageView
                android:id="@+id/imageViewIndicator2"
                style="@style/Widget.AppCompat.Button.Borderless"
                android:layout_width="8dp"
                android:layout_height="8dp"
                android:background="@drawable/onboarding_indicator_unselected" />

        </LinearLayout>

        <android.support.v7.widget.AppCompatButton
            android:id="@+id/buttonFinish"
            style="@style/Widget.AppCompat.Button.Borderless"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="end|center"
            android:text="@string/onboarding_finish_button_description"
            android:contentDescription="@string/onboarding_finish_button_description"
            android:textColor="@android:color/white"
            android:visibility="gone" />

        <ImageButton
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            style="@style/Widget.AppCompat.Button.Borderless"
            android:id="@+id/imageButtonNext"
            android:contentDescription="@string/onboarding_next_button_description"
            android:layout_gravity="end|center"
            android:padding="@dimen/activity_horizontal_margin"
            android:src="@drawable/ic_chevron_right_white_24dp"
            android:tint="@android:color/white" />

    </FrameLayout>

</android.support.design.widget.CoordinatorLayout>
```

#### Fragment
下面是我的`fragment_onboarding.xml`的全部代码:

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".ui.onboarding.OnboardingFragment">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/colorPrimary">

        <ImageView
            android:id="@+id/section_img"
            android:layout_width="192dp"
            android:layout_height="192dp"
            android:layout_gravity="center"
            android:adjustViewBounds="true"
            android:src="@drawable/ic_notifications_black_24dp"
            android:alpha="0.30"
            android:scaleType="fitCenter" />

    </FrameLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom"
        android:gravity="center"
        android:orientation="vertical"
        android:padding="@dimen/activity_horizontal_margin">

        <android.support.v7.widget.AppCompatTextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/section_label"
            android:textAlignment="center"
            style="@style/TextAppearance.AppCompat.Headline"
            android:textColor="@android:color/white"
            tools:text="Page One" />

        <android.support.v7.widget.AppCompatTextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/section_intro"
            style="@style/TextAppearance.AppCompat.Body1"
            android:layout_marginTop="@dimen/activity_horizontal_margin"
            android:alpha="0.7"
            android:gravity="center"
            android:text="Your awesome description about the app"
            android:textColor="@android:color/white" />

    </LinearLayout>

</FrameLayout>
```

![](https://i.loli.net/2018/03/27/5aba2da71dc2a.jpg)

调整完布局文件之后，我们还可以对Java代码进行一定的调整，便于阅读，当然，这是可选的。我们可以把`PlaceholderFragment`和`SectionsPagerAdapter`单独抽取出来，然后统一命名，这样更加便于理解和管理，在后期的维护中也更加容易。

![](https://i.loli.net/2018/03/27/5aba2db08544d.jpg)

#### 控制ViewPager的颜色
由于`ViewPager`已经和`SectionsPagerAdapter`(`OnboardingPagerAdapter`)绑定了，所以不用我们就不用担心这个问题了。

当我们滑动页面时，颜色应该自动的更新。也就是说，Pager的颜色应该根据滑动的距离自行调整。

我们有三个页面，每个页面搭配不同的颜色。

```java
private int bgColors[];

bgColors = new int[]{ContextCompat.getColor(this, R.color.colorPrimary),
                ContextCompat.getColor(this, R.color.cyan_500),
                ContextCompat.getColor(this, R.color.light_blue_500)};
```

然后可以通过为`ViewPager`添加`OnPageChangeListener`做出相应的改变。

```java
viewPager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {

                @Override
                public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
                    int colorUpdate = (Integer) new ArgbEvaluator().evaluate(positionOffset, bgColors[position], bgColors[position == 2 ? position : position + 1]);
                    viewPager.setBackgroundColor(colorUpdate);
                }

                @Override
                public void onPageSelected(int position) {
                    currentPosition = position;
                    updateIndicators(position);
                    viewPager.setBackgroundColor(bgColors[position]);
                    buttonPre.setVisibility(position == 0 ? View.GONE : View.VISIBLE);
                    buttonNext.setVisibility(position == 2 ? View.GONE : View.VISIBLE);
                    buttonFinish.setVisibility(position == 2 ? View.VISIBLE : View.GONE);
                }

                @Override
                public void onPageScrollStateChanged(int state) {

                }
            });
```

我们创建了一个`ArgbEvaluator`用于更新颜色，在前一个页面和后一个页面的颜色之间进行过渡。

想必你也猜到了，`updateIndicators()`方法通过切换两个不同的`drawable`来更新指示器。

```java
private void updateIndicators(int position) {
        for (int i = 0; i < indicators.length; i++) {
            indicators[i].setBackgroundResource(
                    i == position ? R.drawable.onboarding_indicator_selected : R.drawable.onboarding_indicator_unselected
            );
        }
    }
```

注意：

`bgColors[position == 2 ? position : position + 1]`提示当前是否为最后一个位置，如果是，则不再增加，否则会出现`ArrayIndexOutOfBoundsException`数组越界异常。数组的范围为「0」 - 「n-1」。

`currentPosition`变量用于追踪ViewPager的当前位置，我们需要在Pre和Next按钮中对`currentPosition`进行减或者加的操作。

最后，当页面处于最后一页时，需要将FINISH按钮的状态设置为可见。

#### 指示器Drawable
onboarding_indicator_selected.xml:

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <corners android:radius="100dp" />
    <solid android:color="@color/indicator_selected" />
</shape>
```

onboarding_indicator_unselected.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <corners android:radius="100dp" />
    <solid android:color="@color/indicator_unselected" />
</shape>
```

### 调用与触发首次加载
App第一次启动是展示引导页的最佳时机，这样就可以提示用户此App有哪些值得注意的特色功能；但是，从第二次启动开始，App就不应该展示引导页了。我们可以通过在`SharedPreference`中存放一个`boolean`值来判断App是否是第一次加载。

当用户按下FINISH按钮后，结束引导，将`SettingsUtil.KEY_FIRST_LAUNCH`对应的值修改为`false`。

```java
buttonFinish.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    SharedPreferences.Editor ed = sp.edit();
                    ed.putBoolean(SettingsUtil.KEY_FIRST_LAUNCH, false);
                    ed.apply();
                    navigateToMainActivity();
                }
            });
```

如果不是第一次加载，则直接进入`MainActivity`。

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    final SharedPreferences sp = PreferenceManager.getDefaultSharedPreferences(this);
    if (sp.getBoolean(SettingsUtil.KEY_FIRST_LAUNCH, true)) {

        setContentView(R.layout.activity_onboarding);
			
		// Do other things here.
	} else {

        navigateToMainActivity();

        finish();
    }

private void navigateToMainActivity() {
        Intent i = new Intent(this, MainActivity.class);
        i.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        startActivity(i);
    }			
```

现在我们把项目跑起来，不出意外的话，你就会看到[效果预览](#效果预览)里的效果啦。

#### 高级用法
如果你还需要对页面的切换动画进行定制，请访问：https://developer.android.com/training/animation/screen-slide.html#pagetransformer。Medium上`Michell Bak`的这篇文章或许也能帮到你:[Creating an intro screen for your app using ViewPager and PageTransformer](https://android.jlelse.eu/creating-an-intro-screen-for-your-app-using-viewpager-pagetransformer-9950517ea04f).

### 其他优秀开源库
如果你需要更加简单的实现方式，你还可以选择：

+ [AppIntro](https://github.com/apl-devs/AppIntro) - Make a cool intro for your Android app.
+ [material-intro-screen](https://github.com/TangoAgency/material-intro-screen) - Inspired by Heinrich Reimer Material Intro and developed with love from scratch.
+ [MaterialIntroTutorial](https://github.com/riggaroo/MaterialIntroTutorial) - Library demonstrating a material intro tutorial much like the ones on Google Sheets.

### One More Thing...
还记得我之前立的flag吗？为什么我没有做`SKIP`跳过按钮？因为在我的引导页中，并不仅仅只是向用户展示了App的特色功能，实际上，在展示页面的同时还完成了一部分数据的初始化工作，向数据库中插入了几百条数据，而这很明显是耗时操作，放在主线程肯定是不合适的，所以我选择了使用`AsyncTask`，当任务完成后再将Button的状态设置为enable。

```java
private class InitCompaniesDataTask extends AsyncTask<Void, Void, Void> {

    @Override
    protected Void doInBackground(Void... params) {
        CompaniesRepository.getInstance(CompaniesLocalDataSource.getInstance())
                .initData();
        return null;
    }

    @Override
    protected void onPostExecute(Void aVoid) {
        super.onPostExecute(aVoid);
        handler.sendEmptyMessage(MSG_DATA_INSERT_FINISH);
    }

    @Override
    protected void onProgressUpdate(Void... values) {

    }
}

private Handler handler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
            case MSG_DATA_INSERT_FINISH:
                buttonFinish.setText(R.string.onboarding_finish_button_description);
                buttonFinish.setEnabled(true);
                break;
        }
    }
};
```

(事实上，我所使用的`Realm`数据库在数据插入的速度方面还是很乐观的，但是为了保险起见(避免引起ANR)，我仍然还是选择了`AsyncTask`，如果你有更好的解决方式，请不吝赐教。)

### 写在最后
项目完整地址: [Espresso](https://github.com/TonnyL/Espresso) - Espresso is an express delivery tracking app designed with Material Design style, built on MVP(Model-View-Presenter) architecture with RxJava2, Retrofit2, Realm database and ZXing.

参考:[Onboarding with Android ViewPager: The Google Way](http://blog.iamsuleiman.com/onboarding-android-viewpager-google-way/)

如果你觉得文章不错的话，请点个赞吧。有其他问题，可以通过以下方式联系我:

Zhihu: [https://www.zhihu.com/people/kirin-momo/](https://www.zhihu.com/people/kirin-momo/)

Slack: [https://androiddevsslack.slack.com](https://androiddevsslack.slack.com)

Weibo: [http://weibo.com/u/5313690193](http://weibo.com/u/5313690193)

Medium: [https://medium.com/@TonnyL](https://medium.com/@TonnyL)

Twitter: [https://twitter.com/TonnyLZTL](https://twitter.com/TonnyLZTL)