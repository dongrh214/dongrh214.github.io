---
layout: post
title: "EventEmitter简单实现"
subtitle: 'JavaScript EventEmitter简单实现'
author: "Dongrh"
header-style: text
header-mask: 0.3
catalog: true
tags:
  - JavaScript基础
---

### 发布 + 订阅
DOM 的事件机制就是发布订阅模式最常见的实现，这大概是前端最常用的编程模型了，监听某事件，当该事件发生时，监听该事件的监听函数被调用。

### 简单实现
```
var EventEmitter = function(){
    this.events = [];
};

EventEmitter.prototype.on = function (eventName, eventHandler) {
    this.events.push({
        type: eventName,
        handler: eventHandler,
        done: false,
        once: false,
    });
}

EventEmitter.prototype.once = function (eventName, eventHandler) {
    this.events.push({
        type: eventName,
        handler: eventHandler,
        done: false,
        once: true,
    });
}

EventEmitter.prototype.off = function (eventName, eventHandler) {
    this.events.forEach((e, idx) => {
        if (e.eventName === eventName && e.handler === eventHandler) this.events.splice(idx, 1);
    })
}

EventEmitter.prototype.emit = function (eventName) {
    // 从下标为1的地方获取参数
    var args = [].slice.call(arguments, 1);
    var _this = this;
    this.events.forEach((e, idx) => {
        if (e.eventName === eventName) {
            if (e.handler && typeof e.handler === 'function') e.handler.apply(_this, args);
            if (e.once) {
                this.events[idx].done = true;
            }
        }
    })
}
```
