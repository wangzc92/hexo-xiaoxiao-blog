title: 前端解读：彻底理解css3结构伪类选择器
author: wangzc
tags:
  - css3
categories:
  - 前端基础
  - CSS
date: 2017-03-09 13:29:00
---
>css3提供了一整套完备的新选择器。就样式化而言，它允许你匹配更多指定的元素。当你想匹配“列表的最后一项”，或者“总是包含某个东西的第一个段落”，这时你就可以废除那些无意义的id和class了。css选择器有很多，比如属性选择器、后代选择器、否定式选择器、目标伪类选择器、结构伪类选择器等等。今天主要为大家详细讲解css中的结构伪类选择器。

先上全家福:

* :first-child选择与某个元素同级的第一个元素，且是该元素类型；
* :last-child选择与某个元素同级的最后一个元素，且是该元素类型；
* :nth-child()选择与某个元素同级的一个或多个元素，且是该元素类型；
* :nth-last-child()选择与某个元素同级的一个或多个特定的元素，从这个元素的最后一个同级元素开始算；
* :only-child选择的元素是它的父元素的唯一一个子元素；
* :nth-of-type()选择指定的元素；
* :nth-last-of-type()选择指定的元素，从元素的最后一个开始计算；
* :first-of-type选择父级元素的第一个同类子元素；
* :last-of-type选择父级元素的最后一个同类子元素；
* :only-of-type选择一个元素是它的父级元素的唯一一个相同类型的子元素；

是不是快被我绕晕了，好吧，下面对每个进行解析，用图片说话。

## :first-child
:first-child是用来选择与某个元素同级的第一个元素，且是该元素类型，比如这里有个列表，想让列表中的"1"具有不同的样式，我们就可以使用:first-child来实现：
```css
li:first-child{background:yellow;}
```

![first-child](http://ofi3qxlvd.bkt.clouddn.com/image/point/first-child.jpg)

## :last-child
:last-child选择与某个元素同级的最后一个元素，且是该元素类型，与:first-child相对，选择最后一个元素。这里也要同样注意那个问题。例如：
```css
li:last-child{background:yellow;}
```

![last-child](http://ofi3qxlvd.bkt.clouddn.com/image/point/last-child.jpg)

## :nth-child()
这是今天的重点，也是最实用的，好好理解一下。:nth-child()选择与某个元素同级的一个或多个元素，且是该元素类型。参数值是an+b，n是计数器，从0开始。a是倍数，b是偏移量。主要有下面几种形式：
```css
:nth-child(num); /*选择第num个元素*/
:nth-child(n);/*参数是n,n从0开始计算*/
:nth-child(n*num)/*n的倍数选择，n从0开始算*/
:nth-child(n+num);/*选择大于num后面的元素*/
:nth-child(-n+num)/*选择小于num前面的元素*/
:nth-child(n*num+1);/*表示隔几选一*/
//上面num为整数
```

![nth-child](http://ofi3qxlvd.bkt.clouddn.com/image/point/nth-child.jpg)

## :nth-last-child()
:nth-last-child()与:nth-child()用法一样，唯一的区别就是一个从前面数，一个从后面数。这里就不赘述了。

## :only-child
:only-child选择的元素是它的父元素的唯一一个子元素;也就是该元素不能有其他的同级元素。与:only-of-type的区别是，：only-of-type选择的元素可以存在同级元素，但该元素类型只能存在一个。

## :nth-of-type()，:nth-last-of-type()，:first-of-type，:last-of-type和:only-of-type
我之所以这样写是我不想再重复讲述，如果分类的话，前五个可分为一组（简称为child组），后五个分为一组（简称为type组），用法一对一都是一样的，只是有一个重要的不同点。

child组计数时的第几个都是从该元素的第一个同级元素开始，如果第几个与该元素类型一样则生效。而type组计数时的第几个都是从与该元素类型一样的同级元素的第一个开始，不受其他同级元素的干扰。这是重中之重，好好理解一下，可以写几个例子感受一下。

> 结构伪类选择器不兼容IE8及更早版本，所以对浏览器兼容性比较强的小伙伴们还是不要选择这些方法，老老实实定义id或class吧。如果要求不高，这些方法在选取标签添加样式上还是很给力的，快去尝试一下吧。

> 竹风原创，欢迎交流学习。转载请附上本文链接，谢谢！