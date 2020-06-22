---
layout: post
title: "JavaScript reflect-metadata"
subtitle: '详解reflect-metadata功能'
author: "Dongrh"
header-style: text
header-mask: 0.3
catalog: true
tags:
  - reflect-metadata
---

### 引言
Relfect Metadata，简单来说就是你可以通过装饰器给类添加一些自定义的信息，而这些自定义的信息又不影响类的使用，即在类实例化后，在使用过程中，你不会感觉到有新增或减少属性或方法，并且这些添加的信息我们可以通过反射将其提取出来。

[github demo](https://github.com/dongrh214/vue-inversify-demo/)， git clone下来后，安装依赖，npm run serve启动服务，将路由切换到reflect-metadata（访问：http://localhost:8080/reflect-metadata）
```
import 'reflect-metadata'

@Reflect.metadata('classKey', 'demo class meta data')
class Demo {
  @Reflect.metadata('demoMethodPropertyKey', 'demo method property value')
  demoMethod (): void {
    console.log('do demo method...')
  }
}
// 取定义在类上的方法
console.log(Reflect.getOwnMetadata('classKey', Demo))
console.log(Reflect.getMetadata('classKey', Demo))

// 取定义在属性上的元数据
const demo = new Demo()
console.log(Reflect.getOwnMetadata('demoMethodPropertyKey', demo, 'demoMethod'))
console.log(Reflect.getMetadata('demoMethodPropertyKey', demo, 'demoMethod'))
const demo = new Demo()
console.log(Reflect.getOwnMetadata('demoMethodPropertyKey', demo, 'demoMethod'))
console.log(Reflect.getMetadata('demoMethodPropertyKey', demo, 'demoMethod'))

// 非装饰器定义元数据
Reflect.defineMetadata('demoMethodPropertyKey2', 'demoMethodPropertyKey2 value', demo, 'demoMethod')
console.log(Reflect.getMetadata('demoMethodPropertyKey2', demo, 'demoMethod'))
```
依次打印：
```
demo class meta data  
demo class meta data // Demo.prototype.constructor 为 Demo
undefined  // getOwnMetadata获取当前对象上元数据，但demoMethod实际定义在Demo.prototype
demo method property value
demoMethodPropertyKey2 value
```
### Reflect Metadata Api详解[摘自reflect-metadat类型定义文件]
```
declare namespace Reflect {
    /**
    *
      */
    function decorate(decorators: ClassDecorator[], target: Function): Function;
    /**
      *
      */
    function decorate(decorators: (PropertyDecorator | MethodDecorator)[], target: Object, targetKey: string | symbol, descriptor?: PropertyDescriptor): PropertyDescriptor;
    /**
      * 用于装饰器
      */
    function metadata(metadataKey: any, metadataValue: any): {
        (target: Function): void;
        (target: Object, targetKey: string | symbol): void;
    };
    /**
      * 在对象上面定义元数据
      */
    function defineMetadata(metadataKey: any, metadataValue: any, target: Object): void;
    /**
      * 在对象上面定义元数据
      */
    function defineMetadata(metadataKey: any, metadataValue: any, target: Object, targetKey: string | symbol): void;
    /**
      * 对象是否存在元数据
      */
    function hasMetadata(metadataKey: any, target: Object): boolean;
    /**
      * 对象属性是否存在元数据
      */
    function hasMetadata(metadataKey: any, target: Object, targetKey: string | symbol): boolean;
    /**
      * 在对象上面定义元数据
      */
    function hasOwnMetadata(metadataKey: any, target: Object): boolean;
    /**
      * 在对象属性上面定义元数据
      */
    function hasOwnMetadata(metadataKey: any, target: Object, targetKey: string | symbol): boolean;
    /**
      * 获取对象原型的元数据
      */
    function getMetadata(metadataKey: any, target: Object): any;
    /**
      * 获取对象原型属性的元数据
      */
    function getMetadata(metadataKey: any, target: Object, targetKey: string | symbol): any;
    /**
      * 获取对象自身的元数据
      */
    function getOwnMetadata(metadataKey: any, target: Object): any;
    /**
      * 获取对象自身属性的元数据
      */
    function getOwnMetadata(metadataKey: any, target: Object, targetKey: string | symbol): any;
    /**
      * 获取对象原型所有元数据的 Key
      */
    function getMetadataKeys(target: Object): any[];
    /**
      * 获取对象原型属性所有元数据的 Key
      */
    function getMetadataKeys(target: Object, targetKey: string | symbol): any[];
    /**
      * 获取对象所有元数据的 Key
      */
    function getOwnMetadataKeys(target: Object): any[];
    /**
      * 获取对象属性所有元数据的 Key
      */
    function getOwnMetadataKeys(target: Object, targetKey: string | symbol): any[];
    /**
      * 删除对象的元数据
      */
    function deleteMetadata(metadataKey: any, target: Object): boolean;
    /**
      * 删除对象属性的元数据
      */
    function deleteMetadata(metadataKey: any, target: Object, targetKey: string | symbol): boolean;
}
```
- Metadata Key {Any}：简写 k，元数据的 Key，对于一个对象来说，他可以有很多元数据，每一个元数据都对应有一个 Key。
- Metadata Value {Any}： 简写 v，元数据的类型，任意类型都行。
- Target {Object}：简写 o，表示要在这个对象上面添加元数据
- Property {String|Symbol}： 简写 p，用于设置在那个属性上面添加元数据。可以在对象上面添加元数据，也还可以在对象的属性上面添加元数据，当你给一个对象定义元数据的时候，相当于你是默认指定了 undefined 作为 Property
