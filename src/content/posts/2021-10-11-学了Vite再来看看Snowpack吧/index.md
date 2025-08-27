---
title: 学了Vite再来看看Snowpack吧
published: 2021-10-11
description: ''
image: ''
tags: [基建, 前端, Snowpack]
category: '基建'
draft: false 
lang: ''
---

# 什么是 Snowpack？

看下官网的介绍：

**Snowpack is a lightning-fast frontend build tool, designed for the modern web.** It is an alternative to heavier, more complex bundlers like webpack or Parcel in your development workflow. Snowpack leverages JavaScript's native module system ([known as ESM](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)) to avoid unnecessary work and stay fast no matter how big your project grows.

Once you try it, it's impossible to go back to anything else.

****

**Snowpack 是一款闪电般快速的前端构建工具，专为现代 Web 设计。** 它是开发工作流程中更重、更复杂的打包程序（如 webpack 或 Parcel）的替代品。Snowpack 利用 JavaScript 的原生模块系统（[称为 ESM](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)）来避免不必要的工作，无论您的项目有多大，都能保持快速。

一旦你尝试过，就不可能回到其他任何事情上。

从这段介绍可以看出，Snowpack 的特点就是——快！

# 本文目标

-   基于 Snowpack 搭建 react 项目，包括路由、UI等功能；
-   项目的扩展功能，如 typescript、less、eslint、stylelint、prettier；

<!---->

-   围绕项目的相关配置内容，如环境变量、路径别名、打包路径等。

\


# 创建 Snowpack 项目

## 初始化

通过以下命令创建 Snowpack 项目。

```
npx create-snowpack-app react-snowpack --template @snowpack/app-template-minimal
```

创建完成后可以通过以下两个命令进入该目录，进行启动。

```
cd react-snowpack
npm run start
```

运行完后，你就可以看到 Snowpack 启动的项目了。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1707a72e917d4c42adf7107d3559b4d3~tplv-k3u1fbpfcp-zoom-1.image)

打开这个项目你会发现里面空空如也，别急，接下来我们为它添加上 React。

## React 的第一个页面

运行以下命令进行安装。

```
npm install react react-dom react-router-dom --save
```

将 React logo 图片放置在 `src/assets/images` 下。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b13c2882bf674360b3f2929555e83e1e~tplv-k3u1fbpfcp-zoom-1.image)

在根目录下创建文件夹 `src`，`src` 内创建 `App.jsx` 文件，这将是我们的第一个页面，`App.jsx` 的内容如下。

```
import React, { useState, useEffect } from "react";
import logo from "./assets/images/logo.png";
import "./App.css";

function App() {
		const [count, setCount] = useState(0);
  
  	useEffect(() => {
    		const timer = setTimeout(() => setCount(count + 1), 1000);
    		return () => clearTimeout(timer);
  	}, [count, setCount]);
  
  	return (
    		<div className="App">
      			<header className="App-header">
              	<img src={logo} className="App-logo" alt="logo" />
        				<h1>Hello snowpack!</h1>
       				  <p>
          				Page has been open for <code>{count}</code> seconds.
       				  </p>
      			</header>
   			</div>
  );
}

export default App;
```

创建 `src/App.css` ，并添加以下内容：

```
.App {
  	text-align: center;
}

.App p {
  	margin: 0.4rem;
}

.App-logo {
  	height: 40vmin;
}

@media (prefers-reduced-motion: no-preference) {
  	.App-logo {
    	animation: App-logo-spin infinite 20s linear;
  	}
}

.App-header {
    background-color: #282c34;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    font-size: calc(10px + 2vmin);
    color: white;
}

@keyframes App-logo-spin {
    from {
      	transform: rotate(0deg);
    }
    to {
      	transform: rotate(360deg);
    }
}
```

打开 index.html 文件，添加一个空的 `<div>` 标签作为标记你想要用 React 显示内容的位置。

```
<div id="root"></div>
```

删除根目录下的 `index.js` 文件，创建 `index.jsx`，用来渲染 React Dom。

