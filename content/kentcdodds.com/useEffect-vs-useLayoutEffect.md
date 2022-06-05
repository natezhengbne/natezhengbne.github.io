# useEffect vs useLayoutEffect

> This is a translation of the original post [useEffect vs useLayoutEffect](https://kentcdodds.com/blog/useeffect-vs-uselayouteffect) by [Kent C. Dodds](https://kentcdodds.com/) on December 1st, 2020.
>
> 原文来自于[Kent C. Dodds](https://kentcdodds.com/)发布于2019年12月01日的文章 [useEffect vs useLayoutEffect](https://kentcdodds.com/blog/useeffect-vs-uselayouteffect).

这两个 React 的 hook 都可以用来做基本上是相同的事，但是在使用场景上会有稍许不同，所以这里有一些规则来告诉你何时使用对应的 (React Hook)[https://reactjs.org/hooks]。

## useEffect

大部分时间，基本上99%的时间你都是用这个方法，如果你想把你的类组件进行重构以便使用 hook，你将很可能需要把 `componentDidMount` `componentDidUpdate` 或者 `componentWillUnmount` 的代码转移到 `useEffect`来。

**The one catch** (一个条件)会在 React 渲染你的组件后运行，以确保你的回调方法不会阻塞浏览器的绘制过程，这个与类组件中 `componentDidMount` 和 `componentDidUpdate` 在渲染后同步运行的行为有所不同，这样的性能更好。

然而，如果是修改DOM（通过一个DOM的节点引用），并且这个DOM的变更，在其被渲染和你去进行修改的时间内去改变DOM节点的表现，那么就不适合使用 `useEffect`， 而应该使用 `useLayoutEffect`，否则的话，在DOM变更生效的时候，用户会看到页面存在闪烁，这几乎是唯一一次你需要使用 `useLayoutEffect` 而不是 `useEffect` 的时候。

## useLayoutEffect

在React处理完所有的DOM变更后，useLayoutEffect会立即同步运行，当你需要获取DOM的尺寸（比如获取元素的鼠标滚动的位置或者其他一些样式）接着去进行DOM修改或者根据更新的状态去触发一个同步的重渲染。

就调度的顺序而言，这个工作方式与 `componentDidMount` `componentDidUpdate` 相同，在DOM完成更新后你的代码会被立即运行，但是会在浏览器有机会去“绘制”那些变化之前（在浏览器重新绘制之前用户是看不到的）。

## 概要

- useLayoutEffect：在你需要改变DOM，或者进行元素尺寸的测量
- useEffect：如果你不需要与DOM进行交互或者你的DOM的改变是不可观察的（多数情况，你应该用这个）。

## 一个特例

当你更新一个值（比如，元素的ref），并且你想确保在其他代码运行前它是最新的，那么这种情况下，你可能需要使用 `useLayoutEffect` 代替 `useEffect`。

```js
const ref = React.useRef()
React.useEffect(() => {
  ref.current = 'some value'
})

// then, later in another hook or something
React.useLayoutEffect(() => {
  console.log(ref.current) // <-- this logs an old value because this runs first!
})
```

在上面这个例子，应该使用 `useLayoutEffect`。

## 结论

这主要是跟浏览器的默认行为有关，默认的行为是，浏览器在React运行你的代码之前基于DOM的更新来重新绘制，这意味着你的代码不会阻塞浏览器，并且用户可以更快的看到DOM
的更新，因此，大多数时候应该坚持使用 `useEffect`。