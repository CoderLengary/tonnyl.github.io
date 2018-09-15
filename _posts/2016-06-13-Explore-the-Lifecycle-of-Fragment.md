---
layout: post # needs to be post
title: 探索Fragment的生命周期
summary: Fragment 和 Activity 类似，也有自己的生命周期，并且 fragment 的生命周期和 activity 的生命周期特别相似。
featured-img: design
categories: [Android]
---

Fragment 和 Activity 类似，也有自己的生命周期，并且 fragment 的生命周期和 activity 的生命周期特别相似。

Activity 的生命周期由四种状态，运行、暂停、停止、和销毁，类似的，Fragment 也有这四种状态，只是在一些细小的地方有所不同。

Fragment 生命周期图(来自 Android 官网)

![Lifecycle of Fragment](https://i.loli.net/2018/03/27/5ab9ca117b4da.jpg)

下面直接通过工程来认识 Fragment 的生命周期

首先新建 FragmentDemo 工程。

`MainActivity.java`

```java
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentTransaction;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;

public class MainActivity extends AppCompatActivity {
    
    private Button btnAddFragment;
    private FragmentManager fragmentManager;
    private FragmentTransaction fragmentTransaction;
    
    public static final String TAG = "MainActivity";
    
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        initViews();
        initData();
        
        btnAddFragment.setOnClickListener(v -
        	MyFragment fragment = new MyFragment();
        	fragmentTransaction.replace(R.id.container,fragment);
        	fragmentTransaction.commit();
        });
        
        Log.d(TAG,"onCreate");
    }
    
    private void initData() {
        fragmentManager = getSupportFragmentManager();
        fragmentTransaction = fragmentManager.beginTransaction();
    }
    
    private void initViews() {
        btnAddFragment = (Button) findViewById(R.id.btn_add_fragment);
    }
    
}
```

代码很简单，加载相应的布局，通过监听 button 的点击事件，加载 fragment 。

`activity_main.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.marktony.fragmentdemo.MainActivity">
    
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/btn_add_fragment"
        android:text="add_fragment"/>
        
</RelativeLayout>
```

`MyFragment.java`

```java
import android.content.Context;
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v4.app.Fragment;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

/**
 * Created by lizhaotailang on 2016/6/13.
 */
public class MyFragment extends Fragment {

    public static final String TAG = "MY_FRAGMENT";
    
    // empty constructor
    public MyFragment(){
    }
    
	@Override
    public void onAttach(Context context) {
        super.onAttach(context);
        Log.d(TAG,"onAttach");
    }
    
	@Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d(TAG,"onCreate");
    }
    
	@Nullable 
	@Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        Log.d(TAG,"onCreateView");
        return inflater.inflate(R.layout.fragment_my,container,false);
    }
    
	@Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        Log.d(TAG,"onActivityCreated");
    }
    
	@Override
    public void onStart() {
        super.onStart();
        Log.d(TAG,"onStart");
    }
    
	@Override
    public void onResume() {
        super.onResume();
        Log.d(TAG,"onResume");
    }
    
	@Override
    public void onPause() {
        super.onPause();
        Log.d(TAG,"onPause");
    }
    
	@Override
    public void onStop() {
        super.onStop();
        Log.d(TAG,"onStop");
    }
    
	@Override
    public void onDestroyView() {
        super.onDestroyView();
        Log.d(TAG,"onDestroyView");
    }
    
	@Override
    public void onDestroy() {
        super.onDestroy();
        Log.d(TAG,"onDestroy");
    }
    
	@Override
    public void onDetach() {
        super.onDetach();
        Log.d(TAG,"onDetach");
    }
    
}
```

MyFragment 中复写了一些方法，每个方法被调用时打印日志。

ok，大功告成，现在就跑到手机上。

观察打印日志，可以看到:

```bash
com.marktony.fragmentdemo D/MainActivity: onCreate
```

按下按钮时:

```bash
com.marktony.fragmentdemo D/MY_FRAGMENT: onAttach
com.marktony.fragmentdemo D/MY_FRAGMENT: onCreate
com.marktony.fragmentdemo D/MY_FRAGMENT: onCreateView
com.marktony.fragmentdemo D/MY_FRAGMENT: onActivityCreated
com.marktony.fragmentdemo D/MY_FRAGMENT: onStart
com.marktony.fragmentdemo D/MY_FRAGMENT: onResume
```

此时按下 home 键:

```bash
com.marktony.fragmentdemo D/MY_FRAGMENT: onPause
com.marktony.fragmentdemo D/MY_FRAGMENT: onStop
```

重新回到应用:

```bash
com.marktony.fragmentdemo D/MY_FRAGMENT: onStart
com.marktony.fragmentdemo D/MY_FRAGMENT: onResume
```

按下 back 键，此时 fragment 和 activity 均被销毁。

```bash
com.marktony.fragmentdemo D/MY_FRAGMENT: onPause
com.marktony.fragmentdemo D/MY_FRAGMENT: onStop
com.marktony.fragmentdemo D/MY_FRAGMENT: onDestroyView
com.marktony.fragmentdemo D/MY_FRAGMENT: onDestroy
com.marktony.fragmentdemo D/MY_FRAGMENT: onDetach
```

观察日志，不难发现:

当 fragment 被创建时，会依次经历以下状态：

+ onAttach()
+ onCreate()
+ onCreateView()
+ onActivityCreated()

当 fragment 处于可见状态时，会经历:

+ onStart()
+ onResume()

不可见时，经历了:

+ onPause()
+ onStop()

当 fragment 被销毁时（或相应的 activity 被销毁时）

+ onPause()
+ onStop()
+ onDestroyView()
+ onDestroy()
+ onDetach()

由此可见，fragment 的生命周期和 activity 的生命周期真的非常的相似，只是增加了一些新的状态。

在 fragment 中也是可以通过 `onSaveInstanceState()` 方法保存数据的，因为进入停止状态的 fragment 可能由于系统内存不足而被回收，保存下来数据可以在`onCreate()`, `onCreateView()`, `onActivityCreate()` 三个方法中重新得到，他们都含有一个 `Bundle` 类型的 `saveInstanceState` 参数。