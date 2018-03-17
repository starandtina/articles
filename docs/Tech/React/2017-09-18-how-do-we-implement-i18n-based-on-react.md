# 我们如何基于 React 实现 i18n

日期：2017年09月18日

简单来说，我们需要提供一套 API ，根据当前 **locale** 而显示不同的文本消息，日期时间，货币等等。

> 本文不涉及基础概念问题（如什么是i18n，l10n），请自行[Google](https://stackoverflow.com/questions/754520/what-is-the-actual-differences-between-i18n-l10n-g11n-and-specifically-what-does)。

## 目标

- 根据当前 **locale** 异步加载 i18n 资源，包括文本消息，日期时间，货币等等
- **locale** 一旦改变，相应的文本，日期，货币会立即随着变化，**无论React组件树层级多深**
- 可以独立于 React 组件使用
- 一套简单的声明式的 API 优于命令式 API

由于我们的需求，以及下面所列种种原因，最终导致我们没有选择 [react-intl](https://github.com/yahoo/react-intl) 作为我们的国际化解决方案。:(

> 做为前端工程师，我们的核心任务并不是去 Build 更多的 UI 组件库，而是需要更好的去服务我们的 Business。长话短说就是，尽可能选择开源的解决方案，长远来说，这必将会降低成本和减轻风险。

- [Safely use context react-intl#901](https://github.com/yahoo/react-intl/pull/901)
- [Children don't update when context changes react-intl#660](https://github.com/yahoo/react-intl/issues/660)
- [How to implement shouldComponentUpdate with this.context? react#2517](https://github.com/facebook/react/issues/2517)
- [My views aren’t updating when something changes outside of Redux](https://github.com/reactjs/react-redux/blob/93cdfaeaf9d3e5400ffc05fe9d177118286109ca/docs/troubleshooting.md#my-views-arent-updating-when-something-changes-outside-of-redux)

![l10n.gif](/images/l10n.gif)

## 实现

### 异步加载 i18n 资源

我们是用 [Webpack](https://webpack.js.org) 来作为打包工具的，Webpack 天生就支持代码分割，按需异步加载代码块，所以无论是 Webpack@1 中的 [require-ensure](https://webpack.js.org/api/module-methods/#require-ensure)，还是其后续者 Webpack@2 中的 [import() 语法](https://webpack.js.org/guides/code-splitting/#dynamic-imports)，都可以帮助我们做到*根据当前 **locale** 异步加载 i18n 资源*。

默认选择的是 **import()**，首先[它是符合 ECMAScript 规范的](https://github.com/tc39/proposal-dynamic-import)，另外，从 Webpack@2.4 起，Webpack 也支持通过 `webpackChunkName` 来自定义每个区块名，已经可以完全取代 `require.ensure`。

### React - `Context`

首先我们是基于 React 的组件模型的，它提供了一套很好的父子组件的通信方式。一般常见方式是，父组件可以通过 `props` 传递数据给子组件，而子组件可以通过调用 *callback prop* 来回传数据，这样一来，数据流就很清晰，代码也易读并且容易维护。当出现问题时，也很容易找出问题所在，因为你知道是谁传递的这个 `callback`，是谁调用了这个 `callback`，也知道是哪两个父子组件正在通信。

所以，设计思路时的第一个想法就是，我们可以通过这样单向传递的方式来逐级传递 i18n 资源的。但是，问题也随之而来了，随着组件层级的逐级加深，每个组件不得不显示的将 `props` 传递给它的子组件（即使有可能没有用到它），更槽糕的是，有些中间层组件，根本不在我们的控制之内，有可能是第三方的组件库，导致我们根本没有办法保证 i18n 资源可以逐级下传至下层组件。

这里，你会突然醒悟，我们需要的一种机制，可以帮助我们做到**透传**。而[React Context](https://facebook.github.io/react/docs/context.html)完美的解决了这个问题。

> React Context: 当一个组件在它的 **Context** 上定义了一些数据后，任何它的后代子组件都可以访问这些数据。

这也就意味着，任何在组件树中的子组件都可以访问所在 **Context** 中的数据，而并不需要通过 `prop` 来传递。具体使用方法，请参考[官方文档](https://facebook.github.io/react/docs/context.html#how-to-use-context)。

### React@`shouldComponentUpdate` -> Observer Pattern

那么我们现在知道是在 `Context` 中放我们的 i18n 资源数据，那么 `Context` 是什么时候会更新呢？

[官方文档是这样写的](https://facebook.github.io/react/docs/context.html#updating-context)：

> The `getChildContext` function will be called when the state or props changes. In order to update data in the context, trigger a local state update with this.setState. This will trigger a new context and changes will be received by the children.

但是 `Context` 更新了之后，在组件树中的后代子组件能够同步获取最新值吗？如果可以，又是如何获取的呢？

这就取决于中间层组件的 `shouldComponentUpdate` 实现。

任何一个 React 组件都可以定义自己的 `shouldComponentUpdate` 实现（或者继承自 `PureComponent` ），如果它返回 `false`，那就表明该组件及其子组件都不需要重新渲染。但是，如果某一个中间层组件的 `shouldComponentUpdate` 也返回 `false`，那么该组件的子组件也就不会更新，即使是当前组件树中的 `Context` 已经变了。:)

一图胜千言:

```
高层组件（包含 context.color 定义）

    中间层组件（`shouldComponentUpdate` 返回 false）
        ...
            ....
                子组件（从当前 context 中获取数据）
    ...
    ...
```

以上图为例，如果 `context.color` 更新了，子组件是不会重新渲染的，因为它的父级组件的 `shouldComponentUpdate` 返回了 `false`。这样直接导致的结果就是 - **全乱套了**，UI 和状态之间无法同步，从而导致各种莫名其妙的 Bugs，这也是大家常常不愿意用它的另外一个重要原因。

那么，怎么解决呢？答案就是 **`Context` + Observer Pattern**。

首先，我们会有二个假设：

- `Context` 是 **immutable** 的，并且由当前组件自己维护自己的内部状态
- 子组件只需要接受一次 `Context`，之后 `Context` 的更新都能[通过订阅获取](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#observerpatternjavascript)

设计方案有了，接下来，我们来看看如何设计我们的 API。

### API 设计

#### `L10n`

它主要用于：

- 实现观察者模式并且维护观察者列表
- 当 **locale** 改变时，负责通知所有观察者，如 `L10nMessage`，`L10nDateTime`，`L10nCurrency` 或是其它通过 `injecctL10n` 注入的组件
- 提供格式化工具方法，如 `formatMessage`，`formatDateTime`，`formatCurrency` 等等

#### `L10nProvider`

它主要用于：

- 根据当前 **locale** 异步加载 i18n 资源
- 封装 `L10n` 对象，通过 `Context` 暴露给子级组件
- 通常来说，它会作为整个应用最顶层组件使用（如 `Provider` ），其它 `L10nXXX` 组件不能脱离它独立使用

#### `injectL10n`

它主要用于：

- 根据当前 `Context` 中的 `l10n` 对象，生成一个新的 **Bounded 的 l10n 对象**，然后以 *props* 的形式自动注入到绑定组件中
- 实现自动注入 `l10n`，并且订阅 `l10n` 资源的更新，一旦有任何更新，进而会调用 `forceUpdate` 强制更新组件

```
getBoundL10n = () => {
  const l10n = this.context...
  const boundFuncs = ...
  const boundConfig = ...
  return {
    [l10nName]: {
      ...l10n,
      ...boundConfig,
      ...boundFuncs,
    },
  }
}
```

这样就保证在 **locale** 或是其它资源（如 `messages` ）变化后，被注入的组件会有 `props` 的变化，从而会触发组件的重新渲染，进而重新获取 `l10n` 中的数据

#### `L10nMessage`

`L10nMessage` 是 `l10n.formatMessage` 方法的组件封装，它根据当前传入的 `id`，找到对应的文本信息，同时也支持字符串插值。

![L10nMessage.js](/images/L10nMessage.js.png)

#### `L10nDateTime`

`L10nDateTime` 是 `l10n.formatDateTime` 的组件封装，它根据当前传入的 `date` 对象和日期时间的 `format`，转换回对应 `format` 的字符串形式。

比如，`en_US` 的长日期时间格式为 `MMMM d, yyyy h:mm a`，然后传入当前的 `date` 后，就会转换为 `2017/09/31 11:30 a.m.`

#### `L10nCurrency`

`L10nCurrency` 是 `l10n.formatCurrency` 的组件封装，它根据传入的数值 `amount` 和货币代码 `code` （如 `USD` ）,转换回对应格式的字符串表示形式。同时，用户可以指定以下选项：

```
{
  integerOnly: false, // 是否是整数形式
  separationCount: 3, // 多少字符做分隔
  separator: ',',     // 数字之间的分隔符
  negativeMark: '-',  // 是否需要添加负号标记
}
```

### 数据流

那么，有了这些 API 组件后，它们之间是如何协同工作的呢？

以下面这个典型的组件树为例：

```
<L10nProvider locale='zh_CN'>
  ...
    <UpdateBlocker>
        <L10nMessage id='save' />
        ...
    </UpdateBlocker>
  <Form>
    <Field
      placeholder={l10n.formatMessage('save')}
    />
  </Form>
</L10nProvider>
```

> `UpdateBlocker`组件意味着该组件的`shouldComponentUpdate` 生命周期方法始终返回 `false`

- 在 `L10nProvider` 中，会首先初始化一个 `L10n` 对象或是传入的 `L10n` 对象
- `DateTimeSymbols`，`currenciesConfig` 和 `en_US` 的 i18n 文本消息，会默认绑定到上一步生成的 `L10n` 对象上
- 在 `L10nProvider` 的 `componentDidMount` 生命周期方法中，会去动态加载指定 `locale` 的文本消息（此处是 `zh_CN` ）
  - 加载成功后，会与当前 `this.l10n.messages` 合并
  - 通知所有订阅子组件（通过 `injectL10n` 实现自动注入和订阅l10n的更新的组件，如 `L10nMessage` ）
- 子组件收到更新通知后，会从当前 `Context` 中，拿到 immutable 的 `L10n` 对象后，重新生成一个新的 `L10n` 对象字面量，然后以 `prop` 的形式注入到绑定组件中
- 最后，子组件调用对应的格式化方法

## 总结

通过实现 i18n 组件让我们更加深入的了解了 React 的组件模型，以及组件之间的通信方式。对于组件化的编程模型有了更加深入的思考和探索，这样的经历，又何乐而不为呢？:)


