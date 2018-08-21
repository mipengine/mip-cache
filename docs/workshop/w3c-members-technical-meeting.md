## Abstract

Rencently,Baidu MIP team combined with W3C China in Beijing to convene a technical workshop about "The mobile web acceleration solution".The main goal is to explore the standard solution for the tricky implementation of a similar mobile web page acceleration solution such as AMP / MIP.We hope that through standard solutions to solve the web developers confusions and issues when they use AMP / MIP.

## Mainly conclusion

The technical workshop major members include Baidu,Alibaba,Tencent,UC,Xiao mi,CMCC,CUCC etc.We discuss about some important technical point in mobile web acceleration such as AMP / MIP.details as follows:


1. **Identify Browser URL**:how to sovle browser address bar has been shown not correct when current page async change navigation and open a new page by iframe.The main conclusion is we should build a stardand protocol for browser mapping AMP/MIP URL to Website Real URL. 

2. **Reliable CDN Cache**:how to solve AMP/MIP CDN cache can be reliable by brower or content distrubition platform that they can use CDN cache page instead of web page to provide faster page access.The main conclusion is we should build a protocol that browser can identify CDN cache page and use it like a canonical page.

3. **Link Prefech Ability**:how to use <link> prefetch/prerender friendly.The main conclusion is we should provide browser api for control or listner source prefetch.
             
All above is when the content distrubition platform load a AMP / MIP page that is neeeded to face.So we summary above point in the workshop,and we will keep going on explore technical details and find solution of this issues.

At the same time,we intend to set up a W3C CG to discuss technical design and implementation of above technical point with more companies.