```
import React from "react";
import ReactDOM from "react-dom";
import APP from "./src/App";
ReactDOM.render(
  	<React.StrictMode>
    		<APP />
  	</React.StrictMode>,
  	document.getElementById("root")
);
```

等待页面自动刷新后，就可以看到我们创建的第一个页面了。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3bda11788df7400d990ef19629c863af~tplv-k3u1fbpfcp-zoom-1.image)

## 添加路由

在单页面项目中路由是必不可少的，下面我们来添加路由。

分别创建 `src/pages/hello/index.jsx` 文件和 `src/pages/snowpack/index.jsx` 文件，作为路由跳转的测试页面。内容如下：

```
// hello/index.jsx
import React from "react";

function Hello() {
  	return <div className="App">I'm Hello, How are u?</div>;
}

export default Hello;
```

```
// snowpack/index.jsx
import React from "react";

function Snowpack() {
  	return <div className="App">I'm Snowpack, I'm fine thank you, and you?</div>;
}

export default Snowpack;
```

将 `App.jsx` 和 `APP.css` 移到 src/home 下，并更改文件名， `App.jsx` -> `index.jsx` ， `APP.css` -> `index.css` ，更改 `index.jsx` 的组件名为 `Home`。

在 `src` 下重新创建 `APP.jsx` 放置路由内容。

```
import React from "react";
import { HashRouter, Switch, Route } from "react-router-dom";
import Index from "./pages/home/index.jsx";
import Hello from "./pages/hello/index.jsx";
import Snowpack from "./pages/snowpack/index.jsx";

function App() {
  	return (
    		<div className="index">
            <HashRouter>
                <Switch>
                    <Route exact path="/" component={Index} />
                    <Route exact path="/hello" component={Hello} />
                    <Route exact path="/snowpack" component={Snowpack} />
                </Switch>
            </HashRouter>
    		</div>
  	);
}
```

在 `pages/home/index` 下添加路由跳转。

```
import React, { useState, useEffect } from 'react';
import { Link } from 'react-router-dom';
import logo from '../../assets/images/logo.svg';
import './index.css';

function Home() {
    const [count, setCount] = useState(0);

    useEffect(() => {
        const timer = setTimeout(() => setCount(count + 1), 1000);
        return () => clearTimeout(timer);
    }, [count, setCount]);

    return (
        <div className="App">
            <header className="App-header">
                <nav>
                    <ul>
                        <li>
                            <Link to="/hello">hello</Link>
                        </li>
                        <li>
                            <Link to="/snowpack">snowpack</Link>
                        </li>
                    </ul>
                </nav>
                <img src={logo} className="App-logo" alt="logo" />
                <h1>Hello snowpack!</h1>
                <p>
                    Page has been open for <code>{count}</code> seconds.
                </p>
            </header>
        </div>
    );
}

export default Home;
```

现在在页面中出现了跳转链接，点击链接就可以实现跳转了！

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ef8b8b2501245cf98dca1987cb19700~tplv-k3u1fbpfcp-zoom-1.image)跳转效果：![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca17c31a890d46c086dbbcec93450ba7~tplv-k3u1fbpfcp-zoom-1.image)
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e3086a4e3c747cf96429f7db0c6a1de~tplv-k3u1fbpfcp-zoom-1.image)
## Antd

安装以下包，`@ant-design/icons` 为 `antd icon` 依赖库， `dayjs` 为 `antd` 内部组件依赖库。

```
npm install antd @ant-design/icons dayjs --save 
```

然后在 `App.jsx` 中引入 `import "antd/dist/antd.css";` 所需css，就可以直接使用相关组件。

# 工程化配置

基本的项目建好了，下面我们开始工程化相关配置。

## Less

安装 `less`。

```
npm install less --save-dev
```

安装 `snowpack-plugin-less`，使 `snowpack` 可以对 `less` 进行识别并转换为 `css`。

```
npm install snowpack-plugin-less --save-dev
```

然后在 `snowpack.config.mjs` 中添加 `snowpack-plugin-less` 插件。

```
// snowpack.config.mjs
export default {
  	plugins: ['snowpack-plugin-less']
}
```

## Eslint

