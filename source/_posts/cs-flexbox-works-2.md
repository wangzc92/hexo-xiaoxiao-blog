title: 通过动图形象地为你介绍 Flexbox 是如何工作的（二）
author: 'Scott Domes '
tags:
  - css
  - flexbox
categories:
  - 前端笔记
  - CSS
date: 2017-03-11 23:18:00
---
![](https://huzidaha.github.io/images-store/201702/5-1PghNPeo_XAXroKXksXGfCQ.png)

在[上一篇文章](http://huzidaha.com/posts/detail?postId=58aaadb2fc5b7f63e8c23f69)中，我们介绍了 flexbox 的几个属性： `flex-direction`，`justify-content`，`align-items` 和 `align-self`。

这些命令在创建基本布局上是特别有用的。而一旦你开始用 flexbox 创建网站的时候，你需要对其进行深挖以最大化地发挥它的价值。

现在我们来深入学习一下 flexbox，并且告诉你如何利用它来构建一个漂亮的布局。

## 属性1：flex-basis
[上一篇文章](http://huzidaha.com/posts/detail?postId=58aaadb2fc5b7f63e8c23f69)中，主要介绍了应用在容器元素上的属性。这次我们专门介绍关于子元素的使用。

以我之见，我们要介绍的第一个属性，是 flexbox 教程里面解释的最少的属性之一。

但是不要担心，其实它很简单。

`flex-basis` 控制的是一个元素的默认尺寸，但是它可以被 *flexbox 的其他属性*所影响（稍后会详细介绍）。

下面这个 GIF 表示的是它和 `width` 这个属性是可以互换的。

![](https://huzidaha.github.io/images-store/201702/5-2S3LKFr_BICUtAWA5LOFxVw.gif)

然而，`flex-basis` 和 `width` 唯一不同的地方是，它是和 flex 坐标轴保持一致的。

![](https://huzidaha.github.io/images-store/201702/5-3Ruy6jFG7gUpSf76IUcJTQ.png)

`flex-basis` 是在主轴方向上影响元素大小的。

我们来看一下，当我们保持 `flex-basis` 不变的情况下，改变主轴方向，会发生什么。

![](https://huzidaha.github.io/images-store/201702/5-4W4QU1Fw9kDLEH2m-J9VGyw.gif)

这里要注意，原来设置的是 `height` 属性，现在必须要手动地设置为设置 `width` 属性。因此可以看到，`flex-basis` 依赖于 `flex-direction` 来影响 `width` 或者 `height`。

## 属性2：flex grow
让我们继续学习，增加点复杂度。

首先，我们把所有的形状 `width` 都设置为120px：

![](https://huzidaha.github.io/images-store/201702/5-5dON3-0RooiPyfDr0DBEOmA.png)

现在我们来引出一个属性 `flex-grow`，它的默认值是 0。也就是说所有的形状没有被允许自动地填充整个容器。

这又是什么意思呢？让我们来看看，如果把各个形状的 `flex-grow` 设置成 1 会发生什么：

![](https://huzidaha.github.io/images-store/201702/5-6cK-yB4z_L6bmEqoG5qDoRA.png)

所有形状一起充满了整个容器的宽度，并且他们之间的间隙也都是相同的。也就是说 *`flex-grow` 覆盖了 `width`*。

那么令人疑惑的问题又来了，它的值具体表达什么意思呢？`flex-grow: 1` 意味着什么呢？

为了解释这个问题，我们把每个形状的 `flex-grow` 值设置成 999，看一下效果：

![](https://huzidaha.github.io/images-store/201702/5-7p2fLcy13xFU9GjtM4cbHEw.png)

可以看到，并没有变化。

这是因为：`flex-grow` 并不是一个绝对值，而是一个相对值。

对于每个元素来说，重要的不是 `flex-grow` 的值是多大，而是*本元素的这个值和其他元素的这个值相比较，相对大小是怎么样的*。

如果我们设置每个元素的 `flex-grow: 1`，而改变第三个形状的 `flex-grow`，我们可以看到下图的改变：

![](https://huzidaha.github.io/images-store/201702/5-8gHyLHG52cySgLmy0x-edZA.gif)

为了完全理解这个知识点，我们来快速地做一些简单的数学运算。

每个元素 `flex-grow` 的起始值都是 1。把所有元素的该值加起来，总和是 6。因此容器的总宽度被分成了 6 份。**每个形状就被扩展到容器所有可用空间的 1/6**。

然后设置第三个形状的 `flex-grow` 值为 2。那么容器的宽度被分成 7 等份，因为所有 `flex-grow` 属性是：1 + 1 + 2 + 1 + 1 + 1。

第三个形状占了整个容器空间的 2/7，其他的占了 1/7。

同理，当设置第三个形状的 `flex-grow: 3` 的时候，整个容器宽度被分成了 8 份（1 + 1 + 3 + 1 + 1 + 1），第三个形状占了 3/8,其他的占了 1/8。

以此类推。

**`flex-grow` 只和比例相关**，例如，设置第三个形状 `flex-grow: 12`，其余每个形状的 `flex-grow: 4`，可以和第三个设置成 3，其他的设置成 1得到同样的效果，见下图：

![](https://huzidaha.github.io/images-store/201702/5-9JnjR4ULs8de0so1bdUPogw.png)

重点在于，每个形状的 `flex-grow` 和其他形状的是*成比例的*。

最后要提醒的是，要记住 `flex-grow` 和 `flex-basis` 类似，它也是应用在主轴上的。形状都会做宽度的改变，除非我们设置 `flex-direction` 为列。

## 属性3：flex shrink
`flex-shrink` 刚好和 `flex-grow` 相反，它是决定形状收缩多少的。

它只应用于元素必须要缩小以适应容器的情况，即容器太小了。

他的主要用法是指定哪个元素你想要缩小，哪个不想缩小。默认情况是每个形状都 `flex-shrink: 1`，这表示每个形状都会随着容器的缩小而缩小。

我们来在应用中看一下，在下面的 GIF 图中，每个形状的 `flex-grow` 都是 1，所以他们填满了整个容器。每个形状的 `flex-shrink` 也都是 1，所以它们也会像下面图中这样收缩。

![](https://huzidaha.github.io/images-store/201702/5-10FVO9kX3wwqakhcT9JWS2Ww.gif)

那么现在，我们来设置第三个形状的 `flex-shrink` 值为 0。不允许它收缩，所以当它拉伸的时候，会随着容器拉伸，而当收缩的时候，不允许比他的 `width` 还小，即不允许比 120px 还小。

![](https://huzidaha.github.io/images-store/201702/5-11GrLzJ4jH3v2Z5Va_TMOXkQ.gif)

默认值是 1，表示元素默认允许收缩，除非你指定它为不允许。

同样，`flex-shrink` 是和比例相关的。如果设置一个形状的 `flex-shrink` 为 6,而其他的是 2，那么这个形状随着容器空间的压缩，将以 3 倍于其他形状的速度缩小。

这里尤其注意：是空间收缩的速度是 3 倍，**而不是说他的宽度会缩小到原来的 1/3**。

接下来我们要深入了解元素到底收缩或者拉伸了多少，不过首先，我们先回到上一个属性，把所有的知识串起来。

## 属性4：flex
`flex` 是 `grow`，`shrink` 和 `basis` 的简化形式——把他们所有都放到了一起。

它的默认值是：0（grow），1（shrink）和 auto（basis）。

在我们的上一个例子中，我们用它来简化两个形状，下面是它们的属性：
```css
	.square#one {
	  flex: 2 1 300px;
	}
	.square#two {
	  flex: 1 2 300px;
	}
```
两个有着相同的 `flex-basis`。也就是说如果有足够的空间（容器的空间等于 600px 加上边缘和内边距），他们每个的宽度将是 300px。

但是随着容器的拉伸，形状 1 （有更大的 `flex-grow` 值）将会以两倍的速度增长。随着容器的收缩，形状 2 （有更大的 `flex-shrink` 值）将会以两倍的速度压缩。

都放到一起展示，如下图：

![](https://huzidaha.github.io/images-store/201702/5-12BKZt7AT5eFee4KRhe82gew.gif)

## 元素是怎样收缩和拉伸的呢
这里有个问题可能会使人疑惑：当形状 1 拉伸时，并没有拉伸到形状 2 的两倍大小。同样，当形状 2 缩小时，也并没有缩小到形状 1 的一半，尽管 `flex-shrink` 的比值是 2:1。

实际上它的意思，并不是说他们的大小是 2:1 或者 1:2，而是说*它们的收缩率或拉伸率*。

## 简单计算
容器的初始大小是 640px。在除去容器要预留的 20px 的间距后，剩下的空间足够将两个形状恢复到 `flex-basis` 等于 300px。

当容器设置到了 430px 时，空间减小了 210px。形状 1，设置了 `flex-shrink` 是 1，减小了 70px。而形状 2，设置了 `flex-shrink` 是 2，减小了 140px。

当容器减小到 340px，容器空间减小了 300px。这是形状 1 减小 100px，而形状 2 减小 200px。

整体减小空间的分配方式，是按照各自设置的 `flex-shrink` 比例分配的（2:1）。

对于 `flex-grow` 是同样的计算方式。当容器拉伸到 940px 时，整体增加了 300px，形状 1 增加 200px，而形状 2 增加 100px。

**当涉及到 flex 属性时，比例是主要考虑的对象**。

![](https://huzidaha.github.io/images-store/201702/5-1370-KTWYpA2LnLjqi0xDrJA.gif)

从上图中，可以看到宽度是如何根据设置的比率变化的，其中的 delta (∆) 表示和原始 `flex-basis` 相比的变化量。

## 结论
最后重述：`flex-basis` 指的是一个元素在发生伸缩之前，沿着主轴方向的大小。`flex-grow` 指的是在元素拉伸时，和兄弟元素相比的拉伸比例。`flex-shrink` 指的是在元素收缩时，和兄弟元素相比的收缩比例。

我们还有几个 flexbox 属性要讲，请留意随后几周的文章。

非常感谢你们，非常感谢每个花时间阅读、评论和分享的读者，你们的鼓励是我不断前行的动力！





