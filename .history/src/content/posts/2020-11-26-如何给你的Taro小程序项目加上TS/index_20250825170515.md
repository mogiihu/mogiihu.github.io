---
title: 如何给你的Taro小程序项目加上TS
published: 2020-11-25
description: ''
image: ''
tags: [小程序, 前端, Taro, Typescript]
category: 'Typescript'
draft: false 
lang: ''
---

## 前言

现在有越来越多的项目开始使用 ts，我们熟知的 React 、Ant Design 也已经全面转变为 ts。那么 ts 能给我们带来什么呢？

## 粗谈 TypeScript

很多人会好奇，ESLint 可以为我们提示代码不规范以及使用错误等问题，在有 ESLint 的情况下为什么还需要 TypeScript？

首先，ESLint 是用来检测代码规范的，并不能检测代码逻辑问题。

其次，我们都知道 JavaScript 是一种弱类型语言，并且是在运行时检查的动态类型。而 Typescript 相对于 Javascript 为我们带来了*静态类型*，它可以根据我们对变量的类型定义、以及自身的类型断言在编译期对代码进行类型检查。

简单介绍过后，下面我们开始进入正题，给 Taro 项目加上 TS。

## TS 快到项目里来

### 1. 新增包

首先是关于 npm 包的安装，这里我建议通过 `taro init` 选择 Typescript，创建内置 ts 环境的新项目。

下面是 ts 相关包：

```js
"@typescript-eslint/eslint-plugin": "^2.x",
// esLint插件，为TypeScript代码库提供lint规则
"@typescript-eslint/parser": "^2.x",
// 允许ESLint，对TypeScript代码进行整理
"typescript": "^3.7.0",
// 我们的主角 typescript
"@types/react": "^17.0.0",
// React的类型定义
"@types/react-dom": "^17.0.0",
// React-dom 的类型定义
"@types/webpack-env": "^1.13.6",
// webpack（模块API）的类型定义
"@types/wechat-miniprogram": "^3.1.0",
// 微信小程序的类型定义
```

