# Application State Management with React

> This is a translation of the original post [Application State Management with React](https://kentcdodds.com/blog/application-state-management-with-react) by [Kent C. Dodds](https://kentcdodds.com/) on July 21st, 2020.

> 原文来自于[Kent C. Dodds](https://kentcdodds.com/)发布于2020年7月21日的文章 [Application State Management with React](https://kentcdodds.com/blog/application-state-management-with-react).

状态管理在任何应用的开发都是毫无争议的最麻烦的。这也是为什么现在有大量的状态管理库存在，而且还在不断的有新的库出现（甚至有很多竟然建立在其他的库之上，比如 npm 上有数百个 “简化 redux” 的抽象）。尽管状态管理是一个难题，但我认为，使它变得如此困难的原因之一是我们经常过度设计问题解决方案。

从我使用 React 以来，我个人尝试过去实现过一种状态管理方案，随着 React hooks 的发布（以及 React context 的大量改进），这种状态管理方法也大大的简化了。

我们经常谈论要像搭建乐高积木那样去构建 React 组件，我想当人们听到这个，他们不知何故排除了状态管理这块，我个人去解决状态管理问题的方案的“秘诀”是考虑应用的状态将如何映射到其树结构。

react-redux 如此成功的原因之一也是 react-redux 解决了 [props drilling](https://kentcdodds.com/blog/prop-drilling) (也就是 props 向下或者平行传递)的问题，事实上，你能够将你的组件传给 `connect` 函数，以此树下的不同的部分来共享数据，它同时也用到了 reducers/action creators等等，也很不错。但是我坚信 redux 应用如此广泛还是因为它为开发人员们解决了 props drilling 这个痛点。

这就是我只在一个项目上使用过 redux 的原因：我一直看到有开发人员把所有的状态都放在 redux，不只是全局状态，包括了局部状态，这样导致了很多问题，其中最重要的是，当你在与状态进行交互的时候，也会包括了 reducers action creators/types dispatch calls 这些，导致你必须在编辑器中打开许多文件以来追踪大脑里所正在思考的代码逻辑，以确定发生了什么，以及对代码的有没有其他的影响。

需要明白的是对于全局变量这是可以的，但是一些简单的状态（比如打开对话框或者输入框的输入内容）这会是个大问题，更糟糕的是，这样的扩展性不强，随着你的应用越来越大，问题变的更加麻烦，当然，你可以将不同的 reducers 连接起来来管理应用不同的部分，但是这种间接连接的所有 action creator 和 reducer 并不是最优解。

将应用所有的状态放在一个单一的对象也会导致其他问题，无论你是不是用 redux，当 React 的 `<Context.Provider>` 拿到了一个新的值，所有会消费那个值的组件都会被重新渲染，甚至其中某个 function component 只关注部分的数据，这可能会引起性能问题（React-Redux v6 也尝试了这种方式，后来他们意识到这种设计并不适用于 hooks，这迫使他们在v7版本使用了另外的方法解决这些问题），所以我的意见是，将状态在逻辑上分离并且放置在 react tree 的合适的节点，这样就可以避免这个问题。

---

现在开始时解决方案，如果你在使用React构建你的应用，那么你已经拥有了一个状态管理库，你甚至不需要 `npm install` 或者 `yarn add`，它在npm上已经集成进了所有的React包，你不需要额外下载，而且React的团队制作了相应的文档，其实这个就是React自己本身。

> React 就是一个拥有状态管理的库

当你编写React应用的时候，那么你正在把所有的组件组合成一个树的结构，从 `<App />` 开始，到 `<div />`，`<input />` 或者 `<button />` 结束。你无法管理应用在一个中心位置呈现的所有低级复合组件，相反，让每个独立的组件来进行管理，最终成为UI构建的真正有效的方式，你也可以这么做，好比：

```JS
function Counter() {
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(c => c + 1)
  return <button onClick={increment}>{count}</button>
}

function App() {
  return <Counter />
}
```

[Edit on CodeSandbox](https://codesandbox.io/s/4qzj73lozx?fontsize=14&hidenavigation=1&module=%2Fsrc%2F01-simple-count.js&moduleview=1)

这里我所谈到的同样适用于 class component，Hooks 只是让事情变得更加简单一些（特别是 context 几分钟就能掌握）。

```JS
class Counter extends React.Component {
  state = {count: 0}
  increment = () => this.setState(({count}) => ({count: count + 1}))
  render() {
    return <button onClick={this.increment}>{this.state.count}</button>
  }
}
```

“好吧，Kent，当然，在单个组件中管理单个状态很容易，但是当我需要跨组件共享该状态时，该怎么办？例如，如果我想这样做怎么办：”

```JS
function CountDisplay() {
  // where does `count` come from?
  return <div>The current counter count is {count}</div>
}

function App() {
  return (
    <div>
      <CountDisplay />
      <Counter />
    </div>
  )
}
```

“`count` 属性在 `<Count />` 组件中，所以我需要状态管理库可以在 `<Counter />` 更新它并且在 `<CountDisplay />` 组件来获取这个值！”

这个问题的答案基本和React诞生同一时期（可能更早？），并且也有相应的文档，没记错的话应该是：状态提升 [Lifting state up](https://reactjs.org/docs/lifting-state-up.html)。

对于React的这个状态管理问题来说“状态提升 - Lifting State Up”是一个合理的答案，以下是将其应用于这种情况的方法：

```JS
function Counter({count, onIncrementClick}) {
  return <button onClick={onIncrementClick}>{count}</button>
}

function CountDisplay({count}) {
  return <div>The current counter count is {count}</div>
}

function App() {
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(c => c + 1)
  return (
    <div>
      <CountDisplay count={count} />
      <Counter count={count} onIncrementClick={increment} />
    </div>
  )
}
```

[Edit on CodeSandbox](https://codesandbox.io/s/4qzj73lozx?fontsize=14&hidenavigation=1&module=%2Fsrc%2F02-lift-state.js&moduleview=1)

这是一个很直接的解决方法，我们改变了之前修改这个值的方法的所在的地方，并且我们可以一直把状态提升到应用的头部。

“好吧，Kent，那么如何处理属性向下传递 [prop drilling](https://kentcdodds.com/blog/prop-drilling) 的问题？”

好问题，应对这种情况的第一道防线是改变构建组件的方式，利用组合组件模式，即 [component composition](https://reactjs.org/docs/context.html#before-you-use-context)，这样：

```JS
function App() {
  const [someState, setSomeState] = React.useState('some state')
  return (
    <>
      <Header someState={someState} onStateChange={setSomeState} />
      <LeftNav someState={someState} onStateChange={setSomeState} />
      <MainContent someState={someState} onStateChange={setSomeState} />
    </>
  )
}
```

可以这样：

```JS
function App() {
  const [someState, setSomeState] = React.useState('some state')
  return (
    <>
      <Header
        logo={<Logo someState={someState} />}
        settings={<Settings onStateChange={setSomeState} />}
      />
      <LeftNav>
        <SomeLink someState={someState} />
        <SomeOtherLink someState={someState} />
        <Etc someState={someState} />
      </LeftNav>
      <MainContent>
        <SomeSensibleComponent someState={someState} />
        <AndSoOn someState={someState} />
      </MainContent>
    </>
  )
}
```

如果这里不太明白，你可以看看 [Michael Jackson](https://twitter.com/mjackson) 有篇视频 [](https://www.youtube.com/watch?v=3XaXKiXtNjw)，能让更好的理解我的意思。

最后，甚至组合模式都不足够，所以下一步你可以看看 React's Context API，实际上，这已经是一个“解决方案”很长时间了，但是一直以来都是“非官方”的，正如我所说，许多人支持 `react-redux` 正因为它使用我所指的机制来解决了这个问题，而不用担心React的文档中的相关警告。但是现在 `context` 已经是官方支持的API，我们可以直接用。

```JS
// src/count/count-context.js
import * as React from 'react'

const CountContext = React.createContext()

function useCount() {
  const context = React.useContext(CountContext)
  if (!context) {
    throw new Error(`useCount must be used within a CountProvider`)
  }
  return context
}

function CountProvider(props) {
  const [count, setCount] = React.useState(0)
  const value = React.useMemo(() => [count, setCount], [count])
  return <CountContext.Provider value={value} {...props} />
}

export {CountProvider, useCount}
```

```JS
// src/count/page.js
import * as React from 'react'
import {CountProvider, useCount} from './count-context'

function Counter() {
  const [count, setCount] = useCount()
  const increment = () => setCount(c => c + 1)
  return <button onClick={increment}>{count}</button>
}

function CountDisplay() {
  const [count] = useCount()
  return <div>The current counter count is {count}</div>
}

function CountPage() {
  return (
    <div>
      <CountProvider>
        <CountDisplay />
        <Counter />
      </CountProvider>
    </div>
  )
}
```

[Edit on CodeSandbox](https://codesandbox.io/s/4qzj73lozx?fontsize=14&hidenavigation=1&module=%2Fsrc%2F03-context.js&moduleview=1)

> 注意：上面这部分例子是专门用来说明的，我不建议你使用context来解决这个特定的场景，请阅读 [Prop Drilling](https://kentcdodds.com/blog/prop-drilling)，以更好地了解为什么 prop thrilling 不一定是问题，不要太早使用 context ！

很酷的是我们可以把更新状态的常用方法放在 `useCount` 这个方法中：

```JS
function useCount() {
  const context = React.useContext(CountContext)
  if (!context) {
    throw new Error(`useCount must be used within a CountProvider`)
  }
  const [count, setCount] = context

  const increment = () => setCount(c => c + 1)
  return {
    count,
    setCount,
    increment,
  }
}
```

[Edit on CodeSandbox](https://codesandbox.io/s/4qzj73lozx?fontsize=14&hidenavigation=1&module=%2Fsrc%2F04-context-with-logic.js&moduleview=1)

可以轻松的将其改用 `useReducer`，而不是 `useState`。

```js
function countReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT': {
      return {count: state.count + 1}
    }
    default: {
      throw new Error(`Unsupported action type: ${action.type}`)
    }
  }
}

function CountProvider(props) {
  const [state, dispatch] = React.useReducer(countReducer, {count: 0})
  const value = React.useMemo(() => [state, dispatch], [state])
  return <CountContext.Provider value={value} {...props} />
}

function useCount() {
  const context = React.useContext(CountContext)
  if (!context) {
    throw new Error(`useCount must be used within a CountProvider`)
  }
  const [state, dispatch] = context

  const increment = () => dispatch({type: 'INCREMENT'})
  return {
    state,
    dispatch,
    increment,
  }
}
```

[Edit on CodeSandbox](https://codesandbox.io/s/4qzj73lozx?fontsize=14&hidenavigation=1&module=%2Fsrc%2F05-context-with-reducer.js&moduleview=1)

这为你提供了极大的灵活性，并将复杂度降低了几个数量级，下面有关于这么使用的一些提示：

1. 在你的应用中并不需要把所有的状态都放在一个对象中，保持把他们按逻辑分开（比如，用户设置的状态不需要和通知状态放在同一个context），以多个 providers 来提供相应的内容。
2. 并不是所有的 context 需要全局访问！让状态尽可能贴近他们需要的地方。

关于第二点更多的信息，你的应用的结构可能看上去如下所示：

```js
function App() {
  return (
    <ThemeProvider>
      <AuthenticationProvider>
        <Router>
          <Home path="/" />
          <About path="/about" />
          <UserPage path="/:userId" />
          <UserSettings path="/settings" />
          <Notifications path="/notifications" />
        </Router>
      </AuthenticationProvider>
    </ThemeProvider>
  )
}

function Notifications() {
  return (
    <NotificationsProvider>
      <NotificationsTab />
      <NotificationsTypeList />
      <NotificationsList />
    </NotificationsProvider>
  )
}

function UserPage({username}) {
  return (
    <UserProvider username={username}>
      <UserInfo />
      <UserNav />
      <UserActivity />
    </UserProvider>
  )
}

function UserSettings() {
  // this would be the associated hook for the AuthenticationProvider
  const {user} = useAuthenticatedUser()
}
```

请看，每个页面都可以有自己的provider，其中包含其下组件所需的数据。代码拆分对于这些也“有效”。如何将数据导入到各个 provider 这取决于 provider 的 hooks 怎么用，以及在应用中怎么用这些数据，你应该是知道（在provider）数据怎么用的。

有关此 colocation 为何有益的更多内容，可以移步 [State Colocation will make your React app faster](https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster) 和 [Colocation](https://kentcdodds.com/blog/colocation) 的文章，想了解更多关于 context的内容，可以阅读 [How to use React Context effectively](https://kentcdodds.com/blog/how-to-use-react-context-effectively)。

## Server Cache vs UI State（服务端缓存和前端状态）

我想补充的最后一件事，状态有各种类别，但每种类型的状态都可以归入以下两个两种：

1. Server Cache - 状态实际上存在服务端，保存一份在客户端是为了加速访问（比如用户个人数据）。
2. UI State - 该种状态仅在 UI 中用于控制应用的交互部分的状态（比如 modal 的 `isOpen` 状态）。

当我们会错误的把两者结合一起，Server Cache 在本质上与 UI state 存在不同的问题，因此，需要分开处理。如果你也认同这个事实，就是这个 state 并不是真正的 state，只不过是 state 的缓存，然后你可以开始思考它有没有用对，从而正确的使用它。

你当然可以使用你自己的 `useState` 或 `useReducer` 同 `useContext` 来管理状态，言归正传，缓存化是个很麻烦的地方（有种说是计算机科学中最复杂的部分之一），明智的做法是站在巨人的肩膀去使用一个成熟的封装。

这也是为什么我会推荐 [react-query](https://github.com/tannerlinsley/react-query) 来管理这部分的状态。我知道，我之前说过并不需要一个状态管理库，但是我真的不认为 react-query 是一个状态管理库（state management library），我认为它是一个缓存（cache），十分棒的那个，看一看[Tanner Linsley](https://twitter.com/tannerlinsley)的推特。

## What about performance? （性能如何呢？）

当你遵循上面的建议，那么性能是最后一个可能遇到的问题，特别是当你正在实践 [the recommendations around colocation](https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster)。然后，其中肯定存在有性能问题的用例，当你遇到了状态相关的性能问题，首先要去检查，当状态发生变化的时候，有多少组件会被重新渲染，以及那些组件是否需要在状态变化的时候重新渲染，如果它们需要，则这个性能问题并不是状态管理机制的问题，但是，你需要的是加快渲染速度，可以看这 [speed your renders](https://kentcdodds.com/blog/fix-the-slow-render-before-you-fix-the-re-render)。

但是，如果你注意到有很多组件在 DOM 没有更新或没有发生副作用的情况下进行了渲染，则这些组件正在不必要地渲染，这情况在 React 中一直都在发生，通常本身不是问题（你需要的是保持关注在 [focus on making even unnecessary re-renders fast first](https://kentcdodds.com/blog/fix-the-slow-render-before-you-fix-the-re-render)），如果这确实是瓶颈，如下是一些解决方案，使用了 React context：

1. 分离状态到不同的业务快，而不是放在一起，这样的话，部分状态的更新不会触发让所有组件都更新的事件。
2. 优化状态的 provider，即 [Optimize your context provider](https://kentcdodds.com/blog/how-to-use-react-context-effectively)
3. 使用 [jotai](https://github.com/react-spring/jotai)

这里是另外一个推荐使用的库，确实，有些用例用 React 的内置状态管理抽象不太适合。在所有可用的抽象中，jotai是这些用例中最有前途的。如果你很好奇这些用例是什么，jotai把这些完美解决的问题类型列在 [Recoil: State Management for Today's React - Dave McCabe aka @mcc_abe at @ReactEurope 2020](https://www.youtube.com/watch?v=_ISAA_Jt9kI)。Recoil 和 jotai 十分类似（解决了同样类型的问题），但是基于我（有限的）经验，我推荐 jotai。

无论如何，大多数应用程序都不需要像 Recoil 或 jotai 这样的基于原子状态管理工具。

## 结论

再说下，你可以在 class component 实现同样的效果，并不是必须要用 hooks，只是说hooks 使用起来更加简单，但是你可以用 React 15 来实现这个理念，尽可能把状态放在组件近的地方使用，当有 prop drilling 传递的问题的时候才使用 context，这么做将使状态的交互更加容易维护。

> 💿 别忘了看看 [Remix: The Yang to React's Yin](https://kentcdodds.com/blog/remix-the-yang-to-react-s-yin) ☯


