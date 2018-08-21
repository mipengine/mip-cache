# HTTP DNS Optimization

## Abstract
通常在网站节点异常时会调整域名的DNS解析映射，将用户流量调度到可用节点IP。传统的DNS协议存在解析收敛慢、中间节点缓存与劫持问题，推进HTTP DNS 目的旨在加速dns解析、提升DNS安全性。本文的考虑将HTTP-DNS作为数据服务嵌入HTTP header，不考虑建设internet集中式的HTTP-DNS解析系统，主要考虑点是依赖ISP做较多的基础设施建设。

## Introduction

传统的DNS解析通过UDP协议查询，Local DNS递归查询解析记录，权威NS server根据localdns ip判断需要返回的解析ip。对于网站的可用性重点关注以下几方面， * 解析效率：递归节点缺少统一控制，全网收敛时间通常大于TTL。HTTP alternete services（RFC 7838）提供了一种服务调度的思路，在加速解析方面不够灵活。

* 解析精度： 传统的递归解析使用local dns ip来判断就近返回的站点ip，不能精确反应真实用户地理未知。（权威DNS支持edns扩展RFC 7871，大部分ISP的递归节点不支持）。

* DNS安全：DNS query不是安全的协议，无法避免中间环节的劫持。


## DNS analyse efficiency and dispatch precision

### issues

* 递归解析由于递归层级和cache因素无法满足分钟级GTC调度(外网流量调度)；

* 站点IP连接故障的情况期望能立即重试备用IP；

* 递归节点和用户地理拓扑不一定临近，精准的流量调度需要用户ip。

### solutions

* HTTPS header传递解析信息: 浏览器HTTPS req header携带协议头询问服务端是否支持HTTP-DNS，如支持则webserver rsp header中携带解析信息，包括：{推荐IP，备用IP列表，TTL，缓存刷新标记，预解析接口}。

* 浏览器调度优先级: 浏览器解析header信息，下次请求根据header标记优先使用推荐ip调度请求，连接失败依次重试备用ip列表，备用ip不可用则回退到递归解析。浏览器发起的req header中携带重试信息，用于服务端GTC决策。 * 缓存方式：浏览器维护HTTP-DNS 缓存，{域名，推荐ip，备用ip，TTL}，rsp header的缓存刷新标记位非0时无论TTL是否过期立即刷新缓存。

* 预解析：TTL过期的HTTP-DNS信息在浏览器启动时用于预解析，按”浏览器调度优先级”描述的优先级建立解析缓存。预解析是一次HTTPS请求，（建议）rsp header中提供专用的解析接口，避免增加业务负载。

* 服务端调度: webserver对支持HTTP-DNS的浏览器请求，在header中返回解析信息。服务端GTC调度，则修改”推荐ip”和”缓存标志位”让浏览器立即刷新缓存。GTC根据用户clientip和对dns的重试信息，更加精准的调度。

* 用例

* 精准调度：浏览器透传用户ip和dns重试信息

* 解析速度：(参见header定义)缓存刷新标记位可以立即刷新解析信息，浏览器的预解析加速加速第一次解析，后续的解析复用缓存。

## DNS Secrurity

### issues

* web站点无法预期DNS劫持和缓存污染故障的发生，也无法施加有效控制，依赖ISP刷新缓存活用户切换Local DNS，止损流程复杂，通常服务持续有损。

### solution

* 基于TLS协议的解析信息交换：HTTP DNS中的解析信息交换必须使用TLS，在有预解析的情况下曾经成功发起过HTTPS请求的用户后续的DNS解析都是安全的。对于第一次发起HTTPS请求的用户仍然依赖递归解析，存在不安全。下文单独阐述

* 浏览器HTTP-DNS缓存隐私：HTTP-DNS设置高隐私等级，建议只读，减少本地篡改。

### special hint

第一次HTTPS请求的DNS安全：

* 浏览器预加载列表（建议）：对没有本地缓存记录的域名默认使用递归查询，异步对预加载列定义的探测接口探测HTTP-DNS信息，如域名支持HTTP-DNS则后续的请求使用更新后的缓存。预加载列表建议由权威中间机构维护（类似CA），web站点向中间机构注册，并约束SLA。

* 3G和wifi制式切换HTTP-DNS缓存刷新

* 制式切换用户网络接入拓扑发生变化，浏览器感知制式切换重建连接时强制立即刷新缓存。
