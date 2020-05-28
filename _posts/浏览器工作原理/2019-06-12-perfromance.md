---
layout: post
title: "前端性能采集"
subtitle: 'JavaScript perfromace前端性能采集'
author: "Dongrh"
header-img: "img/banner/8.jpg"
catalog: true
tags:
  - JavaScript基础
  - 浏览器工作原理
---

待完善～

### 背景
对于公司来说，性能在一定程度上与利益直接相关。国外有很多这方面的调研数据：

| 性能                     | 收益     |
| :-------------           | :------------- |
| Google 延迟 400ms         | 搜索量下降 0.59%       |
| Bing 延迟 2s       | 收入下降 4.3%       |
| Yahoo 延迟 400ms          | 流量下降 5-9%       |
| Mozilla 页面打开减少 2.2s   | 下载量提升 15.4%       |
| Netflix 开启 Gzip       | 性能提升 13.25% 带宽减少50%       |

性能影响用户体验。家在的延迟，操作的卡顿等都会影响用户的使用体验，当一个页面家在较久，用户可能会直接选择放弃访问，因此知道自家网站性能表现情况，对提升自家网站的商业价值尤为重要，知道每个环节的耗时，对如何提升性能变得至关重要。

### 性能数据采集
Performance是前端性能监控的API。它可以检测页面中的性能，W3C性能小组引入进来的一个新的API，它可以检测到白屏时间、首屏时间、用户可操作的时间节点，页面总下载的时间、DNS查询的时间、TCP链接的时间等。

我们来打印一下这个对象：
![PNG](/img/performance/1.pic_hd.jpg)

- performance.memory对象

  jsHeapSizeLimit：内存大小的限制

  totalJSHeapSize： 总内存的大小。

  usedJSHeapSize：可使用的内存的大小。

  如果 usedJSHeapSize 大于 totalJSHeapSize的话，那么就会出现内存泄露的问题，因此是不允许大于该值的。

- performance.navigation对象
  redirectCount：如果有重定向的话，页面通过几次重定向跳转而来，默认为0；

  type：该值的含义表示的页面打开的方式。默认为0. 可取值为0、1、2、255.

      0（TYPE_NAVIGATE）：表示正常进入该页面(非刷新、非重定向)。

      1（TYPE_RELOAD）：表示通过 window.location.reload 刷新的页面。如果我现在刷新下页面后，再来看该值就变成1了。

      2（TYPE_BACK_FORWARD ）：表示通过浏览器的前进、后退按钮进入的页面。如果我此时先前进下页面，再后退返回到该页面后，查看打印的值，发现变成2了。

      255（TYPE_RESERVED）：表示非以上的方式进入页面的

- performance.onresourcetimingbufferfull：该属性的含义是在一个回调函数。该回调函数会在浏览器的资源时间性能缓冲区满了的时候会执行的。

- performance.timeOrigin：是一系列时间点的基准点，精确到万分之一毫秒。如上截图该值为：1590130008331.259，该值是一个动态的，刷新下，该值是会发生改变的。

- performance.timing：是一系列关键时间点，它包含了网络、解析等一系列的时间数据。
为了方便，从网上弄了一张图片过来，来解析下各个关键时间点的含义如下所示：

  ![PNG](https://static.geekbang.org/infoq/5c6bce24ba464.png)

  navigationStart：含义为：同一个浏览器上一个页面卸载结束时的时间戳。如果没有上一个页面的话，那么该值会和fetchStart的值相同。

  redirectStart:该值的含义是第一个http重定向开始的时间戳，如果没有重定向，或者重定向到一个不同源的话，那么该值返回为0.

  redirectEnd:最后一个HTTP重定向完成时的时间戳。如果没有重定向，或者重定向到一个不同的源，该值也返回为0.

  fetchStart:浏览器准备好使用http请求抓取文档的时间(发生在检查本地缓存之前)。

  domainLookupStart:DNS域名查询开始的时间，如果使用了本地缓存话，或 持久链接，该值则与fetchStart值相同。

  domainLookupEnd:DNS域名查询完成的时间，如果使用了本地缓存话，或 持久链接，该值则与fetchStart值相同。

  connectStart:HTTP 开始建立连接的时间，如果是持久链接的话，该值则和fetchStart值相同，如果在传输层发生了错误且需要重新建立连接的话，那么在这里显示的是新建立的链接开始时间。

  secureConnectionStart:HTTPS 连接开始的时间，如果不是安全连接，则值为 0

  connectEnd：HTTP完成建立连接的时间(完成握手)。如果是持久链接的话，该值则和fetchStart值相同，如果在传输层发生了错误且需要重新建立连接的话，那么在这里显示的是新建立的链接完成时间。

  requestStart:http请求读取真实文档开始的时间，包括从本地读取缓存，链接错误重连时。

  responseStart:开始接收到响应的时间(获取到第一个字节的那个时候)。包括从本地读取缓存。

  responseEnd：HTTP响应全部接收完成时的时间(获取到最后一个字节)。包括从本地读取缓存。

  unloadEventStart:前一个网页（和当前页面同域）unload的时间戳，如果没有前一个网页或前一个网页是不同的域的话，那么该值为0.

  unloadEventEnd:和 unloadEventStart 相对应，返回是前一个网页unload事件绑定的回调函数执行完毕的时间戳。

  domLoading:开始解析渲染DOM树的时间。

  domInteractive:完成解析DOM树的时间（只是DOM树解析完成，但是并没有开始加载网页的资源）。

  domContentLoadedEventStart：DOM解析完成后，网页内资源加载开始的时间。

  domContentLoadedEventEnd:DOM解析完成后，网页内资源加载完成的时间。

  domComplete:DOM树解析完成，且资源也准备就绪的时间。Document.readyState 变为 complete，并将抛出 readystatechange 相关事件。

  loadEventStart:load事件发送给文档。也即load回调函数开始执行的时间，如果没有绑定load事件，则该值为0.

  loadEventEnd:load事件的回调函数执行完毕的时间，如果没有绑定load事件，该值为0.


DNS阶段
![PNG](https://static.geekbang.org/infoq/5c6bc59402b9b.png)

### 各个阶段时间计算
重定向耗时 = redirectEnd - redirectStart;

DNS查询耗时 = domainLookupEnd - domainLookupStart;

TCP链接耗时 = connectEnd - connectStart;

HTTP请求耗时 = responseEnd - responseStart;

解析dom树耗时 = domComplete - domInteractive;

白屏时间 = responseStart - navigationStart;

DOMready时间 = domContentLoadedEventEnd - navigationStart;

onload时间 = loadEventEnd - navigationStart;


参考文档：
https://www.jianshu.com/p/1355232d525a

https://blog.csdn.net/sinat_17775997/article/details/88301120