`@types/**` 此类包，是为了让我们能在 ts 中使用第三方库的 API，关于这方面的介绍可以查看这篇文章：[在 typescript 中使用第三方库](https://mogiihu.github.io/posts/%E5%9C%A8Typescript%E4%B8%AD%E4%BD%BF%E7%94%A8%E7%AC%AC%E4%B8%89%E6%96%B9%E5%BA%93/)。

### 2. tsconfig.json

Typescript 和 ESLint 一样，都需要配置文件作为编译配置。`tsconfig.json` 就是用来配置编译 Typescript 的特定选项。

- `files` : 需要编译的单个文件列表，可以用相对路径和绝对路径来表示。
  ```js
  "files": [
      // 指定编译文件是src目录下的index.ts文件
      "scr/index.ts"
  ]
  ```
- `include` : 需要编译的文件或目录。
  ```js
  "include": [
       // 编译src目录下的所有文件，包括子目录
      "scr",
      // 只编译scr一级目录下的文件
      "scr/*",
      // 只编译scr二级目录下的文件
      "scr/*/*"
  ]
  ```
- `exclude` : 不需要编译的文件或目录，默认排除 `node_modules` 文件。

  ```js
    "exclude": ["dist", "src/assets"],
  ```

- `extends` : 从另一个配置文件里继承 ts 配置，所继承配置文件的 files，include 和 exclude 覆盖源配置文件的属性。

  ```js
    "extends": "./configs/base",
  ```

- `compileOnSave` : 可以让 IDE 在保存文件的时候根据 tsconfig.json 重新生成文件。

  ```js
    "compileOnSave": true,
  ```

- `compilerOptions` : 编译器选项类型有很多，下面只列出常用配置，在这里查看完整的[编译选项器](https://www.tslang.cn/docs/handbook/compiler-options.html)选项列表。

  ```js
    "compilerOptions": {
        "incremental": true, // TS编译器在第一次编译之后会生成一个存储编译信息的文件，第二次编译会在第一次的基础上进行增量编译，可以提高编译的速度
        "tsBuildInfoFile": "./buildFile", // 增量编译文件的存储位置
        "diagnostics": true, // 打印诊断信息
        "target": "ES2017", // 目标语言的版本
        "module": "CommonJS", // 生成代码的模板标准
        "outFile": "./app.js", // 将多个相互依赖的文件生成一个文件，可以用在AMD模块中，即开启时应设置"module": "AMD",
        "lib": ["DOM", "ES2015", "ScriptHost", "ES2019.Array"], // TS需要引用的库，即声明文件，es5 默认引用dom、es5、scripthost,如需要使用es的高级版本特性，通常都需要配置，如es8的数组新特性需要引入"ES2019.Array",
        "allowJS": true, // 允许编译器编译JS，JSX文件
        "checkJs": true, // 允许在JS文件中报错，通常与allowJS一起使用
        "outDir": "./dist", // 指定输出目录
        "rootDir": "./", // 指定输出文件目录(用于输出)，用于控制输出目录结构
        "declaration": true, // 生成声明文件，开启后会自动生成声明文件
        "declarationDir": "./file", // 指定生成声明文件存放目录
        "emitDeclarationOnly": true, // 只生成声明文件，而不会生成js文件
        "sourceMap": true, // 生成目标文件的sourceMap文件
        "inlineSourceMap": true, // 生成目标文件的inline SourceMap，inline SourceMap会包含在生成的js文件中
        "declarationMap": true, // 为声明文件生成sourceMap
        "typeRoots": [], // 声明文件目录，默认时node_modules/@types
        "types": [], // 加载的声明文件包
        "removeComments":true, // 删除注释
        "noEmit": true, // 不输出文件,即编译后不会生成任何js文件
        "noEmitOnError": true, // 发送错误时不输出任何文件
        "noEmitHelpers": true, // 不生成helper函数，减小体积，需要额外安装，常配合importHelpers一起使用
        "importHelpers": true, // 通过tslib引入helper函数，文件必须是模块
        "downlevelIteration": true, // 降级遍历器实现，如果目标源是es3/5，那么遍历器会有降级的实现
        "strict": true, // 开启所有严格的类型检查
        "alwaysStrict": true, // 在代码中注入'use strict'
        "noImplicitAny": true, // 不允许隐式的any类型
        "strictNullChecks": true, // 不允许把null、undefined赋值给其他类型的变量
        "strictFunctionTypes": true, // 不允许函数参数双向协变
        "strictPropertyInitialization": true, // 类的实例属性必须初始化
        "strictBindCallApply": true, // 严格的bind/call/apply检查
        "noImplicitThis": true, // 不允许this有隐式的any类型
        "noUnusedLocals": true, // 检查只声明、未使用的局部变量(只提示不报错)
        "noUnusedParameters": true, // 检查未使用的函数参数(只提示不报错)
        "noFallthroughCasesInSwitch": true, // 防止switch语句贯穿(即如果没有break语句后面不会执行)
        "noImplicitReturns": true, //每个分支都会有返回值
        "esModuleInterop": true, // 允许export=导出，由import from 导入
        "allowUmdGlobalAccess": true, // 允许在模块中全局变量的方式访问umd模块
        "moduleResolution": "node", // 模块解析策略，ts默认用node的解析策略，即相对的方式导入
        "baseUrl": "./", // 解析非相对模块的基地址，默认是当前目录
        "paths": { // 路径映射，相对于baseUrl
            "@components/*": ["./src/components/*"],
            "@util": ["./src/util/*"],
            "@assets/*": ["./src/assets/*"],
        },
        "rootDirs": ["src","out"], // 将多个目录放在一个虚拟目录下，用于运行时，即编译后引入文件的位置可能发生变化，这也设置可以虚拟src和out在同一个目录下，不用再去改变路径也不会报错
        "listEmittedFiles": true, // 打印输出文件
        "listFiles": true, // 打印编译的文件(包括引用的声明文件)
        "jsx": 'React', // 在 .tsx文件里支持JSX
    }
  ```

## 常见报错处理

### wx 报错问题

解决方案有两种

1. 安装定义文件
   ```
   npm install @types/wechat-miniprogram
   ```
2. 安装独立的 npm 包

   ```
   npm install miniprogram-api-typings
   ```

   安装后需要在使用的地方或者根目录手动导入：
   `import 'miniprogram-api-typings';`

   或者在 tsconfig.json 中添加一下配置：

   ```ts
   "compilerOptions": {
       "types": ["miniprogram-api-typings"]
   }
   ```

### promise 报错问题

promise 报错是由于 tsconfig.json 中的 js 目标版本问题，将目标版本改为 ES2016 及以上版本即可解决：

```js
"compilerOptions": {
    "target": "ES2017",
}
```
