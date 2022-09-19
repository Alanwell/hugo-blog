---
title: "浅析vite"
date: 2022-09-19T11:51:30+08:00
draft: false
tags: ["vite"]
categories: ["vite"]
---

<!-- ---
marp: true
theme: default
header: 浅析vite
paginate: true
--- -->

## 浅析vite

> Vite(读音类似于[weɪt]，法语，快的意思) 是一个由原生 ES Module 驱动的 Web 开发构建工具。在开发环境下基于浏览器原生 ES imports 开发，在生产环境下基于 Rollup 打包。

---

### Vite有如下特点：
<br>

- Lightning fast cold server start - 闪电般的冷启动速度
- Instant hot module replacement (HMR) - 即时热模块更换（热更新）
- True on-demand compilation - 真正的按需编译

---

Vite 与传统打包方式的对比

<br>

![](https://cdn.jsdelivr.net/gh/Alanwell/img-base/vs.jpg)


---
**在了解Vite之前，需要先了解下ESM**

随着我们的应用程序越来越大，我们希望将其拆分为多个文件，即所谓的“Module”, 模块。但是长期以来，JavaScript一直没有语言级别的模块语法。社区中产生了一些方法来解决这个问题，比如：

- AMD –最古老的模块系统之一，最初由require.js库实现。
- CommonJS –为Node.js服务器创建的模块系统。
- UMD –另一种模块系统，作为通用模块系统，与AMD和CommonJS兼容。

<br>

语言级模块系统于2015年出现在标准中，此后逐渐发展，现在已得到所有主要浏览器和Node.js的支持。也就是 ES Module

---

目前ESM模块化已经支持92%以上的浏览器，而且且作为 ECMA 标准，未来会有更多浏览器支持ECMA规范(抛弃ie)
<br>

![](https://cdn.jsdelivr.net/gh/Alanwell/img-base/兼容性.jpg)

---

> 当我们在使用模块开发时，其实就是在构建一张模块依赖关系图，当模块加载时，就会从入口文件开始，最终生成完整的模块实例图。

<br>

ESM的执行可以分为三个步骤：

1. 构建: 确定从哪里下载该模块文件、下载并将所有的文件解析为模块记录
2. 实例化: 将模块记录转换为一个模块实例，为所有的模块分配内存空间，依照导出、导入语句把模块指向对应的内存地址。
3. 运行：运行代码，将内存空间填充
   
<br>

```从上面实例化的过程可以看出，ESM使用实时绑定的模式，导出和导入的模块都指向相同的内存地址，也就是值引用。而CJS采用的是值拷贝，即所有导出值都是拷贝值。```

---
### 了解 ESBuild

Vite 针对 TypeScript 做了内置支持，并不需要额外去配置。在编译器方面，Vite 并没有采用官方的 tsc 来编译。而是采用了 ESBuild 这个工具。

ESBuild 是一个用Go语言实现，能够把 TypeScript,Jsx,Tsx 编译到原生JavaScript的一个工具。性能会比 tsc 好上很多。(单线程ES Build 性能大概会好个2,30倍，多线程可能会好到百倍以上 -- 基于16GRAM的6核 2019 MacBook Pro)。

---

### ESBuild 快在哪里？

首先了解一下 tsc 干了什么，tsc是一个 TypeScript 编译器，可以把TypeScript 编译成JavaScript， tsc 在把目标文件解析成 AST 之后，会进行两步操作:
1.  类型检查 
2. AST -> JavaScript 代码。

ESBuild 在把目标文件解析成 AST 之后，则只进行了其中一步操作： AST -> JavaSciript 代码。
这就是ESBuild 比 tsc 快的原因之一。换句话说， ESBuild 没有类型检查的功能。想要做到类型检查需要配合tsc 命令或者一些IDE插件去解决

---
### 基于ESM 的 HMR 热更新

目前所有的打包工具实现热更新的思路都大同小异：主要是通过WebSocket创建浏览器和服务器的通信监听文件的改变，当文件被修改时，服务端发送消息通知客户端修改相应的代码，客户端对应不同的文件进行不同的操作的更新。

- Webpack: 重新编译，请求变更后模块的代码，客户端重新加载

- Vite: 请求变更的模块，再重新加载

Vite 通过 chokidar 来监听文件系统的变更，只用对发生变更的模块重新加载， 只需要精确的使相关模块与其临近的 HMR边界连接失效即可，这样HMR 更新速度就不会因为应用体积的增加而变慢而 Webpack 还要经历一次打包构建。所以 HMR 场景下，Vite 表现也要好于 Webpack。

---
### 核心流程

Vite整个热更新过程可以分成四步

1. 创建一个websocket服务端和client文件，启动服务
2. 通过chokidar（ 一个封装 Node.js 监控文件系统文件变化功能的库）监听文件变更
3. 当代码变更后，服务端进行判断并推送到客户端
4. 客户端根据推送的信息执行不同操作的更新
---

整体流程图：

![](https://cdn.jsdelivr.net/gh/Alanwell/img-base/hmr.jpg)

---

### Vite 冷启动对比
<br>

![](https://cdn.jsdelivr.net/gh/Alanwell/img-base/冷启动.webp)

> 从左到右依次是: vue-cli3 + vue3 的demo, vite 1.0.0-rc + vue 3的demo， vue-cli3 + vue2的demo

<br>
从这个gif可以明显感受到vite比其他两个在启动速度上有明显的优势，vue-cli 3 启动Vue2大概需要5s左右，vue-cli3 启动Vue3需要4s左右，而vite 只需要1s 左右的时间

---

### Vite 生产环境对比

![](https://cdn.jsdelivr.net/gh/Alanwell/img-base/生产环境.webp)

> Vite是基于Rollup实现的生产环境构建，所以在生产环境构建时，Vite 与 基于webpack实现的vue-cli在构建时间上相比差距不大。

<br>
Vite 的生产环境构建可以简单理解为：默认配置了一些rollup的配置（比如： rollup-plugin-vue），然后通过 vite.config.js 来接收一些rollup的相关配置，直接传入到rollup 中参与构建


---
### Vite 使用

> use npm: version >= npm@6.1.0


`$ npm init vite-app <project-name>`
`$ cd <project-name>`
`$ npm install`
`$ npm run dev`


> use yarn: version >= yarn@1.0

`$ yarn create vite-app <project-name>`
`$ cd <project-name>`
`$ yarn`
`$ yarn dev`

---

![](https://cdn.jsdelivr.net/gh/Alanwell/img-base/vite-create-vue.png)