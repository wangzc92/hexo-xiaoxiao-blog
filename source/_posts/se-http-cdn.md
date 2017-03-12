title: 掌握 HTTP 缓存——从请求到响应过程的一切
author: Ulrich Kautz
tags:
  - HTTP
  - CDN
categories:
  - 前端笔记
  - Service
date: 2017-03-11 23:32:00
---
**CDN类的网站曾经一度雄踞 Alexa 域名排行的前 100。以前一些小网站不需要使用 CDN 或者根本负担不起其价格，不过这一现象近几年发生了很大的变化，CDN 市场上出现了很多按次付费，非公司性的提供商，这使得 CDN 变成人人都能负担的起的一种服务了。本文讲述的就是如何使用这种简单易用的缓存服务。**

使用内容分发网络（ CDN ）你需要先正确地认识 HTTP 响应头：和 HTTP 响应头中的哪些标签相关？它们是怎么起作用的？如何使用它们？文章中我会回答这些问题。

本文讲的并不会像教科书那么精确，实际上在某些情况下，为了叙述的清晰、简洁，我会按自己的理解简化某些问题，文章中会通过一些实际的例子来介绍缓存理论。在这篇文章的基础上，还会写一些文章来介绍对于某些指定的 CMS 或框架如何使用 CDN 来作为缓存层。

## 为什么使用 CDN？
CDN 是一个全球分布式网络，它把网站内容更快地传递给全球范围内的一个具体位置，而往往这个具体的位置离实际的内容服务器距离很远。举个例子，你的网站主机在爱尔兰，而你的用户则在澳大利亚访问。这时当你的用户访问你的网站的时候，延迟会很大，把你的（静态）数据用 CDN 放到澳大利亚则会很大程度上提高用户访问网站的体验。

然而 CDN 的使用并不局限于此。其实 CDN 可以理解成一个普通缓存，如**代理缓存**（边缘缓存）。即便你并不关心用户的具体地理位置，你也应该考虑使用 CDN 的代理缓存来提高你的用户体验。

## 为什么使用代理缓存？
简而言之，代理缓存会缓存你网站一些页面，通过缓存来传输“静态”内容非常快。一个简单的例子，假设你有一个带有开始页面的博客，这里面列出了所有近期的博客列表。完成这一过程，PHP 脚本要从数据库中获取到最近的文章实体，并且将它们转换成 HTML 结果页并返回给用户。因此，对于一次请求（访问）包含了：一次 PHP 执行 + 一组数据库查询。对于 1000 次请求（访问）包含了：1000 次 PHP 执行 + 1000 组数据库查询。每一次 PHP 执行都要进行 CPU、内存和 I/O 操作，对于数据库操作也是同样。

请求的需求量和访问用户的多少呈线性正比关系。听起来怎么样？不怎么样，因为这个线性关系是有最大限度的：磁盘最大只能提供一定程度的 I/O，CPU 和内存也都不是无限的。这样下去到了某个点，也就是说某个资源到了瓶颈的时候，就出现问题了：你的网站会访问的非常慢，甚至会出现所有人都不能访问的情况。其实这时其他资源并没有被全部打满。诚然，这时你可以扩展你的硬件规模来突破这一瓶颈，但是这将使工程变得很复杂，成本也更高。实际上还有更简单、更便宜的解决方法。

在中间加一层代理缓存，会减少资源对你的限制。拿前面的例子来讲，使用代理缓存只有第一次请求需要执行 PHP 脚本、查询数据库和生成 HTML 结果页。所有后面过来的请求都会从这个缓存中取内容，读取缓存几乎和直接读取内存一样快。这意味着，上面的线性规模瓶颈的问题解决了！100 个用户或者1000 个用户都没关系，依然只有 1 次 PHP 执行、1 次数据库查询和 1 次的结果页生成。

## CDN != CDN
CDN 的类型也各有不同。网站管理者可能会好奇数据是怎么存储的？存放在哪？以及数据是如何分布在 CDN 上的？是如何分发的呢？本文不是写给网站管理者的而是写给开发者的，所以在这我只能告诉你有“经典 CDN”和“对等 CDN”，后者是现在主流采用的方法。

