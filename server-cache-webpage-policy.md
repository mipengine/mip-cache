# Server cache web page policy

> Author:  [Gang, Chen](mailto:chengang06@baidu.com) [Breezet](taoqignqian01@baidu.com)

## Abstract

近几年，无论是内容分发平台还是浏览器厂商都会通过云端的页面CDN Cache服务，为用户访问Web页面提供更快的页面访问。通过更优质的CDN Cache，内容分发平台或浏览器厂商通过中间代理的方式，让用户享受了更优质的页面速度浏览体验。由于各公司提供的页面缓存服务在对缓存的处理策略不相同，也让站点在提供Web服务时，不清楚应该如何配置能被正确缓存并且对自己业务无其他负面影响。本文会详细讲述其中存在的问题，综合百度在此方面的处理方案给出建议的通用标准实现。

## Introduction

内容分发平台或浏览器对Web页面进行服务端的缓存，主要的目的是因为很多的站点服务本身并没有提供较好的网络环境和服务快速响应，通过将此类站点的页面缓存在CDN Cache等网络中（页面代理缓存），可以是用户访问此类站点时享受到极快的页面加载。

本文将从以下几个方面在总结内容分发平台或浏览器在代理缓存服务策略上的问题和解决方案：

1. Web site option for proxy cache server
2. Web site access info collection when page cached by proxy server
3. Protocol when prefetch cache page
 
## Web site option for proxy cache server

本小节主要讲述页面站点在被浏览器或内容分发平台的代理服务缓存时，所面临的问题，并给出对开发者更友好的缓存服务解决方案建议。

### issues

下图是一个用户访问站点时的请求所经过的缓存相关的路径。
 ![各类缓存在用户一次请求中所处的位置](http://bos.nj.bpc.baidu.com/v1/agroup/0e5f75439b7b188d765ee7e6924f22484ea07d29)

浏览器部分云加速服务，对页面的修改以及缓存对开发者过于透明不可控，包括但是不限于：

* 没有明确的配置协议，让控制哪些页面可以被缓存，哪些页面不能被缓存；
* 页面缓存失效的时间配置，是否沿用HTTP Header中的Cache-Control头，没有明确规范，而各类代理缓存服务对此的处理也不一致；

### solutions
代理缓存服务本身会在页面访问时抓取对应的页面，因此可遵循以下缓存策略：

* 使用robots.txt文件中新增字段描述站点url pattern粒度的是否允许被缓存；
* robots.txt文件本身可缓存时间用`Cache-Control`来控制；
* 缓存时间统一用`Cache-Control`头来控制缓存超时,用`stale-while-revalidate`头来控制平滑更新；
* 对于站点主动配置的缓存系统来说，robots.txt里边的禁止相关的内容，效果等同于Cache-Control: no-cache
* 代理缓存服务请求站点时，处理主要流程如下：
![图片](http://bos.nj.bpc.baidu.com/v1/agroup/6f48af9e732a5dad78e1ee01a3ab0b3d6fba20f5)

### use case

百度搜索结果页中的一个网站开发者，自己站点的`/home/news/data` 路径下的所有页面都是高时效性的页面，不希望被任何加速服务缓存。为了达到这个目的，站长应该在自己站点的`robots.txt`文件中加入如下内容：
```
Cache:*
HttpsCacheDisallow:/home/news/data/
HttpCacheDisallow:/home/news/data/
# 对于所有的Cache来说，https和http的在/home/news/data/路径下的所有内容不允许被缓存
# 如果希望各层加速能平滑更新，那么可以在Cache-Control头里面写入如下内容：max-age=864000, stale-while-revalidate=1728000
# 表明：在86400s内本地缓存有效。在172800s内返回旧缓存的同时，异步发起更新。当时间超过172800s时，缓存失效，重新抓取。
```

所有遵循了缓存规范的服务解析站点的robots.txt文件后，不缓存``/home/news/data``路径下的所有内容。满足了开发者的需求。
当然，作为站长，不能滥用此规范，因为不缓存的页面往往意味着更慢的加载速度。
当然，在这种场景下，处在HttpsCacheDisallow或者HttpCacheDisallow所配置的Path中的页面所返回的Cache-Control等header将会仅仅被用来控制浏览器端的缓存。

## Web site access info collection when page cached by proxy server

代理缓存服务进行页面缓存时，也会给站长带来数据统计上的困扰。本小节试图从数据统计的方面，提供缓存服务在统计上的一些解决方案。

### issues

* 部分页面强依赖请求的Client IP来做一些策略
* 部分站长需要实时知道自己站点的pv特征
* 缓存后，这些信息尚无明确规范回传给站长
* 目前，只有FireFox提出了基于x-forward-for的规范，但是这个规范不属于[任何组织](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/X-Forwarded-For)

### solutions

* 代理缓存服务回传回源等数据的时候都加上`x-forward-for`头来指名真实的用户IP信息
* HTML中新增用于日志回传的标签，用于告知浏览器需要将当前的x-forwad-for发送至对应地址，例如``<meta x-forwad-for='true' sendTo='domain/path'>``

## Protocol when prefetch cache page

在移动互联网下，仅仅是分布式的缓存服务还解决不了从缓存服务器到用户设备之间的移动网络传输慢的问题。浏览器或者内容分发平台通过适当的预取策略，提高打开速度，也是优化体验的重要手段之一。同样的，预取在解决了用户体验的同时，也给开发者带来了许多困惑。本小节将通过对规范的探讨来解决预取带来的问题。

### issues
* 开发者页面经过代理缓存服务没有有效的标识被浏览器识别是预取请求
* 对于不缓存的页面，不应该被预取，以免给站点带来太大压力

### solutions

* 对于允许缓存的页面，直接预取即可，但是header里边带上预取标识`X-moz: prefetch`，让站点知晓预取请求。目前，这个规范只有FireFox支持。

### 用例
对于站点来说，根据`X-moz`头来识别是否是预取请求，已完成更加精准的数据统计工作。对于浏览器或内容分发平台来说，和自己的代理缓存服务配合好，仅预取允许缓存的页面，防止给站点带来过大的压力。

