---
layout: post # needs to be post
title: 浅析Android中Activity的四种启动模式 # title of your post
summary: standard,singleTop,singleTask,singleInstance.
featured-img: sleek #optional - if you want you can include hero image
categories: [Android]
---

standard,singleTop,singleTask,singleInstance.

<!--more-->

启动模式一共有上述的四种，可以在AndroidManifest.xml文件中通过给标签指定android:launchMode属性来选择启动模式。

## standard
standard是Activity默认的启动模式，在不进行显示指定的情况下，所有Activity都会自动的使用这种模式。Android使用返回栈来管理Activity,在standard模式下，每当启动一个新的Activity,它就会在返回栈中入栈，并处于栈顶的位置。对于使用standard模式的Activity,系统不会在乎这个Activity是否已经在返回栈中，每次启动都会创建一个新的Activity实例。

现在通过实践来体会一下standard模式，首先是新建一个ActivityTest项目，修改MainActivity的onCreate()中代码，如下所示：

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    Log.d("Mainactivity",this.toString());
    
    Button btn1 = (Button) findViewById(R.id.btn1);
    
    btn1.setOnClickListener(v -> {
    	Intent intent = new Intent(MainActivity.this,MainActivity.class);
        startActivity(intent);
    });
}
```

代码很简单，在MainActivity的基础上启动MainActivity，从逻辑上讲没什么意义，不过我们在于研究standard模式，因此我们也就不去追究实际的用途了。另外我们还在onCreate()中添加了一句打印日志信息，打印当前Activity的实例。

现在运行程序，然后再MainActivity界面连续点击两次按钮，观察LogCat中打印的日志信息。

```bash
MainActivity: com.marktony.applicationtest.MainActivity@5e658bb
MainActivity: com.marktony.applicationtest.MainActivity@cfae411
MainActivity: com.marktony.applicationtest.MainActivity@b0e0bac
```

从打印信息中可以很清楚的看到，每次点击按钮就会创建一个新的MainActivity的实例，此时返回栈中也会存在三个MainActivity的实例，因此想要完全退出应用，必须点击三次返回键。

## singleTop
在某些情况下，你可能会觉得standard模式不太合理。activity明明已经在栈顶了，为什么再次启动时还要创建一个新的Activity实例呢？这时singleTop模式就派上用场了。当Activity的启动模式被指定为singleTop时，在启动Activity时如果发现返回栈的栈顶已经是该Activity，则可以认为直接使用它，不创建新的实例。

下面看实例,修改AndroidManifest.xml中的MainActivity的启动模式为singleTop:

```xml
<activity android:name=".MainActivity"
    android:launchMode="singleTop">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

重新运行程序，通过LogCat查看日志信息，我们可以看到已经创建了一个新的Activity实例。

```bash
MainActivity: com.marktony.applicationtest.MainActivity@5e658bb
```

之后不管点击多少次按钮，都不会有新的打印信息出现，因为目前Activity已经处于返回栈的栈顶，每当想要再启动一个MainActivity时都会直接使用栈顶的Activity，因此MainActivity也只会有一个实例，仅按一次返回键即可退出程序。

不过当MainActivity不在栈顶时，这是再启动MainActivity，还是会创建新的实例，下面来实验一下。修改MainActivity中的代码，如下所示：

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    Log.d("MainActivity",this.toString());
    
    Button btn1 = (Button) findViewById(R.id.btn1);
    
    btn1.setOnClickListener(v -> {
    	Intent intent = new Intent(MainActivity.this,SecondActivity.class);
        startActivity(intent);
    });
}
```

这次我们点击按钮启动的是SecondActivity。然后修改SecondActivity中onCreate()方法中的代码，如下所示：

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_second);
    
    Button btn2 = (Button) findViewById(R.id.btn2);
    
    btn2.setOnClickListener(v -> {
    	Intent intent = new Intent(SecondActivity.this,MainActivity.class);
        startActivity(intent);
    });
}
```

我们在SecondActivity中的按钮的点击事件中又加入了启动MainActivity的代码。现在重新运行程序，在MainActivity界面点击按钮进入SecondActivity，然后在SecondActivity界面点击按钮，又会重新进入到MainActivity。查看LogCat中的信息：

```bash
MainActivity: com.marktony.applicationtest.MainActivity@eb60bcf
SecondActivity: com.marktony.applicationtest.SecondActivity@5cf7342
MainActivity: com.marktony.applicationtest.MainActivity@a360333
```

可以看到系统创建了两个不同的MainActivity,这是由于在SecondActivity中再次启动Activity时，栈顶的Activity已经变成了SecondActivity，因此会新建一个新的MainActivity。现在按下back键就会返回到SecondActivity,再次按下back键，又会回到MainActivity,再按一次back键才会退出程序。

## singleTask
使用singleTask模式可以很好的解决重复创建栈顶Activity的问题，但是正如你看到的，如果该Activity并没有处于栈顶的位置，还是可能创建多个Activity实例的。那么有没有方法可以让Activity在整个应用程序的上下文中指存在一个实例呢？

这就要借助singleTask模式来实现了。当Activity的启动模式指定为singleTask，每次启动该Activity时系统首先会在返回栈中检查是否存在该Activity的实例，如果发现已经存在则直接使用该实例，并把在这个Activity之上的所有Activity统统出栈，如果没有发现就会创建一个新的实例。

