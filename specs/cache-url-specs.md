# Cache URL规范

## 原则
Cache URL的设计应尽量简单，并且所包含信息能被还原为原URL，因此，URL应包含以下信息：

- 原URL协议（具体指HTTP或HTTPS）
- URL资源类型（页面、图片、其他资源）
- 独立子域名（用户隐私和站点Cookie数据保护考虑）

基于以上，Cache URL规范设计如下。

## 规范

对于页面内所有的引用的相关资源，Cache在处理时应将URL改写为Cache类型的URL。Cache URL类型有页面、图片和其他资源三种，Path上通过`/c`,`/i`,`/r`进行区分。

具体分类和规则示例如下表：
  
| 类型\示例        | 规则          | 示例 |备注|
| :-------------: |:-------------| :-----| :-----|
| HTTP MIP 页面      | 直接在https://{$cachedomain}/c/ 后面拼接mip的url即可| https://{$cachedomain}/c/abc.xyz/index.html |特有的css等content类型元素都可会按照此规范来修改。  注意去掉http://协议头|
| HTTPS MIP 页面     | 直接在https://{$cachedomain}/c/s/ 后面拼接MIP页面的url即可 | https://{$cachedomain}/c/s/abc.xyz/index.html |特有的css等content类型元素都可会按照此规范来修改。  注意去掉https://协议头|
|HTTP 图片| 直接在https://{$cachedomain}/i/ 后面拼接MIP页面引用图片的url即可| https://{$cachedomain}/i/abc/def.jpg|注意去掉http://协议头|
|HTTPS 图片| 直接在https://{$cachedomain}/i/s/ 后面拼接MIP页面引用图片的url即可 | https://{$cachedomain}/i/s/abc.xyz/def.jpg|注意去掉https://协议头|
|HTTP 资源|直接在http://{$cachedomain}/r/ 后面拼接MIP页面引用的url即可| http://{$cachedomain}/r/abc.xyz/casea.tff|已经支持|
|HTTPS 资源| 直接在https://{$cachedomain}/r/s后面拼接MIP页面引用的url即可|http://{$cachedomain}/r/s/abc.xyz/casea.tff |已经支持|

**泛域名替换规则**

上述示例中的{$cachedomain} 按照如下规则生成：
    
源站域名做以下处理：

    1.域名中包含"-"的，替换为"--"
    2.域名中包含"."的，替换为"-"

处理后作为cachecdn的子域组成以下域名：

m.120ask.com/path

转为

m-120ask-com.mipcdn.com/c/m.120ask.com/path



