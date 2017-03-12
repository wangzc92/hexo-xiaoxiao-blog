title: 无循环 JavaScript
author: James Sinclair
tags:
  - JavaScript
categories:
  - 前端笔记
  - JavaScript
date: 2017-03-11 22:52:00
---
之前有讨论过，[缩进导致了代码复杂性的增加（以粗鲁的方式）](http://jrsinclair.com/articles/2017/indentation-is-the-enemy-less-complex-javascript/)。我们的目标是写出复杂度低的 JavaScript 代码。通过选择一种合适的抽象来解决这个问题，可是你怎么能知道选择哪一种抽象呢？很遗憾的是到目前为止，没有找到一个具体的例子能解释这一问题。这篇文章中我们讨论不用任何循环如何处理 JavaScript 数组，最终得出的效果是得到低复杂度的代码。

![](https://huzidaha.github.io/images-store/201702/6-1javascript-without-loops.png)

*循环是一种很重要的控制结构，它很难被重用，也很难插入到其他操作之中。另外，它意味着随着每次迭代，代码也在不断的变化之中。——Luis Atencio*

## 循环
我们先前说过，像循环这样的控制结构引入了复杂性。但是至今也没能很好的解释这是如何发生的。那么我们首先来看一下在 JavaScript 中循环是如何起作用的。

在 JavaScript 中，至少有四、五种实现循环的方法。最基础的是 `while` 循环。首先，先创建一个示例函数和数组。
```javascript
	// oodlify :: String -> String
	function oodlify(s) {
	    return s.replace(/[aeiou]/g, 'oodle');
	}
	
	const input = [
	    'John',
	    'Paul',
	    'George',
	    'Ringo',
	];
```
现在有了一个数组，我们想要用 *oodlify* 函数处理每一个元素。如果用 `while` 循环，就类似于这样：
```javascript
	let i = 0;
	const len = input.length;
	let output = [];
	while (i < len) {
	    let item = input[i];
	    let newItem = oodlify(item);
	    output.push(newItem);
	    i = i + 1;
	}
```
注意看好每一步，首先用了一个计数器 i，把这个计数器初始化为 0，然后在每次循环中将其自增。每次必须要和 *len* 进行比较以保证它在那里停下来。这种方式太有共性了，所以 JavaScript 提供了一个更简单的实现方式： `for` 循环，写起来如下：
```javascript
	const len = input.length;
	let output = [];
	for (let i = 0; i < len; i = i + 1) {
	    let item = input[i];
	    let newItem = oodlify(item);
	    output.push(newItem);
	}
```
这一结构非常有用，它把所有的计数器引用都放到了最上面，而 `while` 循环非常容易把自增的 i 给忘掉，进而引起无限循环。这确实是一个改进，但是重新思考一下这个问题。我们想要达到的目标是在数组的每个元素上运行 *oodlify()* 函数，并且将结果放到一个新的数组中，我们并不关心计数器的问题。

对一个数组中每个元素都进行操作的这种模式也是非常普遍的。因此，在 ES2015 中，有了一种新的循环结构，这种循环结构可以丢弃掉计数器： `for...of` 循环。每一次返回数组的下一个元素给你，代码如下：
```javascript
	let output = [];
	for (let item of input) {
	    let newItem = oodlify(item);
	    output.push(newItem);
	}
```
这样就清晰很多，注意这里计数器和比较都不用了，甚至都不用将数组里的元素做一步额外的取出操作。`for...of` 帮我们做了里面的脏活累活。到此为止，我们用 `for...of` 来代替 `for` 循环，可以很大程度上降低复杂性。但是，我们还可以进一步优化它。

## mapping
`for...of` 循环比 `for` 循环更清晰，但是依然需要一些设定性的代码。如不得不初始化一个 *output* 数组并且每次循环都要调用 *push()* 函数。但是如何解决这个问题，我们不妨先来扩展一下问题。

如果有两个数组需要调用 *oodlify* 函数会怎么样？
```javascript	
	const fellowship = [
	    'frodo',
	    'sam',
	    'gandalf',
	    'aragorn',
	    'boromir',
	    'legolas',
	    'gimli',
	];
	
	const band = [
	    'John',
	    'Paul',
	    'George',
	    'Ringo',
	];
```
很直观的想法是为每个数组做循环：
```javascript	
	let bandoodle = [];
	for (let item of band) {
	    let newItem = oodlify(item);
	    bandoodle.push(newItem);
	}
	
	let floodleship = [];
	for (let item of fellowship) {
	    let newItem = oodlify(item);
	    floodleship.push(newItem);
	}
```
这确实ok。有能正确执行的代码，就比没有好。但是，这是重复性的工作——不够“[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)”。我们来重构它以降低它的重复性，创建一个函数：
```javascript	
	function oodlifyArray(input) {
	    let output = [];
	    for (let item of input) {
	        let newItem = oodlify(item);
	        output.push(newItem);
	    }
	    return output;
	}
	
	let bandoodle = oodlifyArray(band);
	let floodleship = oodlifyArray(fellowship);
```
这看起来好多了，可是如果我们想使用另外一个函数该怎么办？
```javascript	
	function izzlify(s) {
	    return s.replace(/[aeiou]+/g, 'izzle');
	}
```
上面的 *oodlifyArray()* 将不起作用了。可是如果再创建一个 *izzlifyArray()* 函数的话，那就又变成重复的问题了。先不管那么多，我们先将他们并排写出来：
```javascript	
	function oodlifyArray(input) {
	    let output = [];
	    for (let item of input) {
	        let newItem = oodlify(item);
	        output.push(newItem);
	    }
	    return output;
	}
	
	function izzlifyArray(input) {
	    let output = [];
	    for (let item of input) {
	        let newItem = izzlify(item);
	        output.push(newItem);
	    }
	    return output;
	}
```
这两个函数惊人的相似。那么我们是不是可以把他们抽象成一个通用的模式呢？我们想要的是：*给定一个函数和一个数组，通过这个函数，把数组中的每一个元素做操作后放到新的数组中。*我们把这个模式叫做 *map* 。一个数组的 map 函数如下：
```javascript	
	function map(f, a) {
	    let output = [];
	    for (let item of a) {
	        output.push(f(item));
	    }
	    return output;
	}
```
当然，这里并没有完全脱离循环。如果想要脱离循环的话，可以做一个递归的版本出来：
```javascript	
	function map(f, a) {
	    if (a.length === 0) { return []; }
	    return [f(a[0])].concat(map(f, a.slice(1)));
	}
```
递归解决方法非常优雅，仅仅用了两行代码，并且只有很少的缩进。但是通常我们并不倾向于使用递归，因为它在较老的浏览器中的性能非常差。实际上，我们并不是非得自己写 *map*（除非我们自己想写）。*map* 模式非常有共性，因此 JavaScript 提供了一个内置 *map* 方法。使用这个 *map* 方法，上面的代码变成了这样：
```javascript	
	let bandoodle     = band.map(oodlify);
	let floodleship   = fellowship.map(oodlify);
	let bandizzle     = band.map(izzlify);
	let fellowshizzle = fellowship.map(izzlify);
```
可以注意到，缩进消失，循环消失。诚然，循环可能转移到了其他地方，可是这并不是我们所关心的。我们的代码现在变得简洁而富有表达张力。

为什么这个代码这么简单呢？这可能是个很傻的问题，不过也请思考一下。是因为短吗？不是，短并不代表不复杂。它很简单，是因为我们把问题分离了。有两个处理字符串的函数： *oodlify* 和 *izzlify*，这些函数并不需要知道关于数组或者循环的任何事情。同时，有另外一个函数： *map* ，它来处理数组，它不需要知道数组中元素是什么类型的，甚至你想对数组做什么也不用关心。它只需要执行我们所传递的函数就可以了。我们从对数组的处理中，把对字符串的处理分离出来，而不是把它们都混在一起。这就是为什么我们说上面的代码很简单。

## reducing
现在，*map* 已经得心应手了，但是这并没有覆盖到可能需要的每一种循环。只有当你想创建一个和输入数组同样长度的数组时才有用。但是如果你想要向数组中增加几个元素呢？或者想找一个列表中的最短字符串是哪个？其实有时我们对数组进行处理，最终只想得到一个值而已。

来看一个例子，现在有一个关于英雄的数组：
```javascript	
	const heroes = [
	    {name: 'Hulk', strength: 90000},
	    {name: 'Spider-Man', strength: 25000},
	    {name: 'Hawk Eye', strength: 136},
	    {name: 'Thor', strength: 100000},
	    {name: 'Black Widow', strength: 136},
	    {name: 'Vision', strength: 5000},
	    {name: 'Scarlet Witch', strength: 60},
	    {name: 'Mystique', strength: 120},
	    {name: 'Namora', strength: 75000},
	];
```
我们想找最强壮的英雄。使用 `for...of` 循环，像这样：
```javascript	
	let strongest = {strength: 0};
	for (hero of heroes) {
	    if (hero.strength > strongest.strength) {
	        strongest = hero;
	    }
	}
```
虽然这个代码可以正确运行，可是实在太烂了。看这个循环，每次都保存到目前为止最强的英雄。继续提需求，接下来我们想要所有英雄的组合强度值：
```javascript	
	let combinedStrength = 0;
	for (hero of heroes) {
	    combinedStrength += hero.strength;
	}
```
在这两个例子中，都在循环开始之前初始化了一个变量。然后在每一次的循环中，处理一个数组元素，并且更新这个变量。为了使循环变得清晰，现在把数组中间的部分进行重构，重构到函数中。我们要重命名这些变量，以进一步突出相似性。
```javascript	
	function greaterStrength(champion, contender) {
	    return (contender.strength > champion.strength) ? contender : champion;
	}
	
	function addStrength(tally, hero) {
	    return tally + hero.strength;
	}
	
	const initialStrongest = {strength: 0};
	let working = initialStrongest;
	for (hero of heroes) {
	    working = greaterStrength(working, hero);
	}
	const strongest = working;
	
	const initialCombinedStrength = 0;
	working = initialCombinedStrength;
	for (hero of heroes) {
	    working = addStrength(working, hero);
	}
	const combinedStrength = working;
```
写到这，两个循环变得非常相似了。它们两个之间唯一的区别是调用的函数和初始值不同。两个的功能都是对数组进行处理，最终得到一个值。所以，我们创建一个 *reduce* 函数来封装这个模式。
```javascript
	function reduce(f, initialVal, a) {
	    let working = initialVal;
	    for (item of a) {
	        working = f(working, item);
	    }
	    return working;
	}
```
*reduce* 模式在 JavaScript 中也是非常通用，因此 JavaScript 为数组提供了内置的方法，不需要自己来写。通过内置方法，代码就变成了：
```javascript	
	const strongestHero = heroes.reduce(greaterStrength, {strength: 0});
	const combinedStrength = heroes.reduce(addStrength, 0);
```
ok，如果你认真思考，你会注意到上面的代码其实并没有短很多。不过也确实比自己手写的 *reduce* 代码少写了几行。但是我们的目标并不是使代码变短或者少写，而是降低复杂度。那么，我们降低了复杂度了吗？我会说是的。我们把处理个体的循环代码给分离了出去，现在的代码具有很少的耦合性，即很少的互相调用，复杂度得以下降。

*reduce* 方法乍一看可能觉得非常基础。关于 *reduce* 的例子大部分也都很简单，比如做加法。但是没有人说 *reduce* 方法只能返回基本类型，它可以是一个 object 类型，甚至可以是另一个数组。当我首次意识到这个问题的时候，自己也是豁然开朗。所以我们其实可以用 *reduce* 方法来写 *map* 或者 *filter*，这里我把这个任务留给你们自己来尝试。

## filtering
现在我们有了 *map* 处理数组中的每个元素，有了 *reduce* 处理数组维度，经过计算降到只得到一个值。但是如果想获取数组中的某些元素该怎么办？我们来进一步探索，现在增加一些属性到上面的英雄数组中：
```javascript	
	const heroes = [
	    {name: 'Hulk', strength: 90000, sex: 'm'},
	    {name: 'Spider-Man', strength: 25000, sex: 'm'},
	    {name: 'Hawk Eye', strength: 136, sex: 'm'},
	    {name: 'Thor', strength: 100000, sex: 'm'},
	    {name: 'Black Widow', strength: 136, sex: 'f'},
	    {name: 'Vision', strength: 5000, sex: 'm'},
	    {name: 'Scarlet Witch', strength: 60, sex: 'f'},
	    {name: 'Mystique', strength: 120, sex: 'f'},
	    {name: 'Namora', strength: 75000, sex: 'f'},
	];
```
ok，现在有两个问题，我们想要：

1. 找到所有的女性英雄；
2. 找到所有能量值大于500的英雄。

使用普通的 `for...of` 循环，会得到如下代码：
```javascript	
	let femaleHeroes = [];
	for (let hero of heroes) {
	    if (hero.sex === 'f') {
	        femaleHeroes.push(hero);
	    }
	}
	
	let superhumans = [];
	for (let hero of heroes) {
	    if (hero.strength >= 500) {
	        superhumans.push(hero);
	    }
	}
```
上面代码运行起来没有问题，是不是看起来还不错？但是里面又出现了重复的情况。实际上，区别在于 `if` 的判断语句，那么能不能把 `if` 语句重构到一个函数中呢？
```javascript	
	function isFemaleHero(hero) {
	    return (hero.sex === 'f');
	}
	
	function isSuperhuman(hero) {
	    return (hero.strength >= 500);
	}
	
	let femaleHeroes = [];
	for (let hero of heroes) {
	    if (isFemaleHero(hero)) {
	        femaleHeroes.push(hero);
	    }
	}
	
	let superhumans = [];
	for (let hero of heroes) {
	    if (isSuperhuman(hero)) {
	        superhumans.push(hero);
	    }
	}
```
这种只返回 `true` 或者 `false` 的函数，我们一般把它称作 *谓词*。这里用了谓词来判断是否保存当前的英雄元素项。

上面代码的写法会看起来比较长。但是这样的重构很好地避免了之前的代码重复问题。可以进一步地抽象到一个函数中。
```javascript	
	function filter(predicate, arr) {
	    let working = [];
	    for (let item of arr) {
	        if (predicate(item)) {
	            working = working.concat(item);
	        }
	    }
	}
	
	const femaleHeroes = filter(isFemaleHero, heroes);
	const superhumans  = filter(isSuperhuman, heroes);
```
同 *map* 和 *reduce* 一样，JavaScript 提供了一个内置数组方法，没必要自己来实现（除非你自己想写）。用内置数组方法，上面的代码就变成了：
```javascript	
	const femaleHeroes = heroes.filter(isFemaleHero);
	const superhumans  = heroes.filter(isSuperhuman);
```
为什么这段代码比 `for...of` 循环好呢？回想一下整个过程，我们要解决一个“找到满足某一条件的所有英雄”。使用 *filter* 使得问题变得简单化了。我们需要做的就是通过写一个简单函数来告诉 *filter* 哪一个数组元素要保留。不需要考虑数组是什么样的，以及繁琐的中间变量。取而代之的是一个简单的谓词函数，仅此而已。

与其他的迭代器相比，使用 *filter* 是一个出小力办大事的过程。我们不需要通读循环代码来理解到底要过滤什么，要过滤的东西就在传递给他的那个函数里面。

## finding
*filter* 已经信手拈来了吧。这时如果只想找一个英雄该怎么办？比如找 “Black Widow”。使用 *filter* 会写出如下代码：
```javascript	
	function isBlackWidow(hero) {
	    return (hero.name === 'Black Widow');
	}
	
	const blackWidow = heroes.filter(isBlackWidow)[0];
```
这段代码的问题是效率不够高。*filter* 会检查数组中的每一个元素，而我们知道这里面只有一个 “Black Widow”，当找到她的时候就可以停住，不用再看后面的元素了。那么，依旧利用谓词函数，我们写一个 *find* 函数来返回第一次匹配上的元素。
```javascript	
	function find(predicate, arr) {
	    for (let item of arr) {
	        if (predicate(item)) {
	            return item;
	        }
	    }
	}
	
	const blackWidow = find(isBlackWidow, heroes);
```
同样地，JavaScript 已经提供了这样的方法：
```javascript	
	const blackWidow = heroes.find(isBlackWidow);
```
至此为止，*find* 再次体现了出小力办大事的原则。通过 *find* 方法，把问题简化为：你只要关注如何判断你要找的东西就可以了。不必关心迭代器到底怎么实现等细节问题。

## 总结
这些迭代器函数的例子很好地诠释了为什么“抽象”非常有用。回想一下我们所讲的内置方法，每个例子中我们都做了三件事：

1. 避免循环结构，使得代码变的简洁易读；
2. 通过适当的方法名称来描述我们使用的模式，也就是：*map*，*reduce*，*filter* 和 *find*；
3. 把问题从处理整个数组简化到处理每个元素。

这里要注意的是，我们把每个问题都打散，用一个或几个纯函数来解决。而真正令人兴奋的是仅仅通过 4 个模式（当然还有其他的模式，也建议大家去学习一下），在 JS 代码中你就可以消除几乎所有的循环了。这是因为 JS 中几乎每个循环都是用来处理数组，或者生成数组的。通过消除循环，降低了复杂性，也使得代码的可维护性更强。