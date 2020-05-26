---
layout: post
title: "JavaScript执行上下文"
subtitle: 'JavaScript编译及执行原理'
author: "Dongrh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
header-style: text
catalog: true
tags:
  -
  - 浏览器工作原理
---

## 变量提升及作用域

### 变量提升背景
JavaScript设置之初为考虑提高性能，为曾预料到JavaScript会如此流行，在变量提升这一特性设计上，导致了很多与直觉不负的代码，这也是JavaScript的一个重要设计缺陷。

### 相关重要概念
#### 1、执行上下文
JavaScript执行前会进行编译，编译完成会生成执行上下文，执行上下文内部包含变量环境(VariableEnvironment)、词法环境(lexicalEnvironment))和可执行代码，其中变量环境中包含该执行上下文中变量，词法环境主要用于ES6后，同时支持变量提升和块级作用域而使用。
举个例子：
```
  var name = "zhangsan";
  console.log(age); // undefine;
  var age = 23;
  function eat(){
    var food = "火锅";
    console.log(name + " eat "+ food)
  }
  eat();
```

#### 2、作用域
ES6之前，作用域只体现在全局作用域和函数作用域，ES6后提出块级作用域的概念，通过let、const的使用使之JavaScript具备域java/C类似的作用域功能，从而避免JavaScript最初变量提升的缺陷。

#### 3、块级作用域
将变量作用限制于块以内，避免整个执行上下文的污染，需要借助let、const的使用。

#### 4、调用堆栈
JavaScript用于管理函数调用关系的栈的数据结构

#### 5、变量提升规则
- var变量提升: 创建、初始化、不赋值;
```
  console.log(a); // undefined;
  var a = 12;
  function fuc(){
    var b = 3;
    console.log(b); // 3
  }
  console.log(a); // 12;
  func();

  代码分析：
  1、全局代码编译，生成全局执行上下文，此时，全局执行上下文中的变量环境中会添加变量：var a = undefined;function func() {...};
  可执行代码包括:console.log(a);a = 12;console.log(a);func();
  2、全局执行上下文执行console.log(a)时a未赋值，但被初始化未undefined，因此打印undefied，执行a = 12后，变量环境中a的值被赋值为12，再次console.log(a)会打印12，再执行func()会对func进行编译；
  3、func代码编译，生成func执行上下文，此时func执行上下文中的变量环境会添加变量：var b = undefined;可执行代码为：b = 3; console.log(b);
  4、func可执行代码执行，执行b = 3时，func执行上下文变量环境中b被赋值为3，再打印console.log(b)会打印3
```
- const变量提升：创建、不初始化、不赋值;
- function变量提升：创建、初始化、赋值。
- 函数声明优先级高于变量提升优先级


#### 6、暂时性死区
```
  let name = "zhangsan"
  function func() {
    console.log(name);
    let name = "lisi";
  }
  func(); //Uncaught ReferenceError: Cannot access 'name' before initialization
```
【分析】：func外部的name定义在全局执行下上下文的词法环境栈底，而func内部的name，有变量提升在func执行上下文的词法环境栈底，let会变量提升，但不会初始化，也不会赋值，因此会报未初始化错误。
