---
layout: post # needs to be post
title: Android Tools attributes listItem 和 sample data 的用法
summary: Let's learn how to use Android Tools Attributes ListItem and sample data.
featured-img: work
categories: [Android]
---

### tools attributes

你听说过 [layout tools attributes](https://developer.android.com/studio/write/tool-attributes.html) 吗? 

<!--more-->

有时候我们会引入下面的代码, 但是你了解过它具体的作用吗?

```xml
xmlns:tools="http://schemas.android.com/tools"
```

拿最简单的 TextView 举个例子, 我们知道为 TextView 设置 text 的方法是:

```xml
android:text="nb 哄哄的文字信息"
```

我们想要在 XML 中预览数据可以直接通过上面的方法实现, 但是直接在 XML 中写死就有可能出现这样的情况:  从网络或者数据库获取数据有一定的延迟, 用户可能先看到了我们预设的假数据, 然后 TextView 才更新为从数据库或者网络获取到的数据, 这就很尴尬了.

使用 Tools Attributes 就可以避免上面这种尴尬情况的出现.  我们将

```xml
android:text="nb 哄哄的文字信息"
```

替换为

```xml
tools:text="nb 哄哄的文字信息"
```

我们就可以直接在 Android Studio 中预览效果, 但是在打包 APK 时, 这样的预览数据是不会打包进 APK 中, 当然在运行时用户也就不会看到预览数据的.

### list item
对于 RecyclerView, 我们引入了布局之后, 见到的预览效果就是这样的, 嗯, 很干(简)净(陋) .

![recycler view without list item](https://i.loli.net/2018/05/15/5afa771a523dc.png)

我们来给 RecyclerView 加个直观的预览效果:

```xml
tools:listitem="@layout/item_direct_message"
```

![recycler view with list item](https://i.loli.net/2018/05/15/5afa7900a67ad.png)

我们还可以控制预览 tiems 的数量: 

```xml
tools:itemCount="3"
```

遗憾的是, RecyclerView 现在仅支持了以上两个 tools attribute, 而对于`AdapterView` 的子类如 `ListView`, `GridView` 等, 除了支持以上两个 tools attribute, 还支持:

```xml
tools:listheader="@layout/sample_list_header"
tools:listfooter="@layout/sample_list_footer"
```

### sample data

OK, 现在我的确可以预览布局了, 但是还不够, 因为我不想要在资源目录中加上很多的预览资源, 如图片, 文本等, 他们仅仅在预览的时候有用, 打包时我并不需要将它们也打包进 APK. 这个时候就轮到 sample data 开始它的表演了✨ !

从 Android Studio 3.0 开始, 我们就能够使用 Android Studio 提供的预定义数据, 或者自行创建一个 「Sample data」的目录, 然后放入一些假的数据以供预览使用.

#### 预定义数据
Android Studio 3.0 开始提供了一系列的预定义数据, 我们可以在 `tools:text` 属性使用 `@tools/data/`:

```xml
tools:text="@tools:sample/last_names"
```

除了 text 类型的, 预定义数据还包括:

| 属性值     | 占位数据描述 |
| :-------------: | :-------------: |
| @tools:sample/full_names | 随机生成的 @tools:sample/first_names and @tools:sample/last_names 的组合名称 |
| @tools:sample/first_names | 常用的名 |
| @tools:sample/last_names | 常用的姓 |
| @tools:sample/cities | 世界范围内城市的名字 |
| @tools:sample/us_zipcodes | 随机生成的🇺🇸邮政编码 |
| @tools:sample/us_phones | 随机生成的🇺🇸☎️号码, 符合下面的格式: (800) 555-xxxx |
| @tools:sample/lorem | 起源于拉丁文的占位文字 |
| @tools:sample/date/day_of_week | 随机的特定格式的日期和时间 |
| @tools:sample/date/ddmmyy ||
| @tools:sample/date/mmddyy ||
| @tools:sample/date/hhmm ||
| @tools:sample/date/hhmmss ||
| @tools:sample/avatars | 可以用于人物头像的 vector drawables |
| @tools:sample/backgrounds/scenic | 可以用于背景的图片 |

#### 自定义 sample data
在 app (module) 文件夹右键, 选择  new -> Sample Data directory 创建存放假数据的 sample data 文件夹.

![create sample data folder](https://i.loli.net/2018/05/15/5afa7fbb29e56.jpeg)

创建完成后, 我们可以在 app (module) 目录下看到一个名称为 `sampledata` 的文件夹. 在这个文件夹下, 我们创建文本文件如 txt 文件, 添加一些原始数据如 #ff33aa (是的, 可以放置颜色) 或者就是简单的文字, 需要主要每添加一条数据后需要换行, 换言之每行数据占一行.  然后再布局文件中我们就可以通过 `tools:text` 属性引用 `@sample/存放数据的文件的名称` .

```xml
tools:text="@sample/存放数据的文件的名称"
```

除了简单的文本文件外, 还有更高级的用法. 我们可以创建 JSON 文件来展示复杂的数据.

例如我在我的 `sampledata` 目录下创建了一个名为 `github_user.json` 的文件:

```json
{
"github_users": [
{
"name": "Nick Butcher",
"github": "https://github.com/nickbutcher"
},
{
"name": "Chris Banes",
"github": "https://github.com/chrisbanes"
},
{
"name": "Jake Wharton",
"github": "https://github.com/JakeWharton"
},
{
"name": "Romain Guy",
"github": "https://github.com/romainguy"
}
]
}
```

⚠️ 需要注意的是, 这里要求 JSON 文件开头不能是 JsonArray, 只能是 JsonObject. 创建完成后, 需要重新编译一下才能引用到最新的数据.

然后通过下面的方式引用:

```xml
tools:text="@sample/github_user.json/github_users/name[1]"
```

### 其他
除了文章提及的 list item 和 sample data 外, tools attribute 还有很多其他的属性, 我们可以在 Android Developer 查看完整的文档 ->
[Tools Attributes Reference ](https://developer.android.com/studio/write/tool-attributes)

如果文章中有错误之处或建议, 请通过下面的方式联系我:

+ GitHub: https://github.com/TonnyL

+ 微博: https://weibo.com/5313690193

+ 知乎: https://www.zhihu.com/people/lizhaotailang

+ Instagram: https://www.instagram.com/tonny_lztl/

本文参考了 [Android Tools attributes: listItem & sample data rocks!](https://android.jlelse.eu/android-tools-attributes-listitem-sample-data-rocks-bbf49aaa9f07). 

本文由 TonnyL 创作, 发表在 [Tonny’s Blog](https://tonnyl.github.io/) , 转载请遵守 **CC BY-NC-ND 4.0** 协议. 

