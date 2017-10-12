# 我们如何组织 React + Redux 项目结构

最近一年多，在公司的很多项目上都采用了 React/Redux 技术栈，对于 React 或是 Redux 本身没有多大的问题，但是有一个问题一直困扰着我们，那就是**如何组织项目的代码结构**。

最近工作在[一个全新的前端项目上](https://readercloud.active.com)，前端仍然是 React + Redux，与后端通过 API Service/Web Socket 交互，也就是说 Routes, View, States 以及业务逻辑等等全部放在前端处理的，这个对于活跃网络前端来说，是一个全新的尝试，所以重新思考了一下关于如何项目的代码结构。

对于如何组织项目目录结构，特别是针对 React Web App，有一些基本要求是必须达到的：

- 对于任何大小的项目必须保持可扩展性和灵活性
- 很容易知道哪个文件应该放在哪个目录里面
- 很容易在各个代码源文件之间导航（无论是打开一个已知/未知文件，又或是在编辑器中自由的在文件 Tab 之间切换）
- 代码重用很容易

下面是我们目前所采用的一个项目目录结构：

```JavaScript
- api
- build
- scripts
- i18n
  - en_US.json
  _ zh_CN.json
  ...
- src
  - i18n
  - routes
    - Home
      - components
      - containers
      - ducks
        index.js
    - Signin
      - index.js
      - ...
    - Signup
      - index.js
      - ...
  - shared
    - components
      - Header
        - HeaderDropdown
          - HeaderDropdown.js
          - HeaderDropdown.less
          - ...
        - Header.js
        - Header.less
        - ...
    - constants
    - containers
      - Header
        - HeaderContainer.js
    - middlewares
    - modules
      - Signin
        - components
          - Signin.js
          - Signin.less
        - containers
          - SigninContainer.js
      - Signup
        - ...
    - utils
      - formateNumber.js
      - trim.js
      - ...
      - index.js
  - styels
  - tempest.js
- test
- .babelrc
- .eslintrc
- .eslintignore
- .gitignore
- commitlint.config.js
- jest.config.js
- package.json
- README.md
- yarn.lock
```

## **api**

**api** 目录是一个基于 [express.js](http://expressjs.com/) 搭建的一个本地 API Mock server，可以通过 `yarn api` 或是 `yarn start` 跑起来。

默认本地开发调试时，所有的 API 请求都会指向它。另外，我们也加入了对 [faker.js](https://github.com/marak/Faker.js/) 的支持，用于帮助我们生成大量的模拟数据，尽量接近真实的用户环境。

## **build**

**build** 目录包含构建脚本和开发环境的基础配置。目前是使用 [Webpack](http://webpack.js.org/) 来作为 **module bundler**。

```
build
 - config.js
 - webpack.base.js
 - webpack.dev.js
 - webpack.prod.js
 - webpack.eslint.js
 - ...
```

## **scripts**

**scripts** 目录主要包含部署，发布脚本，以及一些工具脚本，如编译打包 **i18n** 文件。

```
scripts
  - release.js
  - compileI18nProperties.js
  - deployer.js
  - bundleAnalyzer.js
  - ...
```

有关发布流程，请参见[前端包发布](/active/2017-08-25-front-end-engineering-in-active.html#发布)

## **i18n**

**i18n** 目录包含编译后的 **i18n** 文件，是以 `JSON` 格式存在的，每个 `JSON` 文件中的国际化文本消息是以键值对的形式存在。

```
i18n
  - de_DE.json
  - en_GB.json
  - en_US.json
  - zh_CN.json
  - ...
```

## **src**

**src** 目录包含所有项目源代码，也就是会包含所有会被 Webpack 打包并最终发送到浏览端的代码。这样做的好处显示易见，代码更好组织管理，同时也可以让一些第三方工具（比如 Babel，Webpack）只需要处理 **src** 目录。

```
src
  - i18n
  - routes
  - shared
  - static
  - styles
  - index.html
  - index.js
```

### **i18n**

**i18n** 目录包含编译前的 **i18n** 源文件，是以 **.properties** 格式存在的。

### **routes**

**routes** 目录包含项目中每个路由的定义，并且路由是可以嵌套定义的。

一个路由目录包含：

- 必须包含一个入口文件`index.js`，由它返回与路由相对应的组件
- **components**, **containers**, **ducks(redux)**, **styles**等等目录
- 嵌套的子路由，如 **events -> events/:id**

从上面看出，每个路由是由 `container` 和 `components` 来构建视图，而数据和业务逻辑则交给 **reducers**, **actions** 和 **selectors** ，另外，我们采用了 [Ducks: Redux Reducer Bundles](https://github.com/erikras/ducks-modular-redux) 模式，我们会把这三块放在 **ducks** 目录中。

这样做的好处：

- 一个路由就相当于一个物理级页面，它有自己的 URL
- 通过路由来切割页面级粒度

在每个路由页面中，我们可以在纵向和横向不同的维度上，再进一步的抽象为不同的业务模块。

具体请参考[分形项目结构](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure)

#### **index.js**

每个路由必须包含一个入口文件 `index.js` ，它主要用于加载与路由相对应的路由组件。

我们希望可以把整个大的 Bundle 拆分成可以在之后异步下载的一个一个小的 Chunk，每个 Chunk 对应一个页面路由。如此一来，就允许我们首先提供粒度最小的引导 Bundle，并在稍后根据需要再异步地加载其它 Chuck。[Webpack在这方面提供了相对应的代码分割支持](https://webpack.js.org/guides/code-splitting/)。

Webpack 支持在模块中通过函数调用实现代码切割，它把 [`import()`](https://github.com/tc39/proposal-dynamic-import) 作为一个代码分离点(code split-point)，并把引入的模块作为一个单独的 Chunk。`import()` 将模块名字作为参数并返回一个 Promoise 对象，即 `import(name) -> Promise`。

```JavaScript
import asyncComponent from 'tempest.js/components/asyncComponent'

import './index.less'

const Signin = asyncComponent(
  () => import('./containers/SigninContainer').then(module => module.default),
  { name: 'Signin' },
)

export default Signin
```

> 从Webpack@2.4.0开始，它引入一个叫[**magic comments**](https://webpack.js.org/api/module-methods/#import-)的新Feature，它让你可以为每个异步Chunk命名，大大简化了资源的管理和SSR。

### **shared**

**shared** 目录主要包含公共的 **components**, **containers**, **modules(redux)**，**middlewares**，以及一些工具函数。

可能大家会很奇怪这里为什么会有这些公共的目录呢？不是应该放在对应的路由目录下面吗？

这是因为我们发现，一个路由常常会跟许多 **domain data** 打交道，比如说，一个 Events 列表页面，不仅需要 `events`，还需要 `user` 和 `filter` 数据。从另外一个方面来讲，一个 **domain data** 也可以同时被多个 `routes` 共享，而它本身可能并不是一个路由。所以我们会这些公共部分提取出来，达到代码重用的目的。

另外，**modules** 目录主要包含项目的模块。组件是构建应用的最小单元，而模块是基于组件构建的，每个页面又是基于模块的。当一个功能足够大且复杂时，我们会把它放置在 **modules** 目录下面，以达到重用的目的，如 `Signin`，`Signup`等等模块。

### **styles**

**styles** 目录包含整个项目的公用样式结构，如 `Layout`, `variables`, `mixins`。根据就近原则，每个组件所对应的样式都是放在相对应的组件所在目录的。

### **static**

**static** 目录主要包含静态资源文件，如 **favicon.ico**。

### `index.js`

`index.js` 是整个项目的 JavaScript 入口文件，它主要用于初始化 React + Redux Web App，并启动整个Web App。

![index.js](/images/react-redux-boilerplate-index-js.png)

### `index.html`

`index.html` 是整个项目的入口，所以的页面浏览都会导向这个页面，它会负责去加载项目对应的 JS/CSS 资源，它是由
 [html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin)协助生成。

### **tempest.js**

[**tempest.js**](https://gitlab.dev.activenetwork.com/fee/tempest.js) 是我们内部基于我们的最佳实践创建一个基于 React + Redux 的框架。它默认提供了一套 **API**，从而帮助我们更高效便捷的创建 Redux App，并最大程度的达到代码复用的目的。

## **test**

**test** 目录主要包含测试文件和一些测试配置文件。

## dotfiles

项目顶层主要还包括其它一些 dotfiles，如 .babelrc，.eslintrc。

```
- .babelrc
- .eslintrc
- .eslintignore
- .gitignore
- commitlint.config.js
- jest.config.js
- package.json
- README.md
- yarn.lock
```

## 实践原则

- [优先使用 `.js` 作为文件名后缀](https://github.com/facebookincubator/create-react-app/issues/87#issuecomment-234627904)
- 每个文件只包含一个 React 组件或是一个函数
- 每个 React 组件/模块自包含，即 JS/CSS/... 包含在组件自己的目录中，文件名需要与默认导出名一致
- 如果某个组件/模块只在另外一个组件/模块中使用，那么就让它嵌套在另外一个组件/模块目录中

```
components
  - Header
    - HeaderDropdown
    - Header.js
    - Header.less
    - ...
```

- 为**utils** 目录添加 `index.js` 索引，由它来暴露公共 API 接口
- 配置 [resolve.modules](https://webpack.js.org/configuration/resolve/#resolve-modules)从而避免嵌套 **imports**，如 `import Button from '../../../../../../../shared/components/Button/Button'`
- 采用 [Redux Ducks](https://github.com/erikras/ducks-modular-redux)，把 **reducers**，**action types** 以及 **actions** 放在一个文件中

## **总结**

目前来说，这种结构对于我们来说是一种最好的解决方案，但是这种结构并不能满足所有的情况（比如 SSR ），但是它已经足够灵活，同时支持水平扩展，我们可以很轻易的知道哪个地方添加新功能，而并不会侵入已有的逻辑。