安装 `eslint` 。

```
npm install eslint --save-dev
```

安装 eslint 包后运行 `eslint --init` ，下图为我的安装选项。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec045c964308413bb71119c41d493859~tplv-k3u1fbpfcp-zoom-1.image)

安装完成后会在项目中自动创建，`.eslintrc`文件，使用 hooks 的同学还需要额外安装 `eslint-plugin-react-hooks` 包，以便 eslint 可以执行 hooks 的相关规则。

下面是推荐配置：

```
module.exports = {
    extends: [
        'eslint:recommended',
        'plugin:react/recommended',
        'plugin:react-hooks/recommended',
        'standard',
        'prettier',
    ],
    env: {
        browser: true,
        commonjs: true,
        es6: true
    },
    parser: '@typescript-eslint/parser',
    parserOptions: {
        ecmaFeatures: {
            jsx: true,
            modules: true
        },
        sourceType: 'module',
        ecmaVersion: 6
    },
    plugins: ['react', '@typescript-eslint'],
    settings: {
        'import/ignore': ['node_modules'],
        react: {
            version: 'latest'
        }
    },
    rules: {
        quotes: [2, 'single'],
        'no-console': 0,
        'no-debugger': 1,
        'no-var': 1,
        semi: ['error', 'always'],
        'no-irregular-whitespace': 0,
        'no-trailing-spaces': 1,
        'eol-last': 0,
        'no-unused-vars': [
            1,
            {
                vars: 'all',
                args: 'after-used'
            }
        ],
        'no-prototype-builtins': 0,
        'no-case-declarations': 0,
        'no-underscore-dangle': 0,
        'no-alert': 2,
        'no-lone-blocks': 0,
        'no-class-assign': 2,
        'no-cond-assign': 2,
        'no-const-assign': 2,
        'no-delete-var': 2,
        'no-dupe-keys': 2,
        'use-isnan': 2,
        'no-duplicate-case': 2,
        'no-dupe-args': 2,
        'no-empty': 2,
        'no-func-assign': 2,
        'no-invalid-this': 0,
        'no-redeclare': 2,
        'no-spaced-func': 2,
        'no-this-before-super': 0,
        'no-undef': 2,
        'no-return-assign': 0,
        'no-script-url': 2,
        'no-use-before-define': 0,
        'no-extra-boolean-cast': 0,
        'no-unreachable': 1,
        'comma-dangle': 2,
        'no-mixed-spaces-and-tabs': 2,
        'prefer-arrow-callback': 0,
        'arrow-parens': 0,
        'arrow-spacing': 0,
        camelcase: 0,
        'jsx-quotes': [1, 'prefer-double'],
        'react/display-name': 0,
        'react/forbid-prop-types': [
            2,
            {
                forbid: ['any']
            }
        ],
        'react/jsx-boolean-value': 0,
        'react/jsx-closing-bracket-location': 1,
        'react/jsx-curly-spacing': [
            2,
            {
                when: 'never',
                children: true
            }
        ],
        'react/jsx-indent': ['error', 4],
        'react/jsx-key': 2,
        'react/jsx-no-bind': 0,
        'react/jsx-no-duplicate-props': 2,
        'react/jsx-no-literals': 0,
        'react/jsx-no-undef': 1,
        'react/jsx-pascal-case': 0,
        'react/jsx-sort-props': 0,
        'react/jsx-uses-react': 1,
        'react/jsx-uses-vars': 2,
        'react/no-danger': 0,
        'react/no-did-mount-set-state': 0,
        'react/no-did-update-set-state': 0,
        'react/no-direct-mutation-state': 2,
        'react/no-multi-comp': 0,
        'react/no-set-state': 0,
        'react/no-unknown-property': 2,
        'react/prefer-es6-class': 2,
        'react/prop-types': 0,
        'react/react-in-jsx-scope': 0,
        'react/self-closing-comp': 0,
        'react/sort-comp': 0,
        'react/no-array-index-key': 0,
        'react/no-deprecated': 1,
        'react/jsx-equals-spacing': 2
    }
};
```

