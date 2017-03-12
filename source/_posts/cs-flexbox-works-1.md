title: 通过动图形象地为你介绍Flexbox是如何工作的（一）
author: 'Scott Domes '
tags:
  - css
  - flexbox
categories:
  - 前端笔记
  - CSS
date: 2017-03-11 23:02:00
---
![](https://huzidaha.github.io/images-store/201702/3-1.png)

flexbox 承诺将我们从万恶的纯 CSS 中拯救出来（如垂直对齐）。

flexbox 也正在实现它的这一目标，但是用户掌握这一新的模型也将会是个挑战。

因此在这里，我们将会用动图介绍 flexbox 是如何工作的，使得我们可以用它来做更好的布局。

flexbox 的潜在原则是使得布局更加灵活和直观。

为了完成这一目标，它允许容器自己来决定如何均匀地分布其中的元素——包括他们的尺寸和他们之间的间距。

这理论上来讲，听起来很美好。但是让我们来看一下实践中会发生什么。

在这篇文章中，我们会钻研5个通用 flexbox 原则。会探索它们都做了什么？你可以如何使用它们？以及它们的结果是什么样的？

## 属性1：display: flex
![](https://huzidaha.github.io/images-store/201702/3-2.gif)

在一个灰色背景的容器`div`里面，有四个颜色不同、尺寸不同的子`div`，此时每个`div`有默认的`display: block`，每一个的宽度也占满了一整行。

为了使用 flexbox，需要将你的**容器**放在 **flex 容器中** ，见如下代码：
```css
	#container {
	  display: flex;
	}
```
![](https://huzidaha.github.io/images-store/201702/3-3.gif)

可以看到，发生了一点变化。你的四个`div`显示到了一行上，但也就仅此而已。可是你要知道，在这背后，你做了一件很有 power 的事情。**你赋予了你的 div 一个叫做** *flex上下文* **的东西**。

你现在可以把它应用在你的上下文中了，是不是比传统的 CSS 简单很多！

## 属性2：flex-direction
一个 flexbox 容器有两个坐标轴：**主轴**和**交叉轴**，直观的来看如下图：
![](https://huzidaha.github.io/images-store/201702/3-4.png)

**默认情况下，元素都是从左到右地分布在主轴上**。这就是为什么当你应用`display: flex`的时候，形状默认水平分布的原因。

`flex-direction`，可以使你的主轴旋转。
```css
	#container {
	  display: flex;
	  flex-direction: column;
	}
```
![](https://huzidaha.github.io/images-store/201702/3-5.gif)

这里有一个很重要的区别：`flex-direction: column`并不是把你的形状分布在交叉轴上。**而是使主轴自身发生了旋转，从水平方向旋转到了垂直方向**。

还有一些其他的`flex-direction`可选项，如：`row-reverse`和`column-reverse`。

![](https://huzidaha.github.io/images-store/201702/3-6.gif)

## 属性3：justify-content
*justify-content*控制的是你在主轴上如何对齐元素。

这里我们需要对主轴和交叉轴的区别有更深一点的理解。首先让我们回到`flex-firection: row`。
```css
	#container {
	  display: flex;
	  flex-direction: row;
	  justify-content: flex-start;
	}
```
使用 *justify-content*，你有五个选择：

1. flex-start
2. flex-end
3. center
4. space-between
5. space-around

![](https://huzidaha.github.io/images-store/201702/3-7.gif)

`space-around`和`space-between`是最直观的。**`space-between`使每个元素之间有相同的距离，但是不包含元素和容器之间的距离。**

`space-around`让每个元素块的两侧有相同的空隙距离。这就意味着**最外层的元素和容器之间的距离，是两个元素之间距离的一半**（每个元素块的左右两侧都贡献了一个不重叠的等距离，因此是两倍的间隙）。

最后小结：记住**`justify-content`是沿着主轴的，`flex-direction`是转换主轴的**。这对你以后移动元素很关键。

## 属性4：align-items
如果你已经消化了`justify-content`，那么`align-items`对你俩讲将是轻而易举的事了。

`justify-content`是沿着主轴的，而`align-items`是应用到交叉轴上的。

![](https://huzidaha.github.io/images-store/201702/3-8.png)

调整`flex-direction`，使得坐标轴看起来和上面的图一样。

接下来，我们一起看一下`align-items`命令。

1. flex-start
2. flex-end
3. center
4. stretch
5. baseline

前三个和`justify-content`没什么区别，后两个则有一些不同。

`stretch`你的元素将会被拉伸充满整个交叉轴。
`baseline`则会使你的文字底部对齐。见图知意。

![](https://huzidaha.github.io/images-store/201702/3-9.gif)

（注意：如果用`align-items: stretch`，你必须要将元素的`height`设置成`auto`，否则`height`属性将会覆盖`stretch`）

对于`baseline`要意识到，如果你把文字标签拿掉，那么将会用元素的底部对齐来替代原来的效果，如下图。

![](https://huzidaha.github.io/images-store/201702/3-10.png)

为了更好的展示主轴和交叉轴，我们结合`justify-content`和`align-items`来看一下两个`flex-direction`的核心不同。

![](https://huzidaha.github.io/images-store/201702/3-11.gif)

用`row`，元素被分布在水平主轴上。
用`column`，被分布在垂直主轴上。

在这两个 case 中，不论垂直还是水平方向，四个元素都是被居中的，但是这两种情况是绝对不能互相替换的。

## 属性5：align-self
*`align-self`*允许你手动操作一个特定元素的对齐方式。

对于一个元素而言，它基本上是对`align-items`的覆盖。尽管`align-self`默认值设成了`auto`，但是它和`align-items`所有的属性都是一样的，这也使得这个元素继承了容器的`align-items`。
```css	
	#container {
	  align-items: flex-start;
	}
	.square#one {
	  align-self: center;
	}
	// 只有这个形状会居中。
```
我们来看一下它设置的结果是什么样的。对前两个形状设置不同的`align-self`，其他元素设置为`align-items: center`和`flex-direction: row`。

![](https://huzidaha.github.io/images-store/201702/3-12.gif)

## 结论
尽管我们仅仅讲了 flexbox 的皮毛，但是这些命令应该也足够你应付很多基本布局了。

如果你还想看到更多的 GIF flexbox 教程，或者这篇教程对你有所帮助，请在下面给我点赞吧，或者给我留言。

感谢你的阅读！



