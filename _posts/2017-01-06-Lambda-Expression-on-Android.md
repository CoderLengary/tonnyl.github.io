---
layout: post # needs to be post
title: 极简主义 - Lambda 表达式在 Android 中的应用
summary: Android 支持完整的 Java7 特性，而升级到 Android Nougat(API Level 24) 之后，支持了 Java8 语言的一部分功能
featured-img: raindrop_glass
categories: [Android]
---

## 前言
Android 支持完整的 Java7 特性，而升级到 Android Nougat(API Level 24) 之后，支持了 Java8 语言的一部分功能：

+ [默认和静态接口方法](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html)
+ [Lambda 表达式](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)(在 API 级别 23 及更低版本中也可用)
+ [重复注解](https://docs.oracle.com/javase/tutorial/java/annotations/repeating.html)
+ [方法引用](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)(在 API 级别 23 及更低版本中也可用)
+ [类型注解](https://docs.oracle.com/javase/tutorial/java/annotations/type_annotations.html)(在 API 级别 23 及更低版本中也可用)

## 准备工作
<del>使用这一部分支持的功能需要一个名为Jack的新编译器。</del> (Jack 工具链已经被废弃, 请看这里: https://android-developers.googleblog.com/2017/03/future-of-java-8-language-feature.html ) 而 Jack 要求 Android Studio 2.1 或更高，所以，没有升级到 2.1 或以上的童鞋，需要先升级才能支持使用 Jack 哟。

在 `app` 目录下 `build.gradle` 文件下添加：

```gradle
android {
  ...
  defaultConfig {
    ...
    jackOptions {
      enabled true
    }
    // 在更老的版本中，使用
    // useJack true
    // 可以达到同样的效果
  }
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}
```

## Lambda 表达式
Lambda 的语法格式如下:

```java
(arg1, arg2...) -> { body }
(type1 arg1, type2 arg2...) -> { body }
```

在未使用Lambda表达式之前，我们设置监听器的代码是这样的:

```java
fab.setOnClickListener(new View.OnClickListener() {
    
	@Override
    public void onClick(View v) {
        // blah, blah
    }
});
```

而使用 Lambda 表达式，代码是这样的:

```java
fab.setOnClickListener(v -> {
	// blah, blah            
});
```

是不是更简洁了呢？

我们查看一下 `OnClickListener` 的源码，发现这个接口中只有一个待实现的方法。

```java
/**
 * Interface definition for a callback to be invoked when a view is clicked.
 */
public interface OnClickListener {
    /**
     * Called when a view has been clicked.
     *
     * @param v The view that was clicked.
     */
    void onClick(View v);
}
```

像这种只有一个待实现方法的接口，都可以使用 Lambda 表达式进行简化。当然，我们也可以自定义接口，然后用 Lambda 表达式进行实现。

## 方法引用
有时会遇到这样的情况，Lambda 表达式什么都没有做，仅仅是调用了已经存在的方法。这种情况下，代码就能进一步精简了。

```java
fab.setOnClickListener( view2 -> showSnackBar(view2));
private void showSnackBar(View view) {
        Snackbar.make(view, "blah, blah", Snackbar.LENGTH_SHORT).show();
}
```

## 上下文推断
下面来实现一些例子来了解 Lambda 的上下文推断:

```java
public interface TestOp {
   String operate (String s, int i);
}
// 使用Lambda表达式进行实现
// 通过上下文，lambda表达式可以推断出s为String类型，而i为int类型
// 返回值为String
TestOp myOp = (s,i) -> s + i;
```

使用方法引用之后：

```java
fab.setOnClickListener(this::showSnackBar);
```

## 总结
Lambda 表达式使用还是很方便的，能够减少编写和维护的代码数量。但是，这种极简主义的写法也并不是所有人都能接受的。