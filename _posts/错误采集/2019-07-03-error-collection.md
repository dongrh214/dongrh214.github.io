---
layout: post
title: "前端错误采集"
subtitle: 'JavaScript 浏览器端错误采集'
author: "Dongrh"
header-img: "img/banner/2.jpeg"
catalog: true
tags:
  - JavaScript基础
  - 浏览器工作原理
---
### 背景
随着前端应用的越趋复杂，以及用户对应用体验的要求越来越高，前端开发工作者在完成功能开发后，并不意味着开发工作已经结束，我们需要知道用户的访问体验如何，可用性如何，开发同学以及测试同学往往无法确保应用无任何缺陷，作为一个合格的开发者，应该时刻关注应用线上的运行情况。那么如何采集错误信息将变得尤为重要，本编文章将着重探讨错误该如何收集，以及如何分析。

### 错误采集
#### 1.1 运行时错误
运行时错误采集可以使用window.onerror和window.addEventListener('error')捕获，但window.onerror会报出运行时错误、语法错误发生时触发，window.onerror异常捕获不能捕获promise的异常错误信息
```
window.onerror = (errMsg, errScript, lineNum, colNum, err) => {
  // 可以拿到的是被throw出来，没有被catch过的错误
  console.log(`errMsg:${errMsg}`); // 错误发生的异常信息（字符串）
  console.log(`errScript:${errScript}`); //错误发生的脚本URL（字符串）
  console.log(`lineNum:${lineNum}`); //错误发生的行号（数字）
  console.log(`colNum: ${colNum}`); //错误发生的列号（数字）
  console.log(`err: ${err}`); //错误发生的Error对象（错误对象）
};
```
#### 1.2 捕获跨域脚本错误
出于安全上的考虑，部分浏览器拿到一个Script error并没有错误本身的message、url等信息，在lineNum和colNum也都是0，并不是真正错误发生时的错误信息的地方。
出现这种情况是浏览器在同源策略限制下所产生的，当页面引用的非同域的外部脚本中抛出了异常，此时本页面无权限获得这个异常详情， 将输出 Script error 的错误信息。在Chrome中有这样的安全机制，他不会将完整的跨域错误信息暴露给你，只在chrome中会出现这样的情况，在Firefox，Safari中均可以正常的拿到完整的错误信息。
解决办法：
- 使用跨源资源共享机制( CORS )，在跨域脚本上配置crossorigin="anonymous"属性
- 响应头中增加 Access-Control-Allow-Origin 来支持跨域资源共享
- window.addEventListener('error')捕获资源加载错误
```
window.addEventListener('error', (errorEvent) => {
    console.log(errorEvent)
    cosnole.log(errorEvent.message)
}, true)
```
此处拿到的是的是这里拿到的是一个event事件，和前面不一样，拿到的并不是一个error对象。且window.addEventListener('error',()=>{})可以监听运行时错误，因此需要将其于window.onerror区分
区分方法：
- event.srcElement inatanceof HTMLScriptElement或HTMLLinkElement或HTMLImageElement时才上报
- errorEvent.message字段，其他的普通错误会有一个message字段，资源加载错误没有这个字段

#### 1.3 捕获unhandledrejection错误
Promise中的错误并不能被try...catch和window.onerror捕获。这时候我们就需要unhandledrejection来帮我们捕获这部分错误
```
window.addEventListener('unhandledrejection', (e) => {
  console.log(e.reason); //异常处理方法中的错误原因（如果存在）
  console.log(e.promise); //特定的 Promise 被 reject 而没有被相应的异常处理方法所处理的promise对象
});
```


#### 1.4 console.error
console.error常常被视为打印的日志，可预知的错误，已经被捕获的错误，已经被处理过的内容。此类错误一般是已知错误，但错误打印前，我们可以对console.error二次封装，在打印前打出采集错误
```
console.error = (func => {
  return (...args) => {
    // 在这里就可以收集到console.error的错误
    func.apply(console, args);
  }
})(console.error);
```
#### 1.5 对xhr和fetch封装

