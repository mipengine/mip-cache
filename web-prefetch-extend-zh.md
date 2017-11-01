# Web prefetch&prerender 拓展

> Author: [AZ](mailto:chenxiyang@baidu.com) [Breezet](mailto:taoqingqian01@baidu.com)

## Abstract

在移动web体验中，让页面快速加载是一个非常重要的因素。页面本身的渲染性能是很重要，本周主要说的是关于浏览器渲染性能以外的页面性能体验瓶颈。当网站已经按照一些标准（类似amp/mip，或者简单的静态化基础html页面）进行构建之后，页面的渲染性能会被极大改善，但如果希望页面能达到瞬时展现的体验，还依赖类似搜索结果页本身的内容分发平台提供更强大的预取和预渲染API。

## Browser prefetch/prerender issues

移动浏览器下，点击搜索结果页结果，页面被渲染前有如下环节需要执行：
域名解析、建立连接、发起请求、处理请求、网络传输、页面渲染、资源加载。

对于移动互联网用户来说，这些环节太多，会耗费很多的时间，经过百度的实际测试，百度搜索结果中的站点平均首屏时间在3.5s左右。

类似amp和mip这类标准已经提供了cdn的缓存和渲染过程的优化，但是对于加载策略，预渲染控制，容器都还没有标准或者基于trick的实现，并没有办法达到瞬时打开的体验。具体存在的问题技术细节描述如下。

### Strategy of prefetch/prerender 
加载策略一方面考虑的是什么时候加载页面和资源，加载多少，从哪里加载以及加载的优先级。这一块更多的是业务应用上的考虑，不需要提供标准或浏览器的API。

另一方面考虑的是哪些页面适合被prefetch/prerender，以及prefetch/prerender对服务的HTTP请求的统计影响。而这一方面是需要有通用的标准（页面是否适合被prefetch/prerender）的，关于统计上的影响，也需要从浏览器发出的HTTP请求上考虑设计新的Policy（Policy Header）供服务端识别。

### Process of prefetch/prerender 
link prefetch/prerender另一方面的问题是缺乏反馈机制，以及对过程的控制。百度对页面是否进行了prefetch/prerender会有一些产品策略上的需求（提供瞬时展现交互体验）和过程跟踪上的需求（帮助决策用户是否当前环境是否适合prefetch/prerender）。

上述两个问题都是目前link prefetch/prerender暂未提供的能力。


## some solutions
我们从preftech和prerender的浏览器能力层面考虑，总结了目前期望浏览器提供的增强能力，具体内容如下。

### add prefetch options
    -   visible  可视区域加载
    -   Concurrent_num 并发数量
    -   scrollStop 滑动停止后才加载
    -   MaxSize 最大加载大小
    -   priority 优先级
    -   networkType 支持按不同网络制式去执行   
    
###  prefetch/prerender policy
考虑结合一定的页面标记将渲染分级，支持指定预渲染级别（prerender_level）
   
    *   level 1，进行dom的解析，可以预加载资源。
    *   level 2，完成CSS的渲染和页面结构的初步计算
    *   level 3，执行一部分特定标记的js代码
    *   level 4，执行全部的js   

### preftech/prerender反馈

可针对preftech/prerender的请求添加函数回调，使Web页面能获取请求过程状态。

### prefetch/prerender请求标识

发送的prefetch/prerender请求中携带标准标识，比如：``content-prefetch-policy``，告知服务端这是一个预取请求。

### 

### Use case

一个具体的例子是百度一个搜索结果页有10条搜索结果，其中前面结果被点击的概率大，所以从应用角度百度会通过link prefetch/prerender为需要的页面设置prefetch/prerender。但此部分prefetch/prerender请求一方面会因为站点本身网络与服务响应问题较慢，此类页面其实是不适合prefetch/prerender，而类似AMP/MIP之类缓存在CDN的页面，页面大小、网络条件以及服务端性能都比较可靠，此类页面就非常适合prefetch/prerender。

此外，站点非常在意被prefetch/prerender后，能否从服务端的访问日志中区别出是被预取还是正常展现。

