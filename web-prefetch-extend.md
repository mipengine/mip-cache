# Web prefetch&prerender expand

> Author: [AZ](mailto:chenxiyang@baidu.com) [Breezet](mailto:taoqingqian01@baidu.com)

## Abstract

Mobile page visiting acceleration is an important factor of mobile web user expierence.Page rendering performance is important on page performance optimization, but in this document, will mainly introduce some thing out side of rendering per performance.Because web site rendering performance will be optimizated when them use some acceleration solution such as amp/mip,but if we want to make web page opening instantly when user was accessing search enginee's search result or opening content distribute platform's content,it also rely on a more accurate prefetch/prerender API.

## Browser prefetch/prerender issues

In mobile web browser,when user click search result,some thing will be proceed as follows:

DNS,Tcp connect,Send request,Server response,Net transmission,Page rendering,Resource loading.

For the mobile web users,every step will waste many time,after the actual test in Baidu search,the average website first screen time in Baidu search result was 3.5s.

Solutions such as amp/mip has been provided cdn cache system and some component for rendering,but for the papge load strategy and prefetch strategy,browsers also need to provide the ability to improve.Specific questions are as follows:

### Strategy of prefetch/prerender 
on the one hand, page load strategy optimization mainly consider size of page,network conditions,priority and so on. This is a business-related strategy to consider, there is no need for browser API improvements.

on the other hand, when web page use resource hints to prefetch/prerender page, the browser needs to consider which pages are suitable for prefetching, and take on prefetch/prerender sign in http requrest so that server side can use it to do some strategy or count this behavior.

### Process of prefetch/prerender 
Browser resource hints provide link prefetch/prerender tags for web page preload, but it also has some limitations such as lack of lack of feedback mechanism and lack of process control. Baidu search will provide different product strategy between page prefetch/prerender success and failed(strategy e.g: if failed, Baidu search will not excute page prefetch in this browser).

Both of these issues are currently temporarily lacking browser capabilities.

Also we have some proposals to solve this issues.

## some solutions

Consider how to provide the ability at the browser level, follows are the expcet ability that browser can provide.

### add prefetch options

Expand the link tag properties:

    - concurrent_num MAX concurrent page prefetch num
    - maxSize max prerender page size
    
###  prefetch/prerender policy
add browser prerender_level for tag attribute.
   
such as ``<link prerender_level="1">``

    * level 1，only parse dom and preload page resource such as image,script,css
    * level 2，parse dom&css and render dom tree
    * level 3，excute specific markup script
    * level 4，excute all script 

### preftech/prerender feedback

add callback for prefetch request, so that web page can get request status.

### prefetch/prerender sign

add sign in prefetch request header, serverside can recognize this is a prefetch request. Such as ``content-prefetch-policy``