配置成功后很多文件已经变红了，是不是有些同学已经犯强迫症，想要手动修改这些问题了呢。不要急，后面我们会安装 `prettier` ，实现一键整理代码格式。

## Stylelint

有了 `eslint` 自然 `stylelint` 也不能少，`eslint` 帮我们检测js的代码格式，`stylelint` 则帮我们检测 `css` 的代码格式。

首先安装 `stylelint` 相关包，然后需要在项目中创建 `stylelint.config.js` 用来存放 `stylelint` 相关规则。

```
npm install stylelint stylelint-config-standard --save-dev 
echo {}> stylelint.config.js
```

下面是推荐配置：

```
module.exports = {
    extends: ['stylelint-config-standard', 'stylelint-config-prettier'],
    ignoreFiles: [
        '**/*.ts',
        '**/*.tsx',
        '**/*.png',
        '**/*.jpg',
        '**/*.jpeg',
        '**/*.gif',
        '**/*.mp3',
        '**/*.json'
    ],
    rules: {
        'at-rule-no-unknown': [
            true,
            {
                ignoreAtRules: ['extends', 'ignores']
            }
        ],
        indentation: 4,
        'number-leading-zero': null,
        'unit-allowed-list': ['em', 'rem', 's', 'px', 'deg', 'all', 'vh', 'vw', '%'],
        'no-eol-whitespace': [
            true,
            {
                ignore: 'empty-lines'
            }
        ],
        'declaration-block-trailing-semicolon': 'always',
        'selector-pseudo-class-no-unknown': [
            true,
            {
                ignorePseudoClasses: ['global']
            }
        ],
        'block-closing-brace-newline-after': 'always',
        'declaration-block-semicolon-newline-after': 'always',
        'no-descending-specificity': null,
        'selector-list-comma-newline-after': 'always',
        'selector-pseudo-element-colon-notation': 'single'
    }
};
```

## Prettier

接下来就到了每个前端开发必备的 `prettier`，有了它再也不用手动整理格式，它可以帮助我们一键整理代码格式。

安装 `prettier`，

```
npm install prettier  --save-dev
```

因为项目中还用到了 `eslint`，所以需要安装 `eslint-config-prettier` ，使 `eslint`可以和 `prettier`相互配合，它可以关闭与 `prettier` 冲突的 `eslint` 规则。

然后通过 ` echo {}> prettier.config.js  `创建 `prettier` 配置文件。

下面是推荐配置：

```
module.exports = {
    // 一行最多 100 字符
    printWidth: 100,
    // 使用 4 个空格缩进
    tabWidth: 4,
    // 不使用缩进符，而使用空格
    useTabs: false,
    // 行尾需要有分号
    semi: true,
    // 使用单引号
    singleQuote: true,
    // 对象的 key 仅在必要时用引号
    quoteProps: 'as-needed',
    // jsx 不使用单引号，而使用双引号
    jsxSingleQuote: false,
    // 末尾不需要逗号
    trailingComma: 'none',
    // 大括号内的首尾需要空格
    bracketSpacing: true,
    // jsx 标签的反尖括号需要换行
    jsxBracketSameLine: false,
    // 箭头函数，只有一个参数的时候，也需要括号
    arrowParens: 'avoid',
    // 每个文件格式化的范围是文件的全部内容
    rangeStart: 0,
    rangeEnd: Infinity,
    // 不需要写文件开头的 @prettier
    requirePragma: false,
    // 不需要自动在文件开头插入 @prettier
    insertPragma: false,
    // 使用默认的折行标准
    proseWrap: 'preserve',
    // 根据显示样式决定 html 要不要折行
    htmlWhitespaceSensitivity: 'css',
    // 换行符使用 lf
    endOfLine: 'lf'
};
```

然后在命令行中运行以下命令就可以将全部文件进行格式化。

```
npx prettier --write .
```

如果希望在 `vscode` 中通过保存一键格式化，需要安装 [Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) 插件，根据[教程](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)对 vscode 配置进行修改。

## Typescript

通过以下命令安装 `typescript` 以及 `react`、`react-dom` 的类型定义。

