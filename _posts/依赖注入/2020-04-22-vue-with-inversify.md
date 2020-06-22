---
layout: post
title: "JavaScript 依赖注入"
subtitle: 'vue基于inversify的ioc应用'
author: "Dongrh"
header-style: text
header-mask: 0.3
catalog: true
tags:
  - 依赖注入
  - inversify
---
### 背景

[InversityJS](https://github.com/inversify/InversifyJS) 是一个IOC(控制反转)框架，包括依赖注入(Dependency Injection) 和依赖查询(Dependency Lookup)。

相比于类继承的方式，控制反转解耦了父类和子类的联系[摘自网络]。

本文将基于最新@vue/cli脚手架创建一个新的vue应用，并将在项目中使用invserify及其主要扩展插件。项目代码[demo]

### 基础项目搭建
1、使用最新脚手架搭建vue项目，首先全局安装vue脚手架工具(已安装忽略)，本文github demo基于 @vue/cli 4.4.4
```
npm install -g @vue/cli
# OR
yarn global add @vue/cli
```
2、项目创建
```
vue create vue-inversify-demo
```
选择手动选择特性方式初始化项目，勾选Babel、Router、Vuex、Linter / Formatter、Unit Testing、Pre-processors，后续特性按各自需要选择

3、至此vue脚手架初始化的项目已可以运行，此文章对部分业务组件稍有改造，不完全对等脚手架初始化的项目，具体可看代码实现

4、引入inversify
```
npm i reflect-metadata inversify inversify-binding-decorators inversify-inject-decorators -S
```
- reflect-metadata：inversify所以来的规范
- inversify-inject-decorators：简化注入插件
- inversify-binding-decorators：简化binding插件

此处内容后续在其他文章再细说，本篇文章主要讲解vue结合inversify的实战，reflect-metadata单独使用暂不做展开介绍，后续其他文章将详细解释具体使用

5、目录结构
```
.
├── README.md
├── babel.config.js
├── package-lock.json
├── package.json
├── public
│   ├── favicon.ico
│   └── index.html
├── src
│   ├── App.vue
│   ├── assets
│   │   └── logo.png
│   ├── components
│   │   └── Inversify.vue
│   ├── entities
│   │   ├── Katana.ts
│   │   ├── Ninja.ts
│   │   ├── Shuriken.ts
│   │   └── index.ts
│   ├── interfaces
│   │   ├── ThrowableWeapon.ts
│   │   ├── Warrior.ts
│   │   ├── Weapon.ts
│   │   └── index.ts
│   ├── inversify.config.ts
│   ├── main.ts
│   ├── router
│   │   └── index.ts
│   ├── shims-tsx.d.ts
│   ├── shims-vue.d.ts
│   ├── store
│   │   └── index.ts
│   ├── types
│   │   ├── index.ts
│   │   └── ioc.ts
│   └── views
│       ├── About.vue
│       └── Home.vue
├── tests
│   └── unit
│       └── example.spec.ts
├── tsconfig.json
└── yarn.lock
```
6、创建接口定义(interfaces)、实体层等文件夹(entities)
- interfaces：存放接口定义
- entities：存放实体类

7、独立创建inversify配置文件
```
<!-- @/inversify.config.ts -->

import { Container } from 'inversify'
import getDecorators from 'inversify-inject-decorators'
import { provide, buildProviderModule } from 'inversify-binding-decorators'
const bootContainer = new Container()
const { lazyInject } = getDecorators(bootContainer, false)
export { provide, lazyInject, bootContainer, buildProviderModule }
```

8、创建interfaces
```
<!-- @/interfaces/Warrior.ts -->
// 武士
export interface Warrior {
  fight (): string;
  sneak (): string;
}

<!-- @/interfaces/Weapon.ts -->
export interface Weapon {
  hit (): string;
}

<!-- @/interfaces/ThrowableWeapon.ts -->
export interface ThrowableWeapon {
  throw (): string;
}

<!-- @/interfaces/index.ts -->
export { Warrior }from "./Warrior";
export { ThrowableWeapon }from "./ThrowableWeapon";
export { Weapon }from "./Weapon";
```

8、创建实体
```
<!-- @/entities/Katana.ts -->
import { Weapon } from '@/interfaces'
import { provide } from '../inversify.config'
import { IOC_TYPES } from '../types'
// 武士刀
@provide(IOC_TYPES.Katana)
export class Katana implements Weapon {
  public hit (): string {
    return 'katan hit...'
  }
}

<!-- @/entities/Shuriken.ts -->
// 手里剑
import { ThrowableWeapon } from '@/interfaces'
import { provide, lazyInject } from '../inversify.config'
import { IOC_TYPES } from '../types'
import { Katana } from './Katana'

@provide(IOC_TYPES.Shuriken)
export class Shuriken implements ThrowableWeapon {
  @lazyInject(IOC_TYPES.Katana)
  katana: Katana

  public throw (): string {
    return 'shuriken throw...'
  }
}

<!-- @/entities/Ninja.ts -->
// 忍者
import { ThrowableWeapon, Warrior, Weapon } from '@/interfaces/'
import { provide, lazyInject } from '../inversify.config'
import { IOC_TYPES } from '../types'
@provide(IOC_TYPES.Ninja)
export class Ninja implements Warrior {
  @lazyInject(IOC_TYPES.Katana)
  private katana: Weapon

  @lazyInject(IOC_TYPES.Shuriken)
  private shuriken: ThrowableWeapon

  public fight (): string {
    return this.katana.hit()
  }

  public sneak (): string {
    return this.shuriken.throw()
  }
}

```
至此，项目运行的基本元素以创建完成，接下来将在组件中引入被使用ioc的实体

### 实际引入
在components下创建Inversify.vue组件，并在views/Home.vue组件中使用该组件

```
<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <Inversify msg="Welcome to Your Vue.js App" text="text from home"/>
  </div>
</template>

<script>
// @ is an alias to /src
import Inversify from '@/components/Inversify'

export default {
  name: 'Home',
  components: {
    Inversify
  }
}
</script>
```

引入ioc的实体
```
<template>
  <div class="hello">
    <h1>{{count}}-{{double}}</h1>
    <h3>{{ msg }}</h3>
    <h2>{{text}}</h2>
    <button @click="startFight">startFight</button>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue, Watch } from 'vue-property-decorator'
import { IOC_TYPES } from '../types'
import 'reflect-metadata'
import { bootContainer, buildProviderModule, lazyInject } from '../inversify.config'
import { Ninja } from '../entities'

class People extends Ninja {}

@Component({
  props: {
    text: String
  }
})
export default class Inversify extends Vue {
@lazyInject(IOC_TYPES.Ninja)
// 此处直接使用Ninja会导致Ninja无法注入，导致异常
people: People;

@Prop() private msg!: string;
private count=0;
private double=0;

created (): void {
  bootContainer.load(buildProviderModule())
}

private startFight (): void {
  console.log(this.people.fight())
  console.log(this.people.sneak())
  this.count++
}

@Watch('count')
doubleCount (): void {
  this.double = this.count * 2
}
}
</script>
```
点击startFight按钮，将会在控制台打印people的功能，Inversify组件内部不需要手动创建对象

### 结束语
本篇主要讲解vue结合inversify及其主要插件的配合使用，具体每个插件的详细功能暂未做介绍，稍后将会在其他篇幅详细介绍