对于开发者，相比于把数据拿到 CDN 以后做什么来说，会对如何把数据放到 CDN 中更感兴趣。说起来，有 **push CDN** 和 **pull CDN** 两种。顾名思义，“push CDN” 表示你要给 CDN 提供内容；“pull CDN” 表示如何从 CDN 取内容。

本文将主要介绍 pull CDN，因为在很多情况下 pull CDN 更加简单易用，不需要费多大事就能集成到现有的网站中。

## pull CDN 是如何起作用的？
我们来做个例子，假设你有一个可访问的网站，URL 是 https://www.foobar.tld。在这样的场景下，域名 www.foobar.tld 会被放到 pull CDN 服务器中，而不是你的网站服务器中。CDN 作为你网站服务器的一个**代理**。

还有一个不被公开的域名指向实际的网站服务器。在这个例子中假设它是 direct.foobar.tld，实际网站服务器叫做**源**。

这个 CDN 将会接受所有的请求。如果它的缓存中有结果的话将会直接返回给用户，否则会将这个请求托管给你实际的网站服务器，然后把返回的结果缓存起来为以后的请求做储备，同时将结果返回给用户。

![](https://huzidaha.github.io/images-store/201702/9-1.png)

最简单的 pull CDN 运行的过程如下：

* 获取一个页面的请求，这个页面：http://www.foobar.tld/some/page
* 把 some/page 当做缓存 key 检查缓存中是否存在
* 在缓存中则直接从缓存中返回结果给用户
* 不在缓存则请求 http://direct.foobar.tld/some/page，把返回的结果以 some/page 作为 key 写入缓存，并返回结果给用户

## 静态内容 VS 动态内容
上面的这一过程对于完全静态的内容完全适用。静态内容指的是如果用户访问同一个 URL 地址，返回的所有数据都是一样的。比如 CSS 文件就有这样的特点，http://www.foobar.tld/public/css/main.css 这个文件是一个普通文件，对于所有访问网站的用户都是一样的，那么它就特别适合用缓存存起来。

和静态文件相对的是动态文件。内容在运行时才能确定，这种情况也是非常常见的。比如多语言问题，需要根据浏览器语言来返回内容。还有一些和 “user session” 相关的内容，比如当用户登陆了以后，就要把“登陆”按钮换成“退出”按钮，你肯定不希望这个被缓存。这些高度活跃的内容（如每小时或者更短时间更新的页面）不能被缓存，或者说不能在缓存中停留时间过长。

这就是缓存有意思的地方，理解和实现它并不难。

## 缓存头
绝大多数的 pull CDN 采用以“每页”缓存形式解决动态内容的问题。为了达到这样的效果，一个简单的方法是 HTPP 响应缓存头。

首先对于缓存头你需要知道有“旧版本”和“新版本”两种，就是说它并不是一开始就设计成当前所使用的这个版本的，也有一个逐渐演变的过程。新版本指的是 HTTP/1.1，而旧版本指的是 HTTP/1.0。它有特别多的可选选项，每个人对这个问题都很头疼。我认为这是大家不愿意使用缓存头的最重要的原因。

言归正传，我们只关注 `ETag` 和 `Cache-Control` 这两个标签就足以了。大多数 CDN 还支持旧版本（`Expires`，`Pragma` 和 `Age`），不过这些只作为向后兼容来使用。

### ETag 头
我们从最简单的开始 `ETag`：它是文档版本的标识符。通常是内容的 MD5 值，不过它也可以包含其他内容，代表的是文档的版本/日期，如： 1.0 或者 2017-02-27。这里注意一点是，它必须用双引号括起来，如：`ETag: "d3b07384d113edec49eaa6238ad5ff00"`。

#### 二次验证
现在来考虑 `ETag` 的实际应用：二次验证。我们暂时不考虑前面代理+源的架构模式，只考虑简单的客户端-服务器模式。如下图：

![](https://huzidaha.github.io/images-store/201702/9-2.png)

假设客户端请求了 http://www.foobar.tld/hello.txt，接着服务端返回了如下的响应内容：
```	
	# REQUEST
	GET /hello.txt HTTP/1.1
	Host: www.foobar.tld
	
	# RESPONSE
	HTTP/1.1 200 OK
	Date: Sun, 05 Feb 2017 12:34:56 UTC
	Server: Apache
	Last-Modified: Sun, 05 Feb 2017 10:34:56 UTC
	ETag: "8a75d48aaf3e72648a4e3747b713d730"
	Content-Length: 8
	Content-Type: text/plain; charset=UTF-8
```	
`the body`在响应里面，有两个有意思的头标识：一个是 `ETag`，内容的 MD5值，一个是 `Last-Modified`，这是 `hello.txt` 文件最后一次被修改的时间。

这里就是二次验证起作用的地方：当客户端在很短的时间内再次访问上面的 URL，客户端浏览器会使用 `If-*` 请求头。如 `If-None-Match` 检查 `ETag` 的内容是否有改变。也就是说，如果 `ETag` 发生变化，客户端接收到的一个完整的新响应；如果 `ETag` 没变化，客户端接收到的是一个表明内容没变化的标识。
```
	GET /hello.txt HTTP/1.1
	If-None-Match: "8a75d48aaf3e72648a4e3747b713d730"
	Host: www.foobar.tld
```
如果 `ETag` 没有改变，那么服务端将会返回：
```	
	HTTP/1.1 304 Not Modified
	Date: Sun, 05 Feb 2017 12:34:57 UTC
	Server: Apache
	Last-Modified: Sun, 05 Feb 2017 10:34:56 UTC
	ETag: "8a75d48aaf3e72648a4e3747b713d730"
	Content-Length: 8
	Content-Type: text/plain; charset=UTF-8
```
正如上面所展示的，这次服务器的响应里面不是 *200 ok*，而是*304 Not Modified*，这就是说它略过包体部分，让客户端直接去自己的缓存里拿数据。在这个例子中，包体内容是 *the body*，比较小，效果不明显。可是想象一下如果是很大的内容呢，或者是很复杂的动态生成内容呢，价值就很大了。

作为一个开发者，你可能会想：“*并没有那么好用嘛，我还不得不掌握 IF- 类的头标识，比以前更费事了*”。

别急，这只是介绍了共享缓存，也就是代理缓存的由来，我们看原始的架构：<客户端-代理-源端>，代理根据自己的缓存返回给客户端 *304 Not Modified*，接下来的章节详解介绍，介绍之前我要先讲一下 `Last-Modfied` 头。

在处理上面那个 `hello.txt` 静态文件的例子时，客户端还可以使用 `If-Not-Modified-Since: Sun, 05 Feb 2017 10:34:56 UTC ` 来达到同样的效果（返回 304 响应）。这对于静态文件来说也很好用，因为响应头中的 `Last-Modified` 标识是根据服务器磁盘上的“更改时间戳”自动生成的。然而，“更改时间戳”对于动态文件通常没什么用，因为动态生成文件频繁更新，时间戳很难确定。我们都知道，你最想缓存起来的是内容，生成内容的代价是最大的，所以 `ETag` 头是更好的选择。

### `Cache-Control`头
`Cache-Control` 头相对来讲难一些。两个原因：第一，`Cache-Control` 既可以用于请求头，也可以用于响应头。本文中着重讨论响应头，因为这是开发者所必须要掌握的。第二，它控制着**两个缓存**：本地缓存（又称私有缓存）和共享缓存。

**本地缓存**，是指在客户端本地机器中的缓存。站在开发者的角度，它并不完全受你的控制，通常浏览器会自己决定是否把某些内容放到缓存中，这意味着：不要依赖于本地缓存。用户也可能在关闭浏览器的时候清理所有缓存，而你并不知道有这样的操作。除非你监测到了某个用户的流量不断上涨，导致缓存内容迅速失效，这时候你才会意识到。

**共享缓存**，也就是本文所介绍的：处于客户端和服务器之间的缓存。即 CDN。你对共享缓存拥有绝对的控制，应该好好地利用它。

现在我们来用一些代码作为示例深入学习一下。

1. `Cache-Control: public max-age=3600`
2. `Cache-Control: private immutable`
3. `Cache-Control: no-cache`
4. `Cache-Control: public max-age=3600 s-maxage=7200`
5. `Cache-Control: public max-age=3600 proxy-revalidate`

乍一看这些代码很令人困惑，但是不要担心，它并没有那么难，我来一点点介绍。首先你要知道 `Cache-Control` 有三种属性：缓冲能力、过期时间和二次验证。

首先是**缓冲能力**，它关注的是缓存到什么地方，和是否应该被缓存。他的几个重要的属性是：

* `private`：表示它只应该存在本地缓存；
* `public`：表示它既可以存在共享缓存，也可以被存在本地缓存；
* `no-cache`：表示不论是本地缓存还是共享缓存，在使用它以前必须用缓存里的值来重新验证；
* `no-store`：表示不允许被缓存。

第二个是**过期时间**，很显然它关注的是内容可以被缓存多久。它的几个重要的属性是：

* `max-age=<seconds>`：设置缓存时间，设置单位为秒。本地缓存和共享缓存都可以；
* `s-maxage=<seconds>`：覆盖 `max-age` 属性。只在共享缓存中起作用。

最后一个是**二次验证**，表示精细控制。它的几个重要属性是：

* `immutable`：表示文档是不能更改的。
* `must-revalidate`：表示客户端（浏览器）必须检查代理服务器上是否存在，即使它已经本地缓存了也要检查。
* `proxy-revalidata`：表示共享缓存（CDN）必须要检查源是否存在，即使已经有缓存。

通过上面的具体解释，现在再来描述上面 `Cache-Control` 的那段代码所表达的意思就好理解多了：

1. 本地缓存和 CDN 缓存均缓存 1 小时；
2. 不能缓存在 CDN，只能缓存在本地。并且一旦被缓存了，则不能被更新；
3. 不能缓存。如果一定要缓存的话，确保对其进行了二次验证；
4. 本地缓存 1 小时，CDN 上缓存 2 小时；
5. 本地和 CDN 均缓存 1 小时。但是如果 CDN 收到请求，则尽管已经缓存了 1 小时，还是要检查源中文档是否已经被改变。

#### 实例
理论会很单调乏味，现在用短的实例来演示如何自动注入 `ETag` 和 `Cache-Control` 头。例子是一个 Apache 的 `.htaccess` 文件，但是我希望你能够领会要领，并且根据你自己的实际情况，应用到你自己的 Web 应用中。
```	
	# 为所有图片设置 ETag，以及缓存时间为 1 天
	<FilesMatch "\.(gif|flv|jpg|jpeg|png|gif|swf)$">
	    FileETag -INode MTime Size
	    Header set Cache-Control "max-age=86400 public"
	</FilesMatch>
	
	# 为所有的 CSS 文件、JS 文件设置 ETag，以及缓存时间为 2 小时，同时保证进行了二次验证
	<FilesMatch "\.(js|css)$">
	    FileETag -INode MTime Size
	    Header set Cache-Control "max-age=7200 public must-revalidate"
	    Header unset Last-Modified
	</FilesMatch>
```
上面例子，是一个对 URL：http://www.foobar.tld/baz.jpg 的响应。包含了一个 `ETag` 头，由更改时间和文件大小所构成，还有 `Cache-Control` 头来设定缓存 1 天的时间。
```	
	# REQUEST
	GET /baz.jpg HTTP/1.1
	Host: www.foobar.tld
	
	# RESPONSE
	HTTP/1.1 200 OK
	Date: Tue, 07 Feb 2017 15:01:20 GMT
	Last-Modified: Tue, 07 Feb 2017 15:01:15 GMT
	ETag: "4-547f20501b9e9"
	Content-Length: 123
	Cache-Control: max-age=86400 public
	Content-Type: image/jpeg
```
对于 URL： http://www.foobar.tld/dist/css/styles.css 的响应同样也包含了 `ETag` 头。由更改时间、文件大小和限定了 2 小时的 `Cache-Control` 构成。`Last-Modfied` 头也删除掉以确保只有 `ETag` 用来做二次验证。
```	
	# REQUEST
	GET /styles.css HTTP/1.1
	Host: www.foobar.tld
	
	# RESPONSE
	HTTP/1.1 200 OK
	Date: Tue, 07 Feb 2017 15:00:00 GMT
	Server: Apache
	ETag: "20-547f1fbe02409"
	Content-Length: 32
	Cache-Control: max-age=7200 public must-revalidate
	Content-Type: text/css
```
## Cookies
你已经知道了缓存头是如何起作用的，现在我们来看下在缓存里面 cookie 起了什么作用。首先， Cookie 的设定也在 HTTP 响应头中，名字是 `Set-Cookie`。设置一个 cookie 的目的是标识这个用户，就是说你需要为每个用户设置一个 cookie。

想象一下缓存的场景，你是否会缓存一个包含了 `Set-Cookie`的 HTTP 响应，在缓存时间内，每个人都会得到相同的 cookie 和同样的用户 session？你肯定不想这样。

另外，用户 session 状态的改变可能会影响到响应内容的变化。一个简单的场景：电商购物车。你给用户要么提供一个空购物车，要么是用户自己选了很多物品的购物车。同样的道理，你不希望这个也被缓存，毕竟每个用户都应该有自己的购物车。

一个解决方法是在运行时通过 JavaScript 设置 Cookie，比如 Google Analytics。GA 通过 JS 设置 cookie，但这个 cookie 既不影响渲染，也不设置 `Set-Cookie` 头。GA 会在目标网站上添加类似于 "you are tracked via Google Analytics" 的图标，*但是只要这些改变都是在运行时添加进去的，就都没有问题*。

## 正确处理 cookie 和缓存
首先你需要知道你网站的 cookie 的工作原理。cookie 是不是只在特定时间使用（如在用户登录过程中使用）？原则上，cookie 是不是会被注入到所有响应？

正如上一节所说的，不论何时服务器返回了一个带有 `Set-Cookie` 的响应，你都希望能够保证它不会被缓存。那么问题就转化成为，当你返回一个带有“用户特性”内容的响应时（如购物车），CDN /代理服务器，会作何操作？

* 如果没设置 `Set-Cookie`，是不是允许缓存呢？
* 如果设置了 `Set-Cookie`，是不是自动丢弃所有 `Cache-Control` 头呢？

其实，如果从应用层面来讲，你尽管可以去实现你所喜欢的 web 应用就可以了，至于 cookie 和 CDN 都是自动设置的。还是用 Apache 的 `.htaccess` 来作为例子来解释：
```
	# 1) 如果 cookie 没设置，允许缓存
	Header set Cache-Control "public max-age=3600" "expr=-z resp('Set-Cookie')
	
	# 2) 如果 cookie 被设置，不允许缓存
	Header always remove Cache-Control "expr=-n resp('Set-Cookie')
	
	# 2a) 第二条的另一种形式，如果设置了 cookie，缓存时间设置成0
	Header set Cache-Control "no-cache max-age=0 must-revalidate" "expr=-n resp('Set-Cookie')
```
* 规则1：如果没设置 `Set-Cookie`，则给 `Cache-Control` 设置一个默认值；
* 规则2：如果设置了 `Set-Cookie`，则忽略 `Cache-Control`；
* 规则2a：是规则2的另一种表示形式，设置最大缓存时间是 0。

### 不设置 cookie 的访问路径
一些 CMS /框架还在使用一种暴力的方式种 cookie。而实际上，决定是否种 cookie 取决于不同的因素，比如会话时间因素。如果你有一个很高安全性的 web 应用，设置会话时间是 5 分钟，那么为每个响应设置一个新 cookie 都不过分。而假设你的应用连“用户特性”都没有，也就是说所有的东西对所有用户都是公用的，那么设置任何形式的 cookie 都是没有道理的。

所以下面这个例子是否适合你自己，很大程度上依赖于你的应用到底是什么类型的。我们来一起看一下，我先给一下这个例子的上下文关系：假设你有个新网站，你的所有文章都在 http://www.foobar.tld/news/item/<ID> 这个路径下面。现在你希望能够保证，所有访问 `/news/item/<ID>` 的路径都不包含 `Set-Cookie`，因为你确定不需要 cookie。
```	
	# 通用 PHP 重定向做法，将"?path=$1"写到重定向规则里
	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^(.*)$ index.php?path=$1 [NC,L,QSA]
	RewriteRule ^$ index.php [NC,L,QSA]
	
	# 利用 query 中的 path= 来判断
	<If "%{QUERY_STRING} =~ m#path=news/item/[^&]+#">
	    Header always unset Set-Cookie
	</If>
```
通过这样的设置，你就可以保证所有访问 `/news/item/<ID>` 的路径都不包含 `Set-Cookie`。而到底是否应该设置 cookie，需要你根据你自己的应用特点来判断。

### 设计出来的缓存能力
有很多设计方案可以使你的 web 应用具有高缓存性。鉴于本文仅仅是一篇文章而不是一本书，我不可能每个点都深入的来讲，但是我可以着重提一下通用的方法。

我还用电商作为例子。假设电商网站首页的 top 位置上展示了正在出售的物品，生成这些物品需要进行若干次的数据库操作，代价比较大，因此希望把它们缓存起来。但是，问题在于购物车，它是为那些登陆用户准备的，所以希望得到的结果是： top 物品是一样的，而针对登陆用户展示购物车。

![](https://huzidaha.github.io/images-store/201702/9-3.png)

那么优化策略首先要为每个用户提供一个和登陆状态无关的“通用”页。然后通过 JavaScript 为已经生成的网页提供购物车。站在用户的视角，最终展示形式是一样的。那么现在你有了两个请求（整个网页请求 + 购物车请求），而不是一个请求（整个网页请求，包含购物车）。ok，现在你可以把代价很大的部分，即 top 物品分离出来，把它们缓存起来了。

![](https://huzidaha.github.io/images-store/201702/9-4.png)

这种方法或者其延伸方法，不适合已经开发好的项目。因为它可能会改变很多接口和视图层（MVC 架构）的内容。最好你在一开始就设计好。

## 缓存失效：busting 和 purging
使用 `max-age` 和 `s-maxage` 你已经可以很好地控制一个指定的响应被缓存多长时间。但是这不足以适用于所有的情况。这些设置都是在返回响应时预设的，而现实情况往往是并不知道一个响应应该设置多久期满。回想一下刚才电商首页的例子：假设它包含了展示在 top 位置的 10 个实体。你设置了 `max-age=900`给这个首页以保证每15分钟刷新一次。现在，其中 1 个实体由于发布了太久了要被撤销，那么你就需要把之前的缓存响应删掉，这时候其实还没到 15 分钟，那么该怎么办？

不要担心，这是一个常见的问题，有很多方法解决。首先我们先来解释一下术语：

* **缓存 busting**，是用来解决浏览器长期缓存问题，它通过版本标识来告诉浏览器该文件有一个新的版本。这时浏览器将不会从本地缓存取内容，而从源服务器请求新版本的文件。
* **缓存 purging**，表示直接从缓存中删除内容（即响应），以使得缓存可以立马得到更新。

### 用于版本管理的缓存 busting
这种方法经常使用在 CSS 文件、JS 文件上。通常一个确切的版本号、一串哈希或者时间戳都可以用作标识，如下面的例子：

* 数字版本号：`style-v1.css`，`style.css?v=1`
* 哈希串版本：`style.css?d3b07384d113edec49eaa6238ad5ff00`
* 时间戳版本：`styles.css?t=1486398121`

这时候在发布程序的时候，你只要注意文件的版本就可以了。举个例子，一个 HTML 网页通过 `<link rel="stylesheet" href="..">` 这种形式包含了一个 CSS 文件。CSS 文件将会被缓存起来，这时如果你想让你的新 CSS 文件起作用，那么用最新的版本号命名它就可以。如果不做任何变化的话，即便你更新了文件，这个 HTML 还会使用缓存中的旧 CSS 文件。

### 缓存 purging
不同 CDN 供应商清除缓存的方式不一样。很多供应商都是基于开源软件 [Varnish](https://varnish-cache.org/) 来构建自己的 CDN 服务，所以一个通用的做法是在 HTPP 请求中使用 `PURGE` 结构，如：
```	
	PURGE /news/item/i-am-obsolete HTTP/1.1
	Host: www.foobar.tld
```
使用这个请求通常需要权限认证，或者是源确认（即 IP 白名单），不过不同供应商的要求也不一样。

清除一个或几个缓存项比较容易，但是在某些场景下，却不是这么简单。举个例子，一个博客的场景，博客里面都有关于作者的部分，现在你要改变关于作者的一些内容，那么你需要手动清理所有包含了作者信息的页面。你确实可以一个一个手动清理，但是假设你有成千上万个网页被影响了，那问题就变得麻烦了。

下面介绍一个解决方案。

### 代理标签
“代理标签” 这个名字来源于 CDN 供应商 [Fastly](https://www.fastly.com/)，不同供应商给它起的名字不一样，比如还有叫它“缓存标签”的，Varnish 叫它 [Hashtwo/Xkey](http://book.varnish-software.com/4.0/chapters/Cache_Invalidation.html#hashtwo-xkey-varnish-software-implementation-of-surrogate-keys)，这里我就不详细介绍其他供应商的情况了。

不论它叫什么，它们的目的都是一样的：给响应打标签。这样你就可以轻松地从缓存中删除相关的标签就可以，甚至都不用知道缓存的到底是什么东西。

还是拿<客户端-代理-源端>来举例子，源端返回一个含有代理标签的响应：
```	
	HTTP/1.1 200 OK
	Content-Type: text/html
	Content-Length: 123
	Surrogate-Key: top-10 company-acme category-foodstuff
```
这个例子中的标签为：`top-10`， `company-acme`，和 `category-foodstuff`。这里给一个电商的实际场景来理解其含义：这个响应包含了电商首页的前 10 个物品，这些物品由 ACME 公司提供，并且其目录类别都设定为食品类。

设置了标签以后，当物品发生了变化以后，你只需要删除包含有 `company-acme` 和 `top-10` 的标签就可以了。是不是很简单？

同样，具体如何清除缓存的操作方法，不同 CDN 供应商是不一样的。

## 写在最后
上面讨论的更多的是理论上的做法，还有很多文章专门介绍不同的 CDN 的使用。如果你想深入了解的话，下面的资料每篇可能都是你需要的。

* [谷歌开发者：HTTP 缓存](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching?hl=en#cache-control)
* [Push CDN 和 Pull CDN](http://www.whoishostingthis.com/blog/2010/06/30/cdns-push-vs-pull/)
* [CDN 类型（管理员视角）](http://www.the-toffee-project.org/index.php?page=32-cdn-content-delivery-networks-types)
* [缓存头概览](https://www.keycdn.com/support/http-caching-headers/)
* [缓存详解（Mozilla）](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching)
* [ETag头详解（Mozilla）](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag)
* [Cace-Control 头详解（Mozilla）](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)
* [If-None-Match 头详解（Mozilla）](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-None-Match)
* [Fastly：代理标签](https://docs.fastly.com/guides/purging/getting-started-with-surrogate-keys)
* [KeyCDN：缓存标签](https://www.keycdn.com/support/purge-cdn-cache/)

