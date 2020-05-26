---
layout: post
title: "防抖和截流"
subtitle: 'JavaScript 防抖和截流简单实现'
author: "Dongrh"
header-style: text
header-mask: 0.3
catalog: true
tags:
  - JavaScript基础
---

### 防抖简单实现
函数防抖（debounce），就是指触发事件后，在 n 秒内函数只能执行一次，如果触发事件后在 n 秒内又触发了事件，则会重新计算函数延执行时间。
```
var debounce = function (idle, action) {
  var last;
  return function(){
    var ctx = this;args = arguments;
    clearTimeout(last);
    last = setTimeout(function(){
        action.apply(ctx, args);
    }, idle);
  }
}
```

### 截流简单实现
触发事件后函数不会立即执行，而是在delay秒后执行，如果在delay秒内又触发了事件，则会重新计算函数执行时间
```
var throttle = function (delay, action) {
  var last = 0;
  return function(){
    var ctx = this;args = arguments;
    var curr = Date.now();
    if (curr - last > delay) {
        action.apply(ctx, args);
        last = curr;
    }
  }
}
```
