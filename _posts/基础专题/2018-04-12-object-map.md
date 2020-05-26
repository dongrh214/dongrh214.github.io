---
layout: post
title: "JavaScript object与map区别"
subtitle: 'JavaScript object与map区别'
author: "Dongrh"
header-style: text
header-mask: 0.3
catalog: true
tags:
  - JavaScript基础
---

### 相同点
- JavaScript Object 和 Map都采取key、value键值对

### 不同点
#### 初始化不同
Map 继承自 Object 对象

Object 支持以下几种方法来创建新的实例：
```
var obj = {
  "key": "value"
};
var obj = new Object();
var obj = Object.create(null);
```
Map仅支持下面这一种构建方法：
```
var map = new Map(["key1", "value1"], [key2, value2]);
```
#### 键要求不同
Object 仅支持字符串或者是symbol
```
var str = "[object Object]";
var person = {
  name: "zhangsan"
}
var house = {}
house[person] = "haha"  
var key = Object.keys(house)[0];
console.log(key); // "[object Object]"
console.log(house[str]); // haha
console.log(house[person]); // haha
console.log(house[{}); // haha
```
即将person对象作为house的key，JavaScript内部会用toString将其转化为string，最终的key值为"[object Object]"，用person对象或{}对像取也会将person或{}转化为"[object Object]"

Map中可以是 JavaScript 支持的所有数据类型，因此可以用一个 Object 来当做一个Map元素的key

#### 数据读取
Object 可以通过 . 和 [ ] 来访问
```
obj.key;
obj[key]; // 如果key不为string，会将其toString
```
Map 想要访问元素，可以使用 Map 本身的原生方法：
```
map.get(key)
```

#### 判断元素是否存在
判断某个元素是不是在 Object 中需要以下操作：
```
obj.key === undefined;
// 或者
key in obj;
```
判断某个元素是否在 Map 中可以使用
```
map.has(key);
```

#### 新增数据
Object 新增一个属性可以使用：
```
obj['key'] = value;
obj.key = value;
```
Map 可以使用 set() 操作：
```
map.set(key, value)
```

#### 变量顺序
Chrome Opera中使用for-in语句遍历Object对象属性时会遵循一个规律：
它们会先提取所有 key 的 parseFloat 值为非负整数的属性，然后根据数字顺序对属性排序首先遍历出来，然后按照对象定义的顺序遍历余下的所有属性。
其它浏览器则完全按照对象定义的顺序遍历属性。

Map实例对象实例中数据的排序是根据用户push的顺序进行排序的

#### 删除数据
在 Object 中没有原生的删除方法，我们可以使用如下方式：
```
delete obj.id;
obj.id = undefined;// 下面这种做法效率更高，在使用 for … in… 去遍历的时候，仍然会访问到该属性
obj = {}; // 删除全部
```
Map 有原生的 delete 方法来删除元素：
```
map.delete(key); //  返回bool
map.clear(); // 删除全部
```

#### 获取size
Map 自身有 size 属性，可以自己维持 size 的变化，Object 则需要借助 Object.keys() 来计算。
```
console.log(Object.keys(obj).length);
```

#### Iterating
Map 自身支持迭代，Object 不支持。
```
console.log(typeof obj[Symbol.iterator]); // undefined
console.log(typeof map[Symbol.iterator]); // function
```

### Object 和 Map应用场景
- 当所要存储的是简单数据类型，并且 key 都为字符串或者 Symbol 的时候，优先使用 Object ，因为Object可以使用 字符变量 的方式创建，更加高效
- 当需要在单独的逻辑中访问属性或者元素的时候，应该使用 Object，例如：
```
var foo = {
    name: "zhangsan",
    age: 18,
    berif: function(){
        return ` ${this.name} ${this.age} years old`;
    }
}
foo.bar();
```
- JSON 直接支持 Object，但不支持 Map
- Map 是纯粹的 hash， 而 Object 还存在一些其他内在逻辑，所以在执行 delete 的时候会有性能问题。所以写入删除密集的情况应该使用 Map。
- Map 会按照插入顺序保持元素的顺序，而Object做不到。
- Map 在存储大量元素的时候性能表现更好，特别是在代码执行时不能确定 key 的类型的情况。
