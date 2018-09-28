# Cache 页面改写规范

## 原则(Principle)

Cache改写的原则是替换页面中潜在的影响页面网络加载的资源，更具体的，应该将页面以及页面内的图片，字体等资源纳入缓存（包括html中的以及css中的），并将页面内的所有资源改成同源地址。同时尽量处理Cache后对原页面造成的不透明影响。

Cache相关改写规范都是在围绕这个原则来进行设计和展开描述的。
  
## Cache 页面中URL改写 

Cache对页面中的各类资源URL（跳转链接、图片资源 、CSS、字体等）的改写分为两类

### Cache-URL改写

Cache-URL改写具体指：将需要Cache的URL改写为符合[Cache URL规范](./cache-url-specs.md)的URL。

    例如：
    /img/testb.jpg

    改写为

    https://mipcache.mipcdn.com/i/m.testdomain.com/img/testb.jpg

### 绝对路径补全

绝对路径补全具体指：将页面中的相对路径改为带域名的绝对路径。一般用于跳转链接，是针对Cache对原页面造成的不透明影响的一种补充修复机制。

    例如： 

    /link-out/testa.html 

    改写为 

    http(s)://m.testdomain.com/link-out/testa.html 


综上，被改写的页面URL类型如下表:

|标签名称|标签类别|属性|绝对路径补全|MIPCache-URL规范改写|备注|
|----- |----| -----| -------- |-------------------| ---| 
|a|	html|	href| 是| 否||
|link|	html|href|是|rel=stylesheet做mip-type改写，否则只做相对路径改写;rel=manifest做mip-type改写，否则只做相对路径改写|
|style|html|src|是|是||
|mip-*	|MIP|	src|是|是|以MIP开头的标签默认的处理方式，表格中提及要特殊处理的不适用此规则|
|mip-video|MIP|poster|是|是||
|mip-pix|MIP|src|是|否||


除去表格中的标签外，页面内嵌的css中的图片，字体等资源，MIPCache也应该尝试抓取并缓存。

### 不做改写

页面中可能存在部分浏览器不能直接解析执行的代码，例如`<template>`

当前不做改写标签：

`<template>`,`<mip-script>`

对于此类标签中的内容，Cache不做改写。（持续更新）

### 特殊处理

Cache为了实现对部分通用组件的加速加载，会将组件的JS代码前置到Cache页面头部。

此部分不作为改写时的强制要求，但在进行特殊处理时，需要在规范中进行说明与补充，保证规范的有效性和一致性。理想的状态是特殊处理的Case也应该作为一种通用改写规则声明到通用规范中。

以下是特殊处理Case列表：

#### 广告JS前置

具体示例如下：

改写前页面片段：

    <!DOCTYPE html>
    <html mip>
    <head>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<meta name="viewport" content="width=device-width,minimum-scale=1,initial-scale=1" />
    ...
    <script src="https://mipcache.bdstatic.com/static/v1/mip.js"></script>
    <script src="https://mipcache.bdstatic.com/static/v1/mip-ad/mip-ad.js"></script>


改写后页面片段：

    <!DOCTYPE html>
    <html mip>
    <head>
    <script async src="https://wm.mipcdn.com/vitxtitxfwzswyfrtxywzf.js" mip-preload="mip-script-wm"></script><script async src="https://wm.mipcdn.com/ofmqjsbbyygkmqxmshumbuthhvzmrtumoezombmimbaohcmgmrvwmymomgmbqdmkdcmydemimqybf.js" mip-preload="mip-script-wm"></script>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<meta name="viewport" content="width=device-width,minimum-scale=1,initial-scale=1" />
    ...
    <script src="https://mipcache.bdstatic.com/static/v1/mip.js"></script>
    <script src="https://mipcache.bdstatic.com/static/vs/4c04a/8d3d4e4956.js"></script>

### AMP兼容
此外，Cache目前也需要兼容AMP标准，针对AMP特殊的标签，URL的处理规则如下：

|标签名称|标签类别|属性|绝对路径补全|MIPCache url规范改写|备注|
|----- |----| -----| -------- |-------------------| ---| 
|link|	html| href| 是| rel为manifest则处理此规则，否则不处理||
|amp-anim|AMP|src|是|是||
|amp-audio|AMP|src|是|是||
|amp-ad|AMP|data-slot|是|否||
|amp-google-vrview-image|AMP|src|是|否||
|amp-iframe| AMP|src|是|否||
|amp-img|AMP|src|是|是||
|amp-install-serviceworker|AMP|src|是|是||
|amp-list|AMP|src|是|否||
|amp-video|AMP|poster|是|是||
