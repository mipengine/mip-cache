# Server cache web page policy

> Author:  [Gang, Chen](mailto:chengang06@baidu.com) [Breezet](taoqignqian01@baidu.com)

## Abstract

Recently, content distribution platform and browser through provide server side page CDN cache to make user visite mobile web page faster. Unlike traditional CDN, they provide proxy caching when the user visit page, so that can accelerate page loading performance. Due to every CDN provider has different policy to cache page, web sites may also need to provide a variety of cache strategies when providing services. This document will describe these issues in detail, summary Baidu is currently doing the practice in this area to give a solution.

## Introduction

Content distribution platform or browser provide a server side cache to web page, because many web sites do not provide a good service and network environment. Through proxy this web sites to CDN cache, user can also visit this websites instantly. But the current specification is not clear, such as the industry commonly used ``x-forwad-for`` does not in any standard. This will lead to developers can not better deal with the user experience after the page cache

This document will summarize the following aspects of the content distribution platform and browser *proxy cache service* related issues and solutions.

1. Web site option for proxy cache server
2. Web site access info collection when page cached by proxy server
 
## Web site option for proxy cache server

This section mainly describe the problems mobile web sites will face when them page cached by content distribution platform or browser, and also put forward the better solutions for web developer.

### issues

Following image is the process when user visit page var content distribute platform cache.
 ![各类缓存在用户一次请求中所处的位置](http://bos.nj.bpc.baidu.com/v1/agroup/ffccdda4058200fbb2b7aa59f17c2cc6bd737561)

Browser acceleration service is a server proxy cache technology. On the modify page content strategy and the cache strategy, it is not visible to the developer. Including but not limited to:

* There is no clear agreement on how to control whether the page should be cached by the content distribution platform
* The statement of page cache expiration time, whether to use Cache control header in http request, there is no clear specification, and various types of cache proxy services processing is inconsistent.

### solutions
Proxy cache service crawl page when user visit the page var content distribution platform or browser, so it can follow the following caching strategy:

* Add new field to describe web sites url patterni in robots.txt, used to define a page whether be cached.
* robots.txt cahche expiration time can described by `Cache-Control`.
* Use `Cache-Control` header to control cache expiration time, and use [stale-while-revalidate](https://tools.ietf.org/html/rfc5861) header to update cache smoothly.
![stale-while-revalidate](http://bos.nj.bpc.baidu.com/v1/agroup/0521445b1415f7e3d0ba849c2fb3796df30e7854)  

Proxy cache service crawl web sites, process as follows:  

![图片](http://bos.nj.bpc.baidu.com/v1/agroup/783f2989ae849f447eb6686071d940707421ed9f)  


### use case

In Baidu search result, a web site url path `/home/news/data` need high timeliness, do not need to be cached. In order to achieve this purpose, web site should config `robots.txt` as follow:

```
Cache:*
HttpsCacheDisallow: /home/news/data/
HttpCacheDisallow: /home/news/data/
# For all cache, whether http or https page can not be cached in path `/home/news/data/`
# stale-while-revalidate addresses that by allowing a response to be used once it becomes stale, triggering a background refresh; 
# For instance, if a page cached by distribution platform web site want to set page update frequency, set Cache-Control: max-age=86400, stale-while-revalidate=172800.The response is fresh for 86400 seconds, and for an additional 172800 seconds, the stale response can be used.
```

## Web site access info collection when page cached by proxy server

when proxy cache service cache page, will bring the problems of statistical. Detail issues and solutions as follow:

### issues

* Parts of pages rely request client ip to do some strategy.
* There is no clear specification of how to pass back user request infomation to the site.
* Currently only Firefox propose a solution based x-forward-for, and it does not belong to [any organization](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/X-Forwarded-For).

### solutions

* When proxy cache service crawl page, add `x-forward-for` header to pass back real user client ip.
* Add a new meta define, used to inform the browser the address need to pass. Such as `<meta x-forward-for"true" sendTo="domain/path">`