```
npm install typescript --save-dev
npm install @types/react --save-dev
npm install @types/react-dom --save-dev
npm install @typescript-eslint/eslint-plugin --save-dev
npm install @typescript-eslint/parser --save-dev
```

如果需要在编译时进行打包检测，还需要安装以下包。

```
npm install @snowpack/plugin-typescript --save-dev
```

并在 `snowpack.config.mjs` 的 `plugins` 中进行配置：

```
// snowpack.config.mjs
export default {
  	plugins: ['@snowpack/plugin-typescript']
}
```

通过 ` echo {}> tsconfig.json  `创建 `tsconfig.json` 并进行配置。

以下为推荐配置：

```
{
    "compilerOptions": {
        "emitDecoratorMetadata": true,
        "experimentalDecorators": true,
        "target": "ESNext",
        "allowSyntheticDefaultImports": true,
        "strict": true,
        "forceConsistentCasingInFileNames": true,
        "allowJs": true,
        "outDir": "./dist/",
        "esModuleInterop": true,
        "noImplicitAny": false,
        "sourceMap": true,
        "module": "esnext",
        "moduleResolution": "node",
        "isolatedModules": true,
        "importHelpers": true,
        "lib": ["esnext", "dom", "dom.iterable"],
        "skipLibCheck": false,
        "jsx": "react",
        "baseUrl": "./src",
        "paths": {
              "@src/*": ["*"],
              "@assets/*": ["assets/*"],
              "@components/*": ["components/*"],
              "@pages/*": ["pages/*"],
              "@utils/*": ["utils/*"],
              "@servers/*": ["servers/*"],
              "@actions/*": ["actions/*"],
              "@config": ["config"],
              "@routeConfig": ["routeConfig"],
              "@request": ["request"]
          }
    },
    "include": ["./src/**/*", "./declaration.d.ts"],
    "exclude": ["node_modules"]
}
```

`tsconfig.json` 中 `paths` 为 ts 所支持的路径别名，除了在 `tsconfig.json` 中需要设置，还需要在 `snowpack.config.mjs` 中进行配置，以便于让 `snowpack` 打包时可以转换成对应路径，关于项目配置请看 [snowpack.config.mjs 相关配置-路径别名配置] 相关章节。

## pre-commit

安装 `husky` 和 `list-staged`。

```
npm install husky -save-dev
npm install lint-staged -save-dev
```

在 `package.json` 中添加一下配置。

```
"scripts": {
				...,
        "lint:jsx": "eslint --ext .jsx,.js src",
        "lint:css": "stylelint --aei .less .css src",
        "precommit": "lint-staged",
        "precommit-msg": "echo 'Pre-commit checks...' && exit 0"
},
"husky": {
		"hooks": {
				"pre-commit": "npm run lint-staged"
		}
},
"lint-staged": {
    "*.{js,jsx,ts,tsx}": [
        "eslint --fix",
        "prettier --write"
    ],
    "*.{css,less}": [
        "stylelint --fix",
        "prettier --write"
    ]
}
```

# snowpack.config.mjs 相关配置

## 打包路径配置

首先我们先整理下项目的公共资源文件，一般来说会把项目的 `index.html` 及 `index.css` 看做静态资源放置在 `/public` 目录下。如下图所示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36d4ebefe1634b10bc513a9c86895196~tplv-k3u1fbpfcp-zoom-1.image)

移动过后注意要修改 `index.html` 中 css 及 js 的引入路径。

按上面步骤操作完成后，项目的基本结构和内容就已经配置好了，现在我们在命令行运行 `npm run build` 看一下打包后的文件结构是怎样的。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ee7dd70958f493ab824796347503382~tplv-k3u1fbpfcp-zoom-1.image)

打包结束后在项目根目录下生成了 `build` 文件夹，展开 `build` 可以看到，`public` 中的文件被打包至 `build/public` 文件中，但通常我们期望 `index.html` 能在 `build` 的根目录下。可以通过以下配置将 `public` 挂载到 `build`根目录下。

```
// snowpack.config.mjs
export default {
  	mount: {
				public: '/',
		}
}
```

