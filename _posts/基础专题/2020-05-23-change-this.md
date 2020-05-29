---
layout: post
title: "改变this的指向"
subtitle: 'JavaScript 改变this的指向'
author: "Dongrh"
header-style: text
header-mask: 0.3
catalog: true
tags:
  - JavaScript基础
---

### 一、apply call bind异同
相同点：
- apply call bind 都是改变函数调用时this指向

不同点：
- apply call 会立即执行该函数, bind是返回绑定this指向后的函数
- apply和call的区别是传入参数格式不同，apply接收数组，call接收参数列

### 二、方法实现

#### 2.1 apply实现

思想：
- 重写 Function 原型上的call实现
- 当调用call传入的第一个参数为 undefined/null时，将期设置为 window
- 利用 this 指向原则，即：函数调用时的引用，决定了 this 的指向；因此，此时 使用 context.fn = this ；将函数的引用设置为context
- 最终直接调用 context.fn(aggs)，即可改变 this 的指向，指向完成后删除context.fn


```
Function.prototype.call = function(context){
    if (typeof this !== 'function') throw new TypeError("caller must be a function...");
    context = context || window;
    const args = Array.prototype.slice.call(arguments, 1);
    context.fn = this;

    // 利用 this 指向特性，函数调用时的引用，决定了函数内部 this 的指向
    const result = context.fn(...args)
    delete context.fn

    return result
}

```

#### 2.2 apply实现
与call方法的类似，只不过需要把参数改为数组
```
Function.prototype.apply = function(context){
    if (typeof this !== 'function') throw new TypeError("caller must be a function...");
    context = context || window;

    context.fn = this;

    let result;
    if (arguments[1]) {
      // 有传入参数 数组
      result = context.fn(arguments[1])
    } else {
      // 没有传入参数
      result = context.fn()
    }

    delete context.fn

    return result
}

```

#### 2.3 bind实现
思路：
- 调用 bind 后会返回一个函数，且需要改变 this 指向；此时，可以利用 call/apply来改变 this 的指向
- bind 只有第一次调用会改变 this 的指向，设计为闭包，且 this 由参数传入，将不会影响后续多次bind调用(多次调用，最终的 this 由第一次调用 this 指向决定)

```
Function.prototype.apply = function(context){
    if (typeof this !== 'function') throw new TypeError("caller must be a function...");
    context = context || window;

    const _this = this;
    const bindArgs = Array.prototype.slice.call(arguments, 1);

    return function inner(){
        const args = arguments;
        _this.apply(context, [].concat(args, bindArgs));

    }
}

function fn(){
    console.log(this.name);
}


const a = {
    name: "我是 a 对象"
}

const b = {
    name: "我是 b 对象"
}
fn.bind(a)()

fn.bind(a).bind(b)()
```
说明：先后打印的都是 "我是 a 对象"；

第一次bind调用后返回inner函数且并未调用，多次调用bind只是改变了_this，但并未改变context，因此多次调用bind，this指向由第一次决定
