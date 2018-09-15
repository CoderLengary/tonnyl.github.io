---
layout: post
title: 探索 Android 中的 Span
summary: Spantastic text styling with Spans
featured-img: design
categories: [Android]
---


## 探索 Android 中的 Span
在 Android 中,使用 Span 定义文本的样式. 通过 Span 改变几个文字的颜色，让它们可点击，放缩文字的大小甚至是绘制自定义的项目符号点(bullet points，国外人名中名字之间的间隔符号 · ，HTML 中无序列表项的默认符号)。Span 能够改变 `TextPaint` 属性，在 `Canvas` 上绘制，甚至是改变文本的布局和影响像行高这样的元素。Span 是可以附加到文本或者从本文分离的标记对象(markup objects)；它们可以被应用到部分或整段的文本中。

让我们来看看Span如何使用、提供了哪些开箱即用的功能、怎样简单地创建我们自己的 Span 以及如何使用和测试它们。

[Styling text in Android](https://medium.com/google-developers/spantastic-text-styling-with-spans-17b0c16b4568#b0e9)

[](https://medium.com/google-developers/spantastic-text-styling-with-spans-17b0c16b4568#149b)

+ [Appearance vs metric affecting spans](https://medium.com/google-developers/spantastic-text-styling-with-spans-17b0c16b4568#f6db)
+ [Character vs paragraph affecting spans](https://medium.com/google-developers/spantastic-text-styling-with-spans-17b0c16b4568#1861)

[Creating custom spans](https://medium.com/google-developers/spantastic-text-styling-with-spans-17b0c16b4568#7657)

[Testing custom spans implementation](https://medium.com/google-developers/spantastic-text-styling-with-spans-17b0c16b4568#e345)

[Testing spans usage](https://medium.com/google-developers/spantastic-text-styling-with-spans-17b0c16b4568#eef0)

### 在 Android 上定义文本样式
Android 提供了几种定义文本样式的方法：

+ **单一样式** —— 样式应用在 TextView 显示的整个文本
+ **多重样式** —— 多种样式应用在字符或者段落级别的文本

**单一样式** 使用 XML 属性或者 [样式和主题](https://developer.android.com/guide/topics/ui/look-and-feel/themes.html) 引入了 TextView 的所有内容的样式。这种方式实现简单，通过 XML 即可实现，但是并不能只定义部分内容的样式。举个例子，通过设置 `textStyle=”bold”`，所有的文本都会变为黑体；你不能只定义特定的几个字符为黑体。

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:textSize="32sp"
    android:textStyle="bold"/>
```

**多重样式** 引入了给一段文本添加多种样式的功能。例如，一个单词斜体而另一个粗体。多重样式可以通过使用 HTML 标签、 Span 或者是在 Canvas 上处理自定义的文本绘制。

![左图：单一样式文本。设置了 textSize=”32sp” 和 textStyle=”bold” 的 TextView 。右图：多重样式文本。设置了 ForegroundColorSpan, StyleSpan(ITALIC), ScaleXSpan(1.5f), StrikethroughSpan 的文本。](https://ws2.sinaimg.cn/large/006tNbRwgy1fu7awud22nj318g118wfs.jpg)

左图：单一样式文本。设置了 textSize=”32sp” 和 textStyle=”bold” 的 TextView 。右图：多重样式文本。设置了 ForegroundColorSpan, StyleSpan(ITALIC), ScaleXSpan(1.5f), StrikethroughSpan 的文本。

**HTML 标签**是解决简单问题的简单办法，例如使文本加粗、斜体，甚至是显示项目符号点。为了展示含有 HTML 标签的文本，使用 [`Html.fromHtml`](https://developer.android.com/reference/android/text/Html.html#fromHtml%28java.lang.String,%20int%29) 方法。在内部实现时，HTML 标签被转换成了 span 。但是请注意，`Html` 类并不支持完整的 HTML 标签和 CSS 样式，例如将小黑点改为其他的颜色。

```kotlin
val text = "My text <ul><li>bullet one</li><li>bullet two</li></ul>"
myTextView.text = Html.fromHtml(text)
```

当你有文本样式的需求，但是 Android 平台默认不支持时，你还可以手动地在 Canvas 上绘制文本，例如让文字弯曲排布。

Span 允许你实现具有更细粒度自定义的多重样式文本。举个例子，通过 [BulletSpan](https://developer.android.com/reference/android/text/style/BulletSpan.html)，你可以定义你的段落文本拥有项目符号点。你可以定制文本和点号之间的间距和点号的颜色。从 Android P 开始，你甚至可以 [设置点号的半径](https://developer.android.com/reference/android/text/style/BulletSpan.html#BulletSpan%28int,%20int,%20int%29) 。你也可以创建 span 的自定义实现。在文章中查看 “创建自定义 span” 部分可以找到如何实现。

```kotlin
val spannable = SpannableString("My text \nbullet one\nbullet two")
spannable.setSpan(
    BulletPointSpan(gapWidthPx, accentColor),
    /* 起始索引 */ 9, /* 终止索引 */ 18,
    Spannable.SPAN_EXCLUSIVE_EXCLUSIVE)
spannable.setSpan(
     BulletPointSpan(gapWidthPx, accentColor),
     /* 起始索引 */ 20, /* 终止索引 */ spannable.length,
     Spannable.SPAN_EXCLUSIVE_EXCLUSIVE)
myTextView.text = spannable
```

![左图：使用 HTML 标签；中图：使用 BulletSpan，默认圆点大小；右图：在 Android P 上使用 BulletSpan 或者自定义实现。](https://ws3.sinaimg.cn/large/006tNbRwgy1fu7bp9k0bjj318g0pqwg1.jpg)

左图：使用 HTML 标签；中图：使用 BulletSpan，默认圆点大小；右图：在 Android P 上使用 BulletSpan 或者自定义实现。

你可以组合使用单一样式和多重样式。你可以考虑将设置给 TextView 的样式作为一种“基本”样式，而 span 文本样式是应用在基本样式“之上”并且会覆盖基本样式的样式。例如，当给一个 TextView 设置了 `textColor=”@color.blue”` 属性且给头4个字符应用了 `ForegroundColorSpan(Color.PINK)`，则头4个字符会使用 span 设置的粉色，而其他文本使用 TextView 属性设置的颜色。

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:textColor="@color/blue"/>
```

```kotlin
val spannable = SpannableString(“Text styling”)
spannable.setSpan(
     ForegroundColorSpan(Color.PINK), 
     0, 4, 
     Spannable.SPAN_EXCLUSIVE_EXCLUSIVE)
myTextView.text = spannable
```

![TextView 组合使用 XML 属性和 span 样式](https://ws3.sinaimg.cn/large/006tNbRwgy1fu7c1hlja1j30g30om74h.jpg)

TextView 组合使用 XML 属性和 span 样式

### 应用 span
当使用 span 时，你会和以下类的其中之一打交道：[SpannedString](https://developer.android.com/reference/android/text/SpannedString.html), [SpannableString](https://developer.android.com/reference/android/text/SpannableString.html) 或 [SpannableStringBuilder](https://developer.android.com/reference/android/text/SpannableStringBuilder.html)。 它们之间的区别在于文本或标记对象是可改变的还是不可改变的以及它们使用的内部结构：`SpannedString` 和 `SpannableString` 使用线性数组记录已添加的 span，而 `SpannableStringBuilder` 使用 [区间树](https://en.wikipedia.org/wiki/Interval_tree)。 

下面是如何决定使用哪一个的方法：

+ 仅 **读取而不设置** 文本和 span？ --> `SpannedString`
+ **设置文本和 span**？ --> `SpannableStringBuilder`
+ 设置 **少量的 span** (<~ 10)？ --> `SpannableString`
+ 设置 **大量的 span** (>~ 10)？ -->  `SpannableStringBuilder`

举个例子，你用到的文本并不会改变，但你想要附加 span 时，你应该使用 `SpannableString`。

```

║     类                 ║    可变文本   ║     可变标记    ║
═════════════════════════════════════════════════════════
║ SpannedString          ║      否      ║       否       ║
║ SpannableString        ║      否      ║       是       ║
║ SpannableStringBuilder ║      是      ║       是       ║
```

上面所有的这些类都继承自 [Spanned](https://developer.android.com/reference/android/text/Spanned.html) 接口，但拥有可变标记的类( `SpannableString` 和 `SpannableStringBuilder`) 同时也继承自 [Spannable](https://developer.android.com/reference/android/text/Spannable.html)。

[Spanned](https://developer.android.com/reference/android/text/Spanned.html) --> 带有不可变标记的不可变文本 

[Spannable](https://developer.android.com/reference/android/text/Spannable.html) (继承自 Spanned) --> 带有可变标记的不可变文本

通过 `Spannable` 对象调用 `setSpan(Object what, int start, int end, int flags)` 方法应用 span。 `What` 对象是一个标记，应用于从开始到结束的索引之间的文本。falg 标志位标记了 span 是否应该扩展至包含插入文本的开始和结束的点。任何标志位设置以后，只要插入文本的位置位于开始位置和结束位置之间，span 就会自动的扩展。

举个例子，设置 `ForegroundColorSpan` 可以像下面这样完成：

```kotlin
val spannable = SpannableStringBuilder(“Text is spantastic!”)
spannable.setSpan(
     ForegroundColorSpan(Color.RED), 
     8, 12, 
     Spannable.SPAN_EXCLUSIVE_INCLUSIVE)
```

因为 span 设置时使用了 [`SPAN_EXCLUSIVE_INCLUSIVE`](https://developer.android.com/reference/android/text/Spanned.html#SPAN_EXCLUSIVE_EXCLUSIVE) 标志位，在 span 的后面插入文本时，新插入的文本也会自动地继承此 span。

```kotlin
val spannable = SpannableStringBuilder(“Text is spantastic!”)
spannable.setSpan(
     ForegroundColorSpan(Color.RED), 
     /* start index */ 8, /* end index */ 12, 
     Spannable.SPAN_EXCLUSIVE_INCLUSIVE)
spannable.insert(12, “(& fon)”)
```

![左图：带有 ForegroundColorSpan 的文本；右图：带有 ForegroundColorSpan 和 Spannable.SPAN_EXCLUSIVE_INCLUSIVE 的文本。](https://ws4.sinaimg.cn/large/006tNbRwgy1fu8jgf3u6xj318g118myj.jpg)

左图：带有 ForegroundColorSpan 的文本；右图：带有 ForegroundColorSpan 和 Spannable.SPAN_EXCLUSIVE_INCLUSIVE 的文本。

如果 span 设置了 `Spannable.SPAN_EXCLUSIVE_EXCLUSIVE` 标志位，在 span 后面插入文本时则不会修改 span 的结束索引。

多个 span 可以被组合且同时附加到同一段文本上。例如，粗体红色的文本可以像这样构建：

```kotlin
val spannable = SpannableString(“Text is spantastic!”)
spannable.setSpan(
     ForegroundColorSpan(Color.RED), 
     8, 12, 
     Spannable.SPAN_EXCLUSIVE_EXCLUSIVE)
spannable.setSpan(
     StyleSpan(BOLD), 
     8, spannable.length, 
     Spannable.SPAN_EXCLUSIVE_EXCLUSIVE)
```

![带有多种 span 的文本：ForegroundColorSpan(Color.RED) 和 StyleSpan(BOLD)](https://ws1.sinaimg.cn/large/006tNbRwgy1fu8jgbbbhwj30g30omglv.jpg)

带有多种 span 的文本：ForegroundColorSpan(Color.RED) 和 StyleSpan(BOLD)

### Framework span
Android framework 定义了几个接口和抽象类，它们会在测量和渲染时被检查。这些类有允许 span 访问像 `TextPaint` 或 `Canvas` 对象的方法。

Android framework 在 [`android.text.style`](https://developer.android.com/reference/android/text/style/package-summary.html) 包、主要接口的字类和抽象类中提供了20+的 span， 我们可以通过下面几个方式对 span 进行分类：

+ 基于 span 是否只改变外形或者 text 的大小或布局
+ 基于 是否会在字符或段落级别影响文本

![span 类别：字符对比段落，外形对比大小](https://ws4.sinaimg.cn/large/006tNbRwgy1fu8jgfkllyj30wr0ac0ti.jpg)

span 类别：字符对比段落，外形对比大小

### 影响外形对比影响尺寸的 span
第一种类型以修改外形的方式在字符级别起作用：文本或背景颜色、下划线、中横线等等，它会触发文本重新绘制但是并不会重新布局。这些 span 引入了 [`UpdateAppearance`](https://developer.android.com/reference/android/text/style/UpdateAppearance.html) 且继承自 [`CharacterStyle`](https://developer.android.com/reference/android/text/style/CharacterStyle.html). `CharacterStyle` 字类通过提供更新 `TextPaint` 的访问方法，定义了怎样绘制文本。

![影响外形的 span](https://ws4.sinaimg.cn/large/006tNbRwgy1fu8jjas6rzj31700jlt9j.jpg)

影响外形的 span

**影响尺寸**的 span 更改了文本的尺寸和布局，因此观察 span 的变化的对象会重新绘制文本以保证布局和渲染的正确。

举个例子，影响文本字体大小的 span 要求重新测量和布局，也要求重新绘制。这种 span 通常继承自 [`MetricAffectingSpan`](https://developer.android.com/reference/android/text/style/MetricAffectingSpan.html) 类。这个抽象类通过提供对 `TextPaint` 的访问，允许字类定义 span 如何影响文本测量，而 `MetricAffectingSpan` 继承自 `CharacterSpan`，子类在字符级别影响文本的外形。

![影响尺寸的 span](https://ws1.sinaimg.cn/large/006tNbRwgy1fu8jseg55qj318g0kemyv.jpg)

影响尺寸的 span

你可能会想要一直重新创建带有文本和标记的 `CharSequence` 并且调用 [`TextView.setText(CharSequence)`](https://developer.android.com/reference/android/widget/TextView.html#setText%28java.lang.CharSequence%29) 方法，但是这样做很有可能一直触发已经创建好的布局和额外的对象的重新测量和重新绘制。为了减少性能损耗，将文本设置为 [`TextView.setText(Spannable, BufferType.SPANNABLE)`](https://developer.android.com/reference/android/widget/TextView.html#setText%28int,%20android.widget.TextView.BufferType%29)，然后当你需要更改 span 的时候，通过将 `TextView.getText()` 转换为 `Spannable` 从 TextView 获得 `Spannable` 对象。我们会在未来的文章中详细讨论 `TextView.setText` 的实现原理和不同的性能优化方式。

举个例子，考虑通过这样的方式设置和获取 `Spannable`：

```kotlin
val spannableString = SpannableString(“Spantastic text”)
// 将文本设置为一个 Spannable
textView.setText(spannableString, BufferType.SPANNABLE)
// 然后获取 TextView 持有的 text 对象引用
// 这里之所以能转换为 Spannable 是因为我们之前将它设置为了 BufferType.SPANNABLE
val spannableText = textView.text as Spannable
```

现在，当我们在 `spannableText` 上设置了 span 之后，我们就不需要再调用 `textView.setText` 了，因为我们正在直接修改 `TextView` 持有的 `CharSequence` 对象的引用。

这是当我们设置了不同的 span 之后会发生什么：

**情形1: 影响外观的 span**

```kotlin
spannableText.setSpan(
     ForegroundColorSpan(colorAccent), 
     0, 4, 
     Spannable.SPAN_EXCLUSIVE_EXCLUSIVE)
```

当我们附加了一个影响外观的 span 之后，`TextView.onDraw` 方法被调用但 `TextView.onLayout` 没有。这是文本重绘，但宽和高保持原样。

**情形2: 影响尺寸的 span**

```kotlin
spannableText.setSpan(
     RelativeSizeSpan(2f), 
     0, 4, 
     Spannable.SPAN_EXCLUSIVE_EXCLUSIVE)
```

因为 [`RelativeSizeSpan`](https://developer.android.com/reference/android/text/style/RelativeSizeSpan.html) 改变了文本的大小，文本的宽和高变化，文本的布局方式（举个例子，在 `TextView` 的大小没有变化的情况下，一个特定的单词现在可能会换行）。`TextView` 需要计算新的大小所以 `onMeasure` 和 `onLayout` 均被调用。

![左图：ForegroundColorSpan——影响外观的 span；右图：RelativeSizeSpan——影响尺寸的 span](https://ws4.sinaimg.cn/large/006tNbRwgy1fueegt50m3j318g118wfw.jpg)
左图：ForegroundColorSpan——影响外观的 span；右图：RelativeSizeSpan——影响尺寸的 span

### 影响字符和影响段落的 span
一个 span 对文本产生的影响既可以在字符级别，更新元素，如背景颜色、样式或大小，也可以在段落级别，更改整个文本块的对齐或者边距。根据所需的样式，span 既可以继承自[`CharacterStyle`](https://developer.android.com/reference/android/text/style/CharacterStyle.html)，也可以引入 [`ParagraphStyle`](https://developer.android.com/reference/android/text/style/ParagraphStyle.html)。 继承自 `ParagraphStyle` 的 span 必须从第一个字符附加到单个段落的最后一个字符，否则 span 不会被显示。在 Android 中，段落是基于换行符 (\n) 定义的。

![在 Android 中，段落是基于换行符 (\n) 定义的。](https://ws4.sinaimg.cn/large/006tNbRwgy1fueeiq8nwhj30am04nq2w.jpg)

在 Android 中，段落是基于换行符 (\n) 定义的。

![影响段落的 span](https://ws3.sinaimg.cn/large/006tNbRwgy1fueejg4dqbj318g0bwab3.jpg)

影响段落的 span

举个例子，像 [`BackgroundColorSpan`](https://developer.android.com/reference/android/text/style/BackgroundColorSpan.html) 这样的 `CharacterStyle`，可以被附加到文本中的任何字符上。这里，我们把它附加到第五到第八个字符上。


```kotlin
val spannable = SpannableString(“Text is\nspantastic”)
spannable.setSpan(
    BackgroundColorSpan(color),
    5, 8,
    Spannable.SPAN_EXCLUSIVE_EXCLUSIVE)
```

`ParagraphStyle` 的 span，像 [`QuoteSpan`](https://developer.android.com/reference/android/text/style/QuoteSpan.html)，只能够被附加到段落的开始，否则行和文本之间的边距就不会出现。例如，“Text is\nspantastic” 在文本的第8个字符包含了一个换行符，所以我们可以给它附加一个 `QuoteSpan`，段落从那里开始就会被添加样式。如果我们在0或8之外的任何位置附加 span，text 就不会被添加样式。

```kotlin
spannable.setSpan(
    QuoteSpan(color), 
    8, text.length, 
    Spannable.SPAN_EXCLUSIVE_EXCLUSIVE)
```

![左图：BackgroundColorSpan -- 影响字符的 span。右图：QuoteSpan -- 影响段落的 span](https://ws1.sinaimg.cn/large/006tNbRwgy1fueek909abj318g118ta0.jpg)

左图：BackgroundColorSpan -- 影响字符的 span。右图：QuoteSpan -- 影响段落的 span 

### 创建自定义 span
在实现你自己的 span 时，你需要确定你的 span 是否会影响字符或段落级别的文本，以及它是否也会影响文本的布局或外观。但是，在从头开始编写自己的实现之前，检查一下是否可以使用 framework 中提供的 span。

太长不看：

+ 在 **字符级别** 影响文本 -> `CharacterStyle`
+ 在 **段落级别** 影响文本 -> `ParagraphStyle`
+ 影响 **文本外观** -> `UpdateAppearance`
+ 影响 **文本尺寸** -> `UpdateLayout`

假设我们需要实现一个 span，它允许以一定的比例增加文本的大小，比如 `RelativeSizeSpan`，并设置文本的颜色，比如 `ForegroundColorSpan`。为此，我们可以扩展 `RelativeSizeSpan`，并且 `RelativeSizeSpan` 提供了 `updateDrawState` 和 `updateMeasureState` 回调，我们可以复写绘制状态回调并设置 `TextPaint` 的颜色。

```kotlin
class RelativeSizeColorSpan(
    @ColorInt private val color: Int,
    size: Float
) : RelativeSizeSpan(size) {
    override fun updateDrawState(textPaint: TextPaint?) {
         super.updateDrawState(ds)
         textPaint?.color = color
    }
}
```

注意：同样的效果可以通过在同一文本上同时应用 `RelativeSizeSpan` 和 `ForegroundColorSpan` 实现。

### 测试自定义 span 的实现
测试 span 意味着检查确实已对 TextPaint 进行了预期的修改，或者是否已经将正确的元素绘制到了 canvas 上。例如，假设一个 span 的自定义实现为段落添加制定大小和颜色的项目符号点，以及左边距和项目符号点之间的间隙。在 [android-text sample](https://github.com/googlesamples/android-text/blob/master/TextStyling-Kotlin/app/src/main/java/com/android/example/text/styling/renderer/spans/BulletPointSpan.kt) 查看具体实现。为了测试这个类，实现一个 AndroidJUnit 类，确实检查：

+ 一个特定大小的圆被绘制在了 canvas 上
+ 如果没有被附加到文本上，则没有任何绘制
+ 基于构造函数的参数，设置了正确的边距

测试 Canvas 的交互可以通过 mock canvas，给 `drawLeadingMargin` 方法传 mock 过的引用并使用正确的参数验证是否已调用正确的方法来实现。

```kotlin
val canvas = mock(Canvas::class.java)
val paint = mock(Paint::class.java)
val text = SpannableString("text")

@Test fun drawLeadingMargin() {
    val x = 10
    val dir = 15
    val top = 5
    val bottom = 7
    val color = Color.RED
    // 给定一个已经设置到文本上的 span
    val span = BulletPointSpan(GAP_WIDTH, color)
    text.setSpan(span, 0, 2, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE)
    // 当前面的边距已经被绘制
    span.drawLeadingMargin(canvas, paint, x, dir, top, 0, bottom,
            text, 0, 0, true, mock(Layout::class.java))
    // 检查确定的 canvas 和 paint 方法在确定的顺序下被调用        
    val inOrder = inOrder(canvas, paint)
    // bullet point paint color is the one we set
    inOrder.verify(paint).color = color
    inOrder.verify(paint).style = eq<Paint.Style>(Paint.Style.FILL)
    // 一个确定大小的圆在确定位置被绘制
    val xCoordinate = GAP_WIDTH.toFloat() + x.toFloat()
    +dir * BulletPointSpan.DEFAULT_BULLET_RADIUS
    val yCoord = (top + bottom) / 2f
    inOrder.verify(canvas)
           .drawCircle(
                eq(xCoordinate),
                eq(yCoord), 
                eq(BulletPointSpan.DEFAULT_BULLET_RADIUS), 
                eq(paint))
    verify(canvas, never()).save()
    verify(canvas, never()).translate(
               eq(xCoordinate), 
               eq(yCoordinate))
}
```

在 [`BulletPointSpanTest`](https://github.com/googlesamples/android-text/blob/master/TextStyling-Kotlin/app/src/androidTest/java/com/android/example/text/styling/renderer/spans/BulletPointSpanTest.kt) 查看其余的测试。

### 测试 span 的用法
[`Spanned`](https://developer.android.com/reference/android/text/Spanned.html) 接口允许给文本设置 span 和从文本获取 span 。通过实现一个 Android JUnit 测试，检查是否在正确的位置添加了正确的 span。在 [`android-text sample`](https://github.com/googlesamples/android-text) 我们把项目符号点标记标签转换为了项目符号点。这是通过给文本在正确的位置附加 `BulletPointSpans`。 下面展示了它是如何被测试的：

```kotlin
@Test fun textWithBulletPoints() {
val result = builder.markdownToSpans(“Points\n* one\n+ two”)
// 检查标记标签被移除
assertEquals(“Points\none\ntwo”, result.toString())
// 获取所有附加到 SpannedString 上的 span
val spans = result.getSpans<Any>(0, result.length, Any::class.java)assertEquals(2, spans.size.toLong())
// 检查 span 确实是 BulletPointSpan
val bulletSpan = spans[0] as BulletPointSpan
// 检查开始和结束索引正是期望值
assertEquals(7, result.getSpanStart(bulletSpan).toLong())
assertEquals(11, result.getSpanEnd(bulletSpan).toLong())
val bulletSpan2 = spans[1] as BulletPointSpan
assertEquals(11, result.getSpanStart(bulletSpan2).toLong())
assertEquals(14, result.getSpanEnd(bulletSpan2).toLong())
}
```

查看 [`MarkdownBuilderTest`](https://github.com/googlesamples/android-text/blob/master/TextStyling-Kotlin/app/src/androidTest/java/com/android/example/text/styling/renderer/MarkdownBuilderTest.kt) 获取更多测试示例。

> 注意，如果你需要在测试之外遍历 span，使用 [`Spanned#nextSpanTransition`](https://developer.android.com/reference/android/text/Spanned.html#nextSpanTransition%28int,%20int,%20java.lang.Class%29) 而不是 [`Spanned#getSpans`](https://developer.android.com/reference/android/text/Spanned.html#getSpans%28int,%20int,%20java.lang.Class%3CT%3E%29)，因为它性能更好。

-------

span 是一个非常强大的概念，它深深的嵌入在文本渲染功能中。它们可以访问 TextPaint 和 Canvas 等组件，这些组件允许在 Android 上使用高度可自定义的文本样式。在 Android P 中，我们为 framework span 添加了大量文档，所以，在实现你自己的 span 之前，查看那些能够获取到的内容。

在以后的文章中，我们将向你详细介绍 span 在底层是如何工作的以及怎样高效地使用它们。例如，你需要使用 [`textView.setText(CharSequence, BufferType)`](https://developer.android.com/reference/android/widget/TextView.html#setText%28java.lang.CharSequence,%20android.widget.TextView.BufferType%29) 或者 [`Spannable.Factory`](https://developer.android.com/reference/android/text/Spannable.Factory.html)。 有关原因的详细信息，请保持关注。

> 非常感谢 [Siyamed Sinir](https://twitter.com/siyamed)，Clara Bayarri 和 [Nick Butcher](https://medium.com/@crafty).

本文原作者 [Florina Muntenescu](https://medium.com/@florina.muntenescu)，Android Developer Advocate @Google . 原文地址：[https://medium.com/google-developers/spantastic-text-styling-with-spans-17b0c16b4568](https://medium.com/google-developers/spantastic-text-styling-with-spans-17b0c16b4568). 本文由 TonnyL 翻译，发表在： [https://tonnyl.github.io/](https://tonnyl.github.io/)