并且打包后的文件不只有我们的业务代码，还有项目的配置代码，如 `package.json`、`perttier.config.js`、`README.md` 等。这些代码是我们不需要的，并且会占用少量空间，可以通过以下配置设置只打包 `src` 目录下的文件。

```
// snowpack.config.mjs
export default {
  	mount: {
				src: '/src',
		}
}
```

## 打包输出目录配置

```
// snowpack.config.mjs
export default {
  	out: 'dist'
};
```

## 路径别名配置

通过 `alias` 可配置路径缩写别名，及 node 包的引入别名。

```
// snowpack.config.mjs
export default {
    alias: {
        // Type 1: node 包 引入别名
        lodash: 'lodash-es',
        // Type 2: 本地文件引入别名
        '@src': './src',
        '@assets': './src/assets',
        '@components': './src/components',
        '@pages': './src/pages',
        '@utils': './src/utils/',
        '@servers': './src/servers',
        '@actions': './src/actions',
        '@config': './src/config.ts',
        '@routeConfig': './src/routeConfig.tsx',
        '@request': './src/request.ts'
    },
};
```

## 打包环境区分

通常我们在打包时，希望通过打包命令区分包所对应环境。\
首先需要在 `package.json` 的 `scripts` 配置中，给对应 `script` 命令添加环境所对应值。

```
"scripts": {
    "start": "NODE_ENV=development snowpack dev",
    "build:test": "NODE_ENV=test snowpack build",
    "build:prod": "NODE_ENV=production snowpack build",
},
```

如果需要在 `snowpack.config.mjs` 中获取环境，通过 `process.env.NODE_ENV` 可直接获取到，如下：

```
// snowpack.config.mjs
export default {
  	plugins: (() => {
        const plugin = ['snowpack-plugin-less'];
        if (process.env.NODE_ENV === 'development') {
            plugin.push('@snowpack/plugin-typescript');
        }
        return plugin;
		})(),
}
```

在js文件中则需要通过 `import.meta.env` 中的 `NODE_ENV` 获取，如下：

```
const { NODE_ENV } = import.meta.env;
```

## 设置全局变量

有时我们还需要在设置一些内置的全局变量，供 html 及 js 文件使用，在 `snowpack.config.mjs` 的 `env` 配置中设置即可，如下：

```
// snowpack.config.mjs
export default {
  	env: {
        ENVIRONMENT: 'test'
    },
}
```

通用在js文件中需要通过 `import.meta.env` 中获取：

```
const { ENVIRONMENT } = import.meta.env;
```

在 HTML 中则可以通过 `%ENVIRONMENT%` 直接使用：

```
<script>
  	console.log('%ENVIRONMENT%');
</script>
```

在 devtools 中可以看到该部分内容变成了下面这样。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/261bebe9ba2c48388f1b971c5b3b682c~tplv-k3u1fbpfcp-zoom-1.image)

# 问题&注意事项

1.  snowpack 运行时页面报错 **Uncaught ReferenceError: require is not defined.**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44dca969c36c4a2ab085a80e0608e28a~tplv-k3u1fbpfcp-zoom-1.image)

报错原因是由于 snowpack 只支持 ESM 标准代码，不能解析 commonjs。

2.  snowpack **无法解析** `cnpm` 安装的包，所以在安装包后启动报错，或者发现包不生效，可以删除 `node_modules`，用 `npm` 或 `yarn` 安装。
2.  npm start 时报错

