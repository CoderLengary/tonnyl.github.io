---
layout: post # needs to be post
title: Android中的ANR现象
summary: 浅谈Android中的ANR现象
featured-img: work
categories: [Android]
---

## 什么是ANR
ANR,Application Not Responding,即程序未响应。发生ANR时，系统会显示一个dialog，提示用户出现了ANR，用户可以选择“等待”，或者是“强制关闭”，结束应用。ANR同崩溃一样，是在开发过程中应该避免的。

![ANR](https://i.loli.net/2018/03/27/5ab9aba480c25.png)

## 什么会触发ANR
+ 主线程被IO操作阻塞。（文档中特别强调:**you should not perform the work on the UI thread(不要在UI线程中进行IO操作)**）。
+ 主线程中存在耗时的操作。
+ 主线程中错误操作。如Thread.wait()或Thread.sleep()等。

在Android中，应用程序的响应是受到Activity Manager 和 Window Manageer系统服务的。一旦出现以下情况，Android系统就会展示dialog：

+ 5秒内没有相应触摸操作。(No response to an input event (such as key press or screen touch events) within 5 seconds.)
+ 广播接收器在10内没有执行完成。(A BroadcastReceiver hasn’t finished executing within 10 seconds.)

## 怎样避免ANR
Android应用通常是完全运行在UI线程(UI Thread)或者说主线程(Main Thread)中。也就是说，你在UI线程中进行的任何长耗时操作都有可能触发ANR。因此，在主线程中的任何操作都应该尽可能快的完成。

+ 在Activity生命周期中的关键方法，例如onCreate()和onResume()中，完成尽可能少的工作。
+ 使用AsyncTask处理耗时操作(网络请求和数据操作等。)
+ 如果使用了Thread或者HandlerThread,确保主线程不会在等待工作线程完成时被阻塞，也就是不会触发Thread.wait()或Thread.sleep()，应该使用Handler处理工作线程的结果。
+ 在BroadcastReceiver中完成那些小而散(small,discrete)的工作，而不是那些长耗时操作或计算的工作，如果需要，应该交给IntentService去完成。

## 技巧提高
通常，100至200ms是用户认为应用运行缓慢的阀值。下面是一些额外的tips，可以使应用避免ANR和及时响应用户：

+ 如果应用正在后台处理用户的输入，那么，处理的进度应该告知用户处理的进度。可以在UI中使用 ProgressBar。
+ 特别的，如果是游戏应用的话，计算工作应该在工作进程(Worker thread)中完成。
+ 如果在初始化中需要进行耗时操作，可以考虑使用欢迎页(splash screen)或者提示用户正在尽快加载，同时显示加载进度信息。这样，用户才不会认为你的应用停滞了(frozen)、
+ 使用Systrace和Traceview来定位应用中响应用户的瓶颈的位置。

文档地址：[Android Developer](https://tonnyl.github.io/2015/05/01/Android-ANR/!http://developer.android.com/training/articles/perf-anr.html)