下面通过代码来直观的理解一下。修改AndroidManifest.xml文件中MainActivity的启动模式：

```xml
<activity android:name=".MainActivity"
    android:launchMode="singleTask">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

然后在MainActivity中添加onRestart()方法，并打印日志：

```java
protected void onRestart() {
    super.onRestart();
    Log.d("MainActivity","onStart");
}
```

最后在SecondActivity中添加onDestroy()方法，并打印日志：

```java
protected void onDestroy() {
    super.onDestroy();
    Log.d("SecondActivity","onDestroy");
}
```

现在重新运行程序，在MainActivity界面点击按钮进入SecondActivity，然后在SecondActivity界面点按钮，又会重新进入到MainActivity。查看LogCat中的日志信息：

```bash
MainActivity: com.marktony.applicationtest.MainActivity@f2c42cf
SecondActivity: com.marktony.applicationtest.SecondActivity@81ea653
MainActivity: onStart
SecondActivity: onDestroy
```

从打印信息中可以看出，在SecondActivity中启动MainActivity时，会发现返回栈里已经存在一个MainActivity的实例，并且是在SecondActivity的下面，于是SecondActivity会从返回栈中出栈，而MainActivity重新成为了栈顶Activity，因此MainActivity的onRestart()方法和SecondActivity的onDestroy()方法会得到执行。现在返回栈中只剩下一个MainActivity的实例了，按一下back键就可以退出程序。

## singleInstance
singleInstance应该是四种启动模式中最特殊也最复杂的一种了。不同于以上三种启动模式，指定为singleInstance的Activity会启用一个新的返回栈来管理这个Activity(其实如果singleTask模式指定了不同的taskAffinity，也会启动一个新的返回栈)。那么这么做有什么意义呢？假设我们的程序中有一个Activity是允许其他程序调用的，如果我们想实现其他程序和我们的程序可以共享这个Activity的实例，应该如何实现呢？使用前面三种启动模式肯定是无法实现的，因为每个程序都会有自己的返回栈，同一个Activity在不同的返回栈中入栈时必然是创建新的实例。而使用singleInstance模式就可以解决这个问题，在这种模式下会有一个单独的返回栈来管理这个Activity，不管是那个应用程序来访问这个Activity，都共用的同一个返回栈，也就解决了共享Activity实例的问题。

OK,talk is cheap,show me the code,我们还是来实践一下。修改AndroidManifest.xml中SecondActivity的启动模式:

```xml
<activity android:name=".SecondActivity"
    android:launchMode="singleInstance">
    <intent-filter>
        <action android:name="com.marktony.applicatiointest.ACTION_START" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="com.marktony.applicationtest.MY_CATEGORY" />
    </intent-filter>
</activity>
```

我们先将SecondActivity的启动模式指定为singleInstance，然后修改MainActivity中onCreate()方法的代码:

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    Log.d("MainActivity","task id is " + getTaskId());
    
    Button btn1 = (Button) findViewById(R.id.btn1);
    
    btn1.setOnClickListener(v -> {
        Intent intent = new Intent(MainActivity.this,SecondActivity.class);
        startActivity(intent);
    });
}
```

在onCreate()方法中打印了当前返回栈的id。然后修改SecondActivity中onCreate()方法的代码：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_second);
    
    Log.d("SecondActivity","task id is " + getTaskId());
    
    Button btn2 = (Button) findViewById(R.id.btn2);
    
    btn2.setOnClickListener(v -> {
        Intent intent = new Intent(SecondActivity.this,ThirdActivity.class);
        startActivity(intent);
    });
}
```

同样在onCreate()方法中打印了当前返回栈的id，然后又修改了按钮点击事件的代码，用于启动ThirdActivity。最后修改ThirdActivity中onCreate()方法的代码：

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_third);
    
    Log.d("ThirdActivity","task id is " + getTaskId());
}
```

仍然是在onCreate()方法中打印了当前返回栈的id。现在重新运行程序，在MainActivity界面中点击按钮进入到SecondActivity，然后在SecondActivity中点击按钮进入到ThirdActivity。查看LogCat中的打印信息：

```bash
MainActivity: task id is 403
SecondActivity: task id is 404
ThirdActivity: task id is 403
```

可以看到，SecondActivity的task id不同于MainActivity和ThirdActivity，这说明SecondActivity确实是存放在一个单独的返回栈中的，而且这个栈中只有SecondActivity这一个Activity。

然后我们按下back键进行返回，你会发现ThirdActivity竟然返回了MainActivity，再按下back键又会返回到SecondActivity，再按下返回键会退出程序，这是为什么呢？

其实原理很简单，由于MainActivity和ThirdActivity是存放在同一个返回栈里的，当在ThirdActivity的界面按下返回键，ThirdActivity会从返回栈中出栈，那么MainActivity就成为了栈顶Activity显示在界面上，因此也就出现了从ThirdActivity直接返回到MainActivity的情况。然后在MainActivity再次按下返回键，这是当前的返回栈已经空了，于是就显示了另一个返回栈的栈顶Activity，及SecondActivity。最后再次按下back键，这是所有的返回栈均被清空了，也就自然的退出了程序。