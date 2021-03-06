# 活跃网络的前端工程化实践之路

日期：2017-10-19

在活跃网络（成都）呆了三年半左右，搬过砖，踩过雷，架过构，管过人，想聊聊这些年在活跃网络的前端工程化实践之路。

## 前端架构历史

在 2014 年之前，活跃网络（ACTIVE Network）的前端技术架构基本可以分为以下两种类型，后端技术包括但不限于 Java, .NET, PHP, RoR，Adobe Flex 等等。

- [以 **jQuery** 为基础的单体 **Web App**](http://www.active.com/)
- [以 **Backbone.js + RequireJS** 为基础构建的基于 MVC 的 **SPA**](https://endurance.active.com/)
- **React** 之路...

很明显，在第一种方案中，是以后端为主的 MVC，前端开发重度依赖于后端的开发环境，并且前后端的职责纠缠不清（路由，模板到底应该放在前端还是后端？），很多时候前后端的代码会严重耦合。

而对于业务场景较复杂的系统，前后端的代码经常会混杂在一起，长此以往，可维护性以及可扩展性都将无从谈起，后面功能的开发也只能是在以前代码的基础上修修补补，直到无法忍受的一天就会产生重构的想法。

第二种方案中，是随着AJAX的兴起而带来的前端SPA时代。前端负责**视图+路由+模板+AJAX**，后端只是负责提供数据接口（RESTFul API，WebSocket 等等），从此前后端职责更加清晰，前端可以自由选择自己的框架，库，及至模块引擎，也可以进行合理的垂直分层，让各层各司其职，互相配合。

同时，在工程化实践过程中，我们遇到的挑战也是很明显的，主要有以下几个方面：

- 前端没有统一的**标准规范**，包括底层技术选型（库，框架，工具，测试等等），开发工作流程，风格指南，代码规范等等
- 前端底层技术栈不统一，无法达到最大程度的复用，小到组件级别复用，大到流程的复用等等
- 前端没有统一的自动化工具，流程，如 CI/CD，全靠手动
- 前端没有测试（单元测试，集成测试或是自动化测试）来保证代码质量，基本是靠工程师自测，或是QA的手动测试外加部分自动化（只能覆盖核心流程且易经常变动）

所有这些都是我们目前所遇到的挑战和必须要解决的难题，但是，有一点是始终没有变的，那就是最终前端工程化实践的核心诉求都是一致的：

> 减少开发技术成本，提升产品迭代效率以及产品体验！！！

## 前端标准化

对于前端标准化来说，我们要做的就是建立好前端的基础底层架构，规范开发技术选型，达到最大程度的复用，从而减少我们的开发技术成本，进而提升产品的迭代效率。

在这一方面，最近一年半时间内，我们做了很多工作，也有了很多产出。

### 企业级的 `CSS` 框架

**Active.css** 是活跃网络内部的企业级 `CSS` 框架，它确保了整个活跃网络产品线可以保持一致的外观和行为，并且让我们可以很容易的去对整个产品线做更新。它所提供的数十几种灵活实用且可重用的组件，可以帮助我们在之后的开发过程中进一步提高生产力，并且，它也让所有的开发者，设计师，产品经理，以及市场人员可以参考同一套标准，做到有据可依，有理可循。

在过去，每个产品团队会实现一套自己的 `CSS` 样式库，所以常常会遇到一些问题：

- 很难去维护和重用CSS样式
- 保持活跃网络所有产品线体验一致

但是，有了 **Active.css** 帮助我们统一产品的风格指南，我们会有如下的优势：

- 标准化

整个公司可以共用一套 **CSS** 样式库，从而可以继续维持一个更高标准和最佳实践

- 一致性

针对开发者，设计师，产品经理，以及市场人员等等，建立了一套公共语言，并且让它应用到不同的产品的同时，还能继续保持一致性

- 高效

每个产品团队不用自己去开发和维护公共样式库，只需要重用一套代码标准规范。也就意味着，开发人员可以花更多的时间在构建业务上，而不是花在考虑应该如何写基础样式上

- 可维护性

以组件为最小单元来构建每个 **CSS** 组件，通过组合进而构建更为复杂的组件或者业务单元

另外，为了维持整体的CSS风格统一，也推出了 *CSS 风格指南*，它会提供一些关于CSS的最佳实践，如文件目录结构组织，命名，样式规则等等。

![ACTIVE CSS Style Guidelines](/images/active-css-guidelines.png)

### 基于 React 的 **UI** 组件库

从 2016 年初开始，团队开始采用 [React](https://facebook.github.io/immutable-js/) 作为构建前端视图的基础库，在这过程中，逐渐把公用组件剥离出来，使用得这些组件可以在各个项目中能够得到最大程度的复用。

组件主要包括以下几类：

- 独立于业务的单一组件：`Button`，`Input`，`Dropdown`，`Table`，`Modal`，`DatePikcer`，`Pagination`等等
- 独立于业务的复合组件：`Form`，`CheckboxGroup`，`RadioGroup`，`SearchInput`，`Tabs`，`Table`，`TagEditor`等等
- 业务组件（需要在多个项目中重用）：`AddressEditor`，以及与[i18n相关的组件 API](http://fee.surge.sh/2017-09-18-how-do-we-implement-i18n-based-on-react.html)
- 工具组件：`ShowAt`，`HideAt`，`Viewport`等等

每个组件所对应的样式定义都是单独放在 **Active.css** 中，也就是说，**JS** 和 **CSS** 是分开单独维护开发的。

为什么需要这样做呢？

活跃网络有很多存在了十几年的老项目，他们的实现方式多种多样，但是随着客户的需求变化，近年来他们都有一个统一的诉求，那就是『改头换面』，重新换上符合公司产品风格指南的全新的样式风格。但是为了尽量减少工作量，我们一般只做样式上的修改，而不是对整个应用做重构，内部称之为『Reskin』。

同时，我们也针对这套 UI 库，构建了它的线上 API文档，组件使用者可以通过它快速查询某个组件的API 接口规范，并且提供很方便的在线 *REPL*。

![ACTIVE UI](/images/active-ui.png)

### Git 工作流

#### 功能分支工作流

为了提高开发效率，形成规范化的代码管理，活跃网络已经完成了从 SVN 到 Git 的迁移，内部采用的是[功能分支工作流](https://www.atlassian.com/git/tutorials/comparing-workflows#feature-branch-workflow)，也就是说所有新功能的开发都是基于一个独立的功能分支，而不是传统的master分支。这样每个工程师都可以在属于自己的功能分支上开发，而不会影响到主分支，主分支可以随时发布上线，当然这得借助于自动化的 **CI/CD** 来保证。

### Git 提交消息格式

当我们提交代码时，都会涉及到一个永恒不变的话题---**如何给 `Commit` 加一段简短且有意义的描述**。

#### 如何写 Git 提交消息

那么我们应该如何写一个 **Git 提交消息**呢？

请移玉步至 [How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/)

**TLDR**:

- 用一个空行来分隔开主题和正文
- 主题限制在 50 个字符内
- 主题行首字母开始
- 主题行不要以句号结尾
- 主题行使用祈使语气
- 正文的每行限制在72个字符内，超过部分折行显示
- 使用正文部分去解释 是什么 和 为什么 而不是 怎么做

目前团队内部参考的Git提交消息最佳实践是如下二个：

- [Udacity Git Commit Message Style Guide](https://udacity.github.io/git-styleguide/)
- [AngluarJS Commit Message Format](https://github.com/angular/angular.js/blob/master/CONTRIBUTING.md#commit-message-format)

二者大同小异，完全满足上面这七大原则，都可以帮助团队统一 **Git 提交消息格式**，从而与他人协作更加方便。

> 推荐使用[commitizen](https://github.com/commitizen/cz-cli)来帮助团队规范化整个流程，与npm scripts配合自动化整个流程。

#### 自动检测 Git 提交消息格式

随着团队人员增多，我们需要一种工具能够自动化帮助我们检测 Git 提交消息格式是否满足规范要求。推荐使用 [marionebl/commitlint](https://github.com/marionebl/commitlint) 来自动化 Lint Git 提交的消息格式，并且配合 *prepush hook* 一起使用。

#### 自动生成`CHANGELOG`

如果是库或是框架的开发，我们需要提供 *CHANGELOG* ，以让团队内部和外部人员知道我们最近所做的更新，如修改的 Bugs，新增加的功能或是其它性能上的提升等等

如果需要根据 Git 的提交和其它元数据自动生成*CHANGELOG*，请参考 [conventional-changelog/conventional-changelog](https://github.com/conventional-changelog/conventional-changelog) 命令行工具。

### 前端技术选型

[前端开发技术日新月异](http://stateofjs.com/2016/frontend/)，在做前端技术选型架构时，团队内部会充分考虑各个框架库的优缺点，选择更加贴合项目需求的成熟技术方案。

目的很简单，希望能够最大限度的帮助我们减少技术成本，提高迭代效率，运行效率，以及用户体验。

#### Single Page App(SPA)

我们经常提到SPA，那么到底什么是SPA呢？

- 它是一个单一的 **HTML** 页面
- 它在页面之间跳转时，不需要重新刷新
- 它使用 **JavaScript** 来编写业务逻辑，并运行在浏览器端
- 在用户与页面交互时，它可以部分更新页面
- （可选）基于路由实现[按需加载](https://webpack.js.org/guides/code-splitting/#dynamic-imports)
- （可选）基于 [Service Worker](https://github.com/GoogleChrome/sw-precache) 提供离线支持

> 目前并没有使用 **Node.js** 来作为中间的 **Web Server** 层，前后端之间唯一的通信桥梁就是 **Web API**。如果之后需要支持 **SSR**（server-side rendering） 或是 **State Persistence** 等等时，会考虑在前后端之间添加 **Node.js** 层。

#### Dev Environment

如何管理好上万行代码前端单页应用是团队的一个核心挑战，一个良好的项目目录结构则是首要条件，具体内容，请参考 [我们如何组织 React + Redux 项目结构](/react/2017-06-04-how-we-structure-react-redux-project-stucture.html)。

#### React

使用 [React <sub>视图层</sub>](https://github.com/facebook/react)来构建基于组件的富用户界面层。React是基于组件化的设计，与前端的技术趋势（模块化，组件化）非常接近，在前端社区中也非常流行，支持者众多，并且它是由Facebook官方支持和亲身实践的，最近也修改为[ MIT 许可](https://code.facebook.com/posts/300798627056246/relicensing-react-jest-flow-and-immutable-js/)。下面是很喜欢的一些 **React** 的特性：

- 组合（Composition）
- 单向数据流（Unidirectional Dataflow）
- 声明式（Declarative）
- 显示变化（Explicit Mutations）
- React API 简单，小而美
- 没有多余的概念，只是 **JavaScript**
- 虚拟 DOM
- 函数式编程
- SSR（服务端渲染）
- ...

#### Types

类型系统可以选择的很多，如 **[prop-types](https://reactjs.org/docs/typechecking-with-proptypes.html)**，**[flow](https://flow.org/)**，以及 **[TypeScript](https://www.typescriptlang.org/)**。

我更倾向于 prop-types，因为它的侵入性很小，但是已经为 React 组件提供了足够的类型安全，另外再加上测试和 ESLint，在最近的项目中，很难发现运行时类型错误。

#### Router

使用 [react-router@v4 <sub>Declarative routing for React</sub>](https://github.com/ReactTraining/react-router) 来作为客户端的路由解决方案，从而让用户可以在页面之间跳转而不需要重新刷新整个页面。从**react-router@v4**开始，它的基础代码全部被重写了，如今的`Route`更偏向于组件，它只是基于当前的URL是否与所设定的路径一致，从而决定显示或是隐藏组件，这也更加**React**，更加贴合组件化的思想。想了解更多的改变，请移玉步至 [All About React Router 4](https://css-tricks.com/react-router-4/)。

#### Redux

使用 [Redux <sub>Predictable State Container</sub>](http://redux.js.org/) 来构造和管理复杂的 **Web App** 状态，它能帮助我们解决：

- 管理各种各样的状态：Domain数据，UI状态，App状态等等
- 在Web应用的整个生成周期中，在哪里保存所有的数据呢？
- 保存好这些数据后，应该如何处理这些数据的修改呢？
- 如何让整个Web应用知道到这些数据变化了呢？

#### Redux Promise Middleware

在整个Redux **Web App** 中，统一使用团队内部构建的Redux Promise Middleware来处理所有的副作用（异步逻辑），比如 **RESTFul API**。它与 *redux-thunk*， *redux-saga* 或是 *redux-obserable* 相比，它更适合我们团队的需求，简单易上手，基本满足我们的需求：

- 相当于一个轻量级的库，专门用于 **Resolve Promises** 和 **Reject Promises** ，并且它是完全基于乐观更新的
- 让我们可以更加灵活的去处理Redux中的异步 `Actions`，同时依据一定的命名规范会自动派发 `${REQUEST}`, `${REQUEST}_SUCCESS` 和 `${REQUEST}_FAILURE` 的 `Actions`

#### Isomorphic Fetch

基于 [isomorphic-fetch](https://github.com/matthew-andrews/isomorphic-fetch) 封装了一套 **api.js**，作为 **Web API** 工具来处理 **AJAX** 的请求和返回。

#### `Immutable.js`

使用 [Immutable.js](https://facebook.github.io/immutable-js/) 来作为状态存储库。它提供了很多常用的 Persistent Immutable data structures 来帮助我们管理数据存储，并且保证它们是 Immutability 的。这就要求我们操作所有数据时，都要当作不可变数据（immutable data）来对待，保证每次都是全新对象，没有引用关系，这样才能保证State的独立性，便于测试和追踪变化。同样的，它也是来自于 Facebook，与 React 是天生一对。

#### React UI 组件库@`react-aui`

使用基于 React 构建的 **react-aui** 作为基础的 UI 组件库，它包含了一系列满足我们的 *Product Style Guide* 的灵活，实用且可重用的组件。

#### `Tempest.js`

内部开发和维护的一个基于 React + Redux 以及包含我们的一些最佳实践的框架，以帮助我们能够快速的构建 Web App.

#### `build-tools`

之前，我们每次新建一个项目时，都需要去建立和配置很多工具集，包括测试，构建，发布，代码检查等等，因为已经有以前的成功经验，所以大多数时候直接 copy-and-paste 就搞定了。

但是，如果 Webpack 新版本性能提升了，然后需要升级了，又或是我们需要新增加一个 ESLint 规则，再或者是我们需要新增一个 **prepush hook** ，如 `commitlint`，然后我们又需要在每个项目里面单独维护，这大大影响我们的工作效率。

`build-tools` 提供了我们常见的脚本 (build/release/lint/test/...)，并且允许高度自定义。 如果你需要维护多个项目的话，这样的工具可以让你更容易管理你的工具依赖并随时保持更新。

比如，如果你要扩展 Jest 配置，你可以这样做：

```
const { jest: jestConfig } = require('build-tools/config')

const { setupFiles } = jestConfig

// Config it depending on your requirements
module.exports = Object.assign(jestConfig, {
  setupFiles: [
    '<rootDir>/test/polyfill.js',
    '<rootDir>/test/setup.js',
  ],
})

```

![build-tools-package-diff](/images/build-tools-package-diff.png)

#### Styling

对于 React 组件来说，我们可以数十种方式来为组件添加样式，比如内联样式，CSS，LESS/SASS, [CSS Modules](https://github.com/css-modules/css-modules) 以及各种 [CSS-in-JS](https://github.com/MicheleBertoli/css-in-js) 实现。

目前我们选用的是 [LESS](http://lesscss.org/) + [BEM](http://css/guidelines/)，但是 LESS 我们只限定使用它的 *Variables*， *Mixins* 和 *Nested Rules*，其它功能不推荐使用。

#### Yarn

使用 [Yarn](https://yarnpkg.com/) 来做为我们的包管理工具。

#### Jest

使用 [Jest](https://facebook.github.io/jest/) 和 [Enzyme](https://github.com/airbnb/enzyme) 来作为测试的解决方案。

有一些特别的 Jest 配套工具推荐，以提高写测试效率：

- [enzyme-to-json](https://github.com/adriantoine/enzyme-to-json#usage): Cutomize your serializer for Enzyme wrappers
- [snapshot-diff](https://github.com/jest-community/snapshot-diff): Diffing snapshot utility for Jest
- [jest-expect-contain-deep](https://github.com/Thinkmill/jest-expect-contain-deep): Assert deeply nested values in Jest

如果想从其它测试框架迁移至 Jest，推荐 [jest-codemods](https://github.com/skovhus/jest-codemods)。

#### Webpack

使用 [Webpack](https://webpack.js.org/) 来作为 Module Bundler，它提供了丰富的 Loaders 和 Plugins 来帮助我们处理各种各样的资源，同时它还支持 Tree Shaking 和 Code Splting，可以更好的帮助我们做组件化开发与资源的管理。

> 根据增量的原则，我们应该精心规划每个页面的资源加载策略，使得用户无论访问哪个页面都能按需加载页面所需资源，没访问过的无需加载，访问过的可以缓存复用，最终带来流畅的应用体验。

#### 其它

- 优先使用 **ES6 Class** 而不是 **[createClass](https://reactjs.org/docs/react-without-es6.html)**
- 优先使用 **[React Stateless Functional Component](https://hackernoon.com/react-stateless-functional-components-nine-wins-you-might-have-overlooked-997b0d933dbc)** 而不是 ES6 Class Component
- 优先使用 **[Class Prop Arrow Function](https://github.com/tc39/proposal-class-fields)** 来 Binding `this`
- 优先使用 **[Render Prop](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce)** 而不是 **[HoC](https://reactjs.org/docs/higher-order-components.html)** 来实现复用
- [优先使用 `.js` 作为文件名后缀](https://github.com/facebookincubator/create-react-app/issues/87#issuecomment-234627904)
- 每个文件只包含一个 React 组件或是一个函数
- 每个 React 组件/模块自包含，即 JS/CSS/... 包含在组件自己的目录中，文件名需要与默认导出名一致
- 如果某个组件/模块只在另外一个组件/模块中使用，那么就让它嵌套在另外一个组件/模块目录中

### 开发流程

请移玉步至[活跃网络前端开发流程](/2017-09-25-active-development-workflow.html)。

### 项目目录结构

请移玉步至[我们如何组织 React + Redux 项目结构](/react/2017-06-04-how-we-structure-react-redux-project-stucture.html)。

### 风格指南和代码规范（ESLint + Prettier）

风格指南是代码规范的其中一种形式，它针对的是单个文件的布局，而代码风格更加强调的是一些编程最佳实践，文件和目录的设计等等。

**JavaScript** 是一门动态并且类型松散的语言，再加上团队成员水平参差不齐，急需要一种工具可以帮助我们发现代码中常见的 Bugs 或是潜在的问题，提高代码质量，保持团队代码风格一致。因此我们引入了[ESLint](https://eslint.org/)作为代码的静态分析工具，并自动集成到我们的CI/CD流程中，也就是一旦发现ESLint错误，那么整个CI/CD会失败，直到修复这些错误为止。

另外，ESLint支持创建 [ESLint Shareable Configs](https://eslint.org/docs/developer-guide/shareable-configs)，团队内部基于 [eslint-airbnb](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb) 创建了适应自己的 *ESLint Shareable Config*。

最近，我们也集成了 [Prettier](https://github.com/prettier/prettier) 来作为代码格式化工具，之后，**Prettier** 会主要负责代码风格一致，而 **ESLint** 会确保我们的代码质量是满足最佳实践的，各自分工，互不耽误。

![.eslintrc](/images/eslintrc.png)

### 测试

前端的测试解决方案是基于 **Jest + Enzyme** 的，并且会添加 **prepush hook**，以确保在提交代码前，至少在本地机器上跑测试通过的。

基本上，我们会要求单元测试的代码覆盖率至少是要达到**百分之九十**的，它表示的是 *Statements*，*Branches*，*Functions*，*Lines* 这四个维度的平均值。有了这些测试后，在之后的开发，重构或是修复 Bug 中，我们就可以依据需求任意的做出相应的修改，同时，我们也有足够的信心不会破坏已有的代码。

![jest.config.js](/images/jest.config.js.png)

### 自动化 - CI/CD

**CI** 也就是我们经常说的持续集成，指的是频繁地（或者是一天多次）将代码集成到主干分支，它现在是我们开发流程中必不可少的一环。它主要可以帮助我们：

- 快速发现错误。每完成一点更新，就集成到主干，可以快速发现错误，定位错误也比较容易。
- 防止分支大幅偏离主干。如果不是经常集成，主干又在不断更新，会导致以后集成的难度变大，甚至难以集成。

持续集成的目的，就是让产品可以快速迭代，同时还能保持高质量。它的核心措施是，代码集成到主干之前，必须通过自动化测试。只要有一个测试用例失败，就不能集成。

**CD**也就是持续交付（Continuous delivery），它指的是频繁地将软件的新版本，交付给质量团队或者用户，以供评审。如果评审通过，代码就进入生产阶段。
持续交付可以看作持续集成的下一步。它强调的是，不管怎么更新，软件是随时随地可以交付的。

在 ACTIVE Network，工程师在自己的分支上完成代码开发后，会先提交一个 Gitlab Merge Request(MR，类似 GitHub 的 PR)，以供团队其它成员做 Code Review。代码仓库都会预先设置好 [Webhook](https://docs.gitlab.com/ce/user/project/integrations/webhooks.html)，所以只要有创建，更新或是合并 MR 时，都会自动运行仓库的 [Jenkins Pipeline](https://jenkins.io/doc/book/pipeline/getting-started/)，下面是一个典型的 **Jenkinsfile**：

![Jenkinsfile](/images/Jenkinsfile.png)

一般来说，它会执行如下的步骤：

1. 拉出最新的需要合并的分支代码
1. 安装依赖
1. 运行 Lint 脚本
1. 运行测试（包括单元测试和集成测试）
1. 发布测试代码覆盖率报告

## 发布

通过 CI 流程，代码已经可以合并到主干分支了，我们就可以基于它做发布。一般发布流程包括：

- 运行ESLint
- 运行测试
- 构建（Webpack）
- 在 `package.json` 中升级版本号（如果没有版本，则只升级PATCH版本）
- 创建一个新的提交
- 创建一个叫 `v*` 的标签（如 v1.15.35 ），并指向上一步创建的提交
- 推送至远程分支(master)
- 以 **zip** 格式将构建成功的前端资源文件（HTML, JS, CSS, FONTS等等）打包，并发布到远端的Nexus服务器
- 推送 `v*` 标签至远程仓库
- 创建并推送 `latest` 标签至远程仓库

这样一来，前端代码库拥有了自己的版本管理，并且每个打包的版本只是一些构建后的 HTML/JS/CSS/Fonts 的集合，任意的后端服务都可以使用它。

## 部署

一般会在 Jenkins CI 机器上执行如下步骤：

- 首先从 `package.json` 中获取最新的版本号
- 后端的生产服务器在部署前，会先备份当前代码
- 从远程的 **Nexus** 服务器上下载指定版本（第一步中获取的）的前端代码库，之后将打包文件解压到本地的一个目录
- 之后再将运行路径的符号链接指向这个目录或者是通过 **Rsync** 将解压文件传输至生产服务器上指定目录中
- 重新启动应用

这方面的部署工具有 **Ansible**，**Chef**，**Puppet** 等，我们内部用的是 **Ansible** 来做部署。

## 团队建设

如何保持团队增长是一个难题！说白了，就是想让大家一起开开心写代码，同时个人也能有所成长收获。

一个**Team Leader**的成功更重要的是取决于他/她的存在是否推进了公司的成功，尤其是技术相关方面的成功，另外他的存在是否帮助了团队里每个工程师的成功，帮助他/她们作为一个个体的成长和成熟。

### 技术分享

前端技术瞬息万变，库，框架，工具集，开发流程，测试，CI/CD等等，基本上每隔三个月就会更新换代，这就要求我们时刻保持更新，自我成长，多去接触更广阔的世界。

我们内部每周四定期会有一个Tech Talk，针对某一项技术（无论前后端，新或是旧，只要有用），或是一个大家平常遇到的项目问题，或是技术调研的结果。分享的目的在于将自己所了解的东西以最简单的方式讲给大家听，然后自己和其他人在这个过程中都能有所成长。

关于技术分享更多的主题，可以参考[Trello Board](https://trello.com/b/7GoAEqNs/active-front-end-tech-talk)。

### Code Reivew

写代码可以帮助我们快速成长，同样的，作为一个代码审核人，也是一个自我学习和成长的过程。代码审核最重要的好处就是帮助我们最大限度的确保**代码质量**，统一代码风格，提早发现Bugs。同时，这也能帮助我们建立一个健康的工程师文化，促进团队成员之间的沟通，保持代码的简洁和可维护性。

代码审核绝对不是指责的工具，同事之间相互的CR过程中，我们也能从彼此那里学到很多编程技巧，并且培养起很好的编程习惯，**学习和成长**才是核心目的。

为了规范化CR流程，我们会把CR加入到我们开发流程中来，尽量自动化整个流程，另外，针对CR，我们也有一些行为准则：

- 使用 **Gitlab Merge Request** 来管理 CR
- 每个 CR 必须满足 [SRP](https://en.wikipedia.org/wiki/Single_responsibility_principle) 原则，每次只关注某某一个功能
- 提交 CR 后，必须确保 CI 是成功的
- 使用 **precommit hook** 来做代码规范或是自动化测试检查，而不是留到 CR 中
- 每个人都要参与 CR 中，而不只是 Senior 工程师来对 Junior 工程师做的
- 在代码提交到主干分支前做 CR（`pre-commit` CR），而不是合并到主干分支之后
- 每次提交审核的代码必须少于 400 行，避免提交数几个文件或是数千行代码

> 推荐使用 [husky](https://github.com/typicode/husky) 来实现 **Git hooks**。

