---
layout: post
title: "valueOf 和 toString理解"
subtitle: 'JavaScript valueOf 和 toString理解'
author: "Dongrh"
header-style: text
header-mask: 0.3
catalog: true
tags:
  - JavaScript基础
---

### valueOf
valueOf()函数用于返回指定对象的原始值。
该方法属于Object对象，由于所有的对象都"继承"了Object的对象实例，因此几乎所有的实例对象都可以使用该方法，但部分对象会对valueOf进行重写

| 对象     | 返回值     |
| :------------- | :------------- |
| Array       | 数组实例对象。      |
| Boolean       | 布尔值。      |
| Date	       | 以毫秒数存储的时间值，从 UTC 1970 年 1 月 1 日午夜开始计算。      |
| Function	       | 函数本身。      |
| Number	       | 数字值。    |
| Object	      | 对象本身。这是默认设置。    |
| String	       | 字符串值。 |


### toString
toString()函数用于将当前对象以字符串的形式返回。
该方法属于Object对象，由于所有的对象都"继承"了Object的对象实例，因此几乎所有的实例对象都可以使用该方法，但部分对象会对valueOf进行重写

| 对象     | 返回值     |
| :------------- | :------------- |
| Array       | 将 Array 的每个元素转换为字符串，并将它们依次连接起来，两个元素之间用英文逗号作为分隔符进行拼接。      |
| Boolean       | 如果布尔值是true，则返回"true"。否则返回"false"。    |
| Date	       | 返回日期的文本表示。      |
| Function	       | 返回如下格式的字符串，其中 functionname 是一个函数的名称，此函数的 toString 方法被调用： "function functionName() { [native code] }"     |
| Number	       | 返回数值的字符串表示。还可返回以指定进制表示的字符串    |
| Object	      | 返回"[object ObjectName]"，其中 ObjectName 是对象类型的名称。    |
| String	       | 返回 String 对象的值。 |
| Error	       | 返回一个包含相关错误信息的字符串。 |


### 实例
Array：重写了 toString()
```
var arr = [1, "2", 3];
console.log(arr.valueOf()); // [1, "2", 3]
console.log(arr.toString()); // 1,2,3
```
布尔：Boolean对象有重写toString方法
```
var b = true;
console.log(b.valueOf()); // true
var bObj = new Boolean(true);
console.log(bObj == bObj.valueOf()); // true
console.log(bObj === bObj.valueOf()); // false, bObj为对象，bObj.valueOf()为布尔值
console.log(bObj.toString()); // "true"
```
Date：重写了 toString() 也重写了 valueOf()
```
var date = new Date();
console.log(date.valueOf()); //1531915919230
console.log(date) // Wed Jul 18 2018 20:11:59 GMT+0800 (中国标准时间)
console.log(date.toString()); // Wed Jul 18 2018 20:11:59 GMT+0800 (中国标准时间)
```
Function：重写了 toString()
```
function foo(){}
console.log(foo.valueOf() === foo); // true
var bar = new Function("a", "b", "return a + b");
console.log(bar.valueOf() === bar); // true，两个都是引用类型

console.log(typeof bar.toString()); // string类型，值为：function anonymous(a,b
) {
return a + b
}
```
Number：Number对象有重写了 toString()
```
var num = 123.456;
console.log(num.valueOf()); // 123.456
console.log(num.toString()); // "123.456"
```
Object
```
var persopn = {
  name: "zhangsan",
  age: 23
}
console.log(persopn.valueOf() === persopn); // true

console.log(persopn.toString()); // "[object Object]"
```
String：String对象重写了 toString()
```
var str = "cbwhiewncsjkndf";
console.log(str.valueOf() === str); // true
var str2 = new String("sdkjscbsdcs");
console.log(str2.valueOf() === str2); // false, 两者的值相等，但不全等，因为类型不同，前者为string类型，后者为object类型

console.log(str2.toString())
```
Error：重写了 toString()
```
var err = new Error("wo cuo le");
console.log(err.valueOf() === err); // true
console.log(err.toString()); // "Error: wo cuo le"
```

### 类型转换应用

JavaScript类型转换对照表(来源于网络)
![PNG](/img/basic/javescript类型转换.png)

