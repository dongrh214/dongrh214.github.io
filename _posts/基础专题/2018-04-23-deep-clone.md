---
layout: post
title: "深拷贝简单实现"
subtitle: 'JavaScript 深拷贝简单实现'
author: "Dongrh"
header-style: text
header-mask: 0.3
catalog: true
tags:
  - JavaScript基础
---


### 简单实现
#### 方案一：
```
var obj = {name: "zhangsan", specs: {...}}
var newObjStr = JSON.stringify(obj);
var newObj = JSON.parse(obj);
```

#### 方案二：深度递归遍历
```
function deepClone(obj){
    var result = typeof obj.splice === 'function' ? [] : {}, key;
    if (obj && typeof obj === 'object'){
        for (key in obj){
            if (obj[key] && typeof obj[key] === 'object') {
                result[key] = deepClone(obj[key])
            } else {
                result[key] = obj[key];
            }
        }
        return result;
    }
    return obj;
}

var obj = {
    age: 12,
    specs: {
        hei:123,
        sdsd: [2,232]
    }
}

console.log(deepClone(obj))
```

### 衍生问题
对象最大深度
```
function maxLevel(obj){
    var level = 0
    if (obj && typeof obj === 'object'){
        if (Object.prototype.toString.call(obj) === '[object Object]') level++;
        for (key in obj){
            if (obj[key] && typeof obj[key] === 'object') {
                level += deepClone(obj[key]);
            } else {}
        }
        return level;
    }
    return level;
}

var obj = {
    age: 12,
    specs: {
        hei:123,
        sdsd: [2,{ggg:111}]
    }
}

console.log(deepClone(obj))
```