```
[18:30:15] [esinstall:xmlhttprequest-ssl] /Users/huxiaomiao/pivos/react-snowpack3/node_modules/xmlhttprequest-ssl/lib/XMLHttpRequest.js
   Module "fs" (Node.js built-in) is not available in the browser. Run Snowpack with --polyfill-node to fix.
[18:30:15] [esinstall:xmlhttprequest-ssl] /Users/huxiaomiao/pivos/react-snowpack3/node_modules/xmlhttprequest-ssl/lib/XMLHttpRequest.js
   Module "url" (Node.js built-in) is not available in the browser. Run Snowpack with --polyfill-node to fix.
[18:30:15] [esinstall:xmlhttprequest-ssl] /Users/huxiaomiao/pivos/react-snowpack3/node_modules/xmlhttprequest-ssl/lib/XMLHttpRequest.js
   Module "https" (Node.js built-in) is not available in the browser. Run Snowpack with --polyfill-node to fix.
[18:30:15] [esinstall:xmlhttprequest-ssl] /Users/huxiaomiao/pivos/react-snowpack3/node_modules/xmlhttprequest-ssl/lib/XMLHttpRequest.js
   Module "child_process" (Node.js built-in) is not available in the browser. Run Snowpack with --polyfill-node to fix.
[18:30:15] [esinstall:xmlhttprequest-ssl] /Users/huxiaomiao/pivos/react-snowpack3/node_modules/xmlhttprequest-ssl/lib/XMLHttpRequest.js
   Module "url" (Node.js built-in) is not available in the browser. Run Snowpack with --polyfill-node to fix.
[18:30:15] [esinstall:xmlhttprequest-ssl] /Users/huxiaomiao/pivos/react-snowpack3/node_modules/xmlhttprequest-ssl/lib/XMLHttpRequest.js
   Module "fs" (Node.js built-in) is not available in the browser. Run Snowpack with --polyfill-node to fix.
[18:30:15] [esinstall:xmlhttprequest-ssl] /Users/huxiaomiao/pivos/react-snowpack3/node_modules/xmlhttprequest-ssl/lib/XMLHttpRequest.js
   Module "http" (Node.js built-in) is not available in the browser. Run Snowpack with --polyfill-node to fix.
[18:30:15] [esinstall:xmlhttprequest-ssl] /Users/huxiaomiao/pivos/react-snowpack3/node_modules/xmlhttprequest-ssl/lib/XMLHttpRequest.js
   Module "http" (Node.js built-in) is not available in the browser. Run Snowpack with --polyfill-node to fix.
[18:30:15] [esinstall:xmlhttprequest-ssl] /Users/huxiaomiao/pivos/react-snowpack3/node_modules/xmlhttprequest-ssl/lib/XMLHttpRequest.js
   Module "https" (Node.js built-in) is not available in the browser. Run Snowpack with --polyfill-node to fix.
[18:30:15] [esinstall:xmlhttprequest-ssl] /Users/huxiaomiao/pivos/react-snowpack3/node_modules/xmlhttprequest-ssl/lib/XMLHttpRequest.js
   Module "child_process" (Node.js built-in) is not available in the browser. Run Snowpack with --polyfill-node to fix.
[18:30:15] [esinstall:xmlhttprequest-ssl] url?commonjs-external
   Module "url" (Node.js built-in) is not available in the browser. Run Snowpack with --polyfill-node to fix.
[18:30:15] [esinstall:xmlhttprequest-ssl] https?commonjs-external
   Module "https" (Node.js built-in) is not available in the browser. Run Snowpack with --polyfill-node to fix.
[18:30:15] [esinstall:xmlhttprequest-ssl] http?commonjs-external
   Module "http" (Node.js built-in) is not available in the browser. Run Snowpack with --polyfill-node to fix.
[18:30:15] [esinstall:xmlhttprequest-ssl] fs?commonjs-external
   Module "fs" (Node.js built-in) is not available in the browser. Run Snowpack with --polyfill-node to fix.
[18:30:15] [esinstall:xmlhttprequest-ssl] child_process?commonjs-external
   Module "child_process" (Node.js built-in) is not available in the browser. Run Snowpack with --polyfill-node to fix.
[18:30:15] [snowpack] Install failed for xmlhttprequest-ssl.
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c1dff51ac8f14ee2b9e223cb2f2b5319~tplv-k3u1fbpfcp-zoom-1.image)

解决方式：

<https://www.snowpack.dev/reference/configuration#packageoptionspolyfillnode>

# 最后

经过这些一个基本的项目框架就基本完成了，小伙伴们快去动手实践下吧，有问题欢迎在评论区探讨。

下面为我搭建的简单项目框架，想体验的朋友们可以直接 clone 下来玩哦~

<https://github.com/mogiihu/snowpack-react>