双等于号比较规则
- 首先看双等号前后有没有NaN，如果存在NaN一律返回false
- 再看双等号前后有没有布尔，有布尔就将布尔转换为数字。（false是0，true是1）
- 接着看双等号前后有没有字符串，有三种情况：

    1、对方是对象，对象使用toString()或者valueOf()进行转换；

    2、对方是数字，字符串转换为数字；

    3、对方是字符串，直接比较；

    4、其它返回false；
- 如果是数字，对方是对象，对象取valueOf()或者toString()进行比较，其它一律返回false
- null，undefined不会进行转换，但它们俩相等

#### 代码演示:

1、数字于对象比较
```
var arr = {};
arr.valueOf = function () { console.log("valueOf");return 1; }
arr.toString = function () { console.log("toString"); return 2; }
console.log(arr == 1);

//打印：
//valueOf
//true
//说明：先调用 valueOf返回1，直接与数字比较

var arr = {};
arr.valueOf = function () { console.log("valueOf");return "1"; }
arr.toString = function () { console.log("toString"); return 1; }
console.log(arr == 1);
//打印：
//valueOf
//true
//说明：先调用 valueOf返回 "1"，再通过parseInt转为数字，再次数字比较

var arr = {};
arr.valueOf = function () { console.log("valueOf");return this; }
arr.toString = function () { console.log("toString"); return 1; }
console.log(arr == 1);
//打印：
//valueOf
//toString
//true
//说明：先调用 valueOf返回 {}，再通过toString转为数字 1，再次数字比较
```

2、字符串与对象比较(与数字于对象比较类似)
```
var arr = {};
arr.valueOf = function () { console.log("valueOf");return 1; }
arr.toString = function () { console.log("toString"); return 2; }
console.log(arr == "1");
//打印：
//valueOf
//true
//说明：先调用 valueOf返回1，直接与数字比较

var arr = {};
arr.valueOf = function () { console.log("valueOf");return "1"; }
arr.toString = function () { console.log("toString"); return 1; }
console.log(arr == "1");
//打印：
//valueOf
//true
//说明：先调用 valueOf返回 "1"，再通过parseInt转为数字，再次数字比较

var arr = {};
arr.valueOf = function () { console.log("valueOf");return this; }
arr.toString = function () { console.log("toString"); return 1; }
console.log(arr == "1");
//打印：
//valueOf
//toString
//true
//说明：先调用 valueOf返回 {}，再通过toString转为数字 1，再次数字比较
```

备注：大多数对象隐式转换为值类型都是首先尝试调用valueOf()方法。
但是Date对象是个例外，此对象的valueOf()和toString()方法都经过精心重写，默认是调用toString()方法，比如使用+运算符，如果在其他算数运算环境中，则会转而调用valueOf()方法
```
var date = new Date(1531915919230)
date.valueOf = function(){console.log("valueOf"); return 1531915919230}
date.toString = function(){console.log("toString"); return "Wed Jul 18 2018 20:11:59 GMT+0800 (中国标准时间)"}
date

// 打印：
// toString
// Wed Jul 18 2018 20:11:59 GMT+0800 (中国标准时间)1


var date = new Date(1531915919230)
date.valueOf = function(){console.log("valueOf"); return 1531915919230}
date.toString = function(){console.log("toString"); return "Wed Jul 18 2018 20:11:59 GMT+0800 (中国标准时间)"}
date + 1

// 打印：
// toString
// Wed Jul 18 2018 20:11:59 GMT+0800 (中国标准时间)1

var date = new Date(1531915919230)
date.valueOf = function(){console.log("valueOf"); return 1531915919230}
date.toString = function(){console.log("toString"); return "Wed Jul 18 2018 20:11:59 GMT+0800 (中国标准时间)"}
date - 1

// 打印：
// valueOf
// 1531915919229


var date = new Date(1531915919230)
date.valueOf = function(){console.log("valueOf"); return 1531915919230}
date.toString = function(){console.log("toString"); return "Wed Jul 18 2018 20:11:59 GMT+0800 (中国标准时间)"}
date - "1"

// 打印：
// valueOf
// 1531915919229
```