### sourceMap使用
解决跨域或者将脚本存放在同域之后，你可能会将代码压缩一下再发布，这时候便出现了压缩后的代码无法找到原始报错位置的问题。如图，我们用webpack将代码打包压缩成bundle.js。
此时报的错误lineNo可能是一个非常小的数字，一般是1，而columnNo会是一个很大的数字，这里是10000，因为所有代码都压缩到了一行。
这是我们需要开启source-map了，没错，我们利用webpack打包压缩后生成一份对应脚本的map文件就能进行追踪了，在webpack中开启source-map功能
```
// demo： webpack
module.exports = {
    ...
    devtool: '#source-map',
    ...
}
```
//打包压缩的文件末尾会带上这样的注释：
```
!function(e){var o={};function n(r){if(o[r])return o[r].exports;var t=o[r]={i:r,l:!1,exports:{}}...;
//# sourceMappingURL=bundle.js.map
```
//意思是该文件对应的map文件为bundle.js.map。下面便是一个source-map文件的内容，是一个JSON对象：
```
version: 3, // Source map的版本
sources: ["webpack:///./common/components/InitProgress/index.scss?5018", ...], // 转换前的文件
names: ["uploadFuncs", "__webpack_require__", ...], // 转换前的所有变量名和属性名
mappings: ";;;;;;;;;AAAA;AACA;AACA;AACA;AACA;AACA;AACA;AACA;AACA;AACA;AACA;A...", // 记录位置信息的字符串
file: "static/chunks/styles.js", // 转换后的文件名
sourcesContent: ["// extracted by extract-css-chunks-webpack-plugin\nmodule.exports = ..."], // 源代码
sourceRoot: "" // 转换前的文件所在的目录
```
如此，既然我们拿到了对应脚本的map文件，那么我们该如何进行解析获取压缩前文件的异常信息呢？这个我会在下面异常上报的时候进行介绍


### 异常上报
以上介绍了前端异常捕获的相关知识点，那么接下来我们既然成功捕获了异常，那么该如何上报呢？

在脚本代码没有被压缩的情况下可以直接捕获后上传对应的异常信息，这里就不做介绍了，下面主要讲解常见的处理压缩文件上报的方法。

#### 上报异常
当捕获到异常时，我们可以将异常信息传递给接口，以window.onerror为例：
```
function upload(errorInfo) {
  if (XMLHttpRequest) {
    var xhr = new XMLHttpRequest();
    // 上报给服务器，由nginx转发到node中间层，稍后定义
    hr.open('post', '/api/fe/errorMsg', true);
    // 设置请求头，plain/text方便复杂post请求跨域
    xhr.setRequestHeader('Content-Type', 'plain/text');
    // 发送参数
    xhr.send(JSON.stringify(errorInfo));
  } else {
    console.log("current browser is not support XMLHttpRequest...")
  }
}


window.onerror = function(errorMessage, scriptURI, lineNo, columnNo, error) {
    // 构建错误对象
    var errorInfo = {
        errorMessage: errorMessage || null,
        scriptURI: scriptURI || null,
        lineNo: lineNo || null,
        columnNo: columnNo || null,
        stack: error && error.stack ? error.stack : null
    };
    upload(errorInfo);
}
```

#### sourceMap解析
其实source-map格式的文件是一种数据类型，目前市面上也有专门解析它的相应工具包，在浏览器环境或者node环境下比较流行的是一款叫做’source-map’的插件。
引入该插件后，前端浏览器可以对map文件进行解析，但因为前端解析速度较慢，所以这里不做推荐，我们还是使用服务器解析。
如果你的应用有node中间层，那么你完全可以将异常信息提交到中间层，然后解析map文件后将数据传递给后台服务器，中间层代码如下：
```
const express = require('express');
const fs = require('fs');
const router = express.Router();
const fetch = require('node-fetch');
const sourceMap = require('source-map');
const path = require('path');
const resolve = file => path.resolve(__dirname, file);

function uploadErrorLog(errorInfo, successHandler, errorHandler) {
  let url = ''; // 上报地址
  // 将异常上报至后台
  fetch(url, {
      method: 'POST',
      headers: {
          'Content-Type': 'application/json'
      },
      body: JSON.stringify()
  }).then(function(response) {
      return response.json();
  }).then(function(json) {
      successHandler(json)       
  }).catch(function(e){
    errorHandler(e);
  });
}


// 定义post接口
router.post('/errorMsg/', function(req, res) {
    let error = req.body; // 获取前端传过来的报错对象
    let url = error.scriptURI; // 压缩文件路径
    if (url) {
        let fileUrl = url.slice(url.indexOf('client/')) + '.map'; // map文件路径
        // 解析sourceMap
        let smc = new sourceMap.SourceMapConsumer(fs.readFileSync(resolve('../' + fileUrl), 'utf8')); // 返回一个promise对象
        smc.then(function(result) {
            // 解析原始报错数据
            let ret = result.originalPositionFor({
                line: error.lineNo, // 压缩后的行号
                column: error.columnNo // 压缩后的列号
            });
            const errorInfo = {
                errorMessage: error.errorMessage, // 报错信息
                source: ret.source, // 报错文件路径
                line: ret.line, // 报错文件行号
                column: ret.column, // 报错文件列号
                stack: error.stack // 报错堆栈
            }
            uploadErrorLog(errorInfo, (json) => {
                res.json(json);
            }, (e) => {
                res.json(e);
            });
        })
    } else {
      res.json({
        success: true
      });
    }
});

module.exports = router;
```
这里我们通过前端传过来的异常文件路径获取服务器端map文件地址，然后将压缩后的行列号传递给sourceMap返回的promise对象进行解析，通过originalPositionFor方法我们能获取到原始的报错行列号和文件地址，最后通过ajax将需要的异常信息访问日志服务器，通过配置nginx生成访问日志，以便接下来的解析展示
