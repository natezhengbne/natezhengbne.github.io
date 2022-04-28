# 使用 React Testing Library 的一些常见错误

> This is a translation of the original post [Common mistakes with React Testing Library](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library) by [Kent C. Dodds](https://kentcdodds.com/) on May 4th, 2020.

> 原文来自于[Kent C. Dodds](https://kentcdodds.com/)发布于2020年5月4日的文章 [Common mistakes with React Testing Library](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library).

大家好，我对于当时的测试生态不太满意，所以写了个测试库 [React Testing Library](https://github.com/testing-library/react-testing-library), 它扩展于 DOM Testing Library，然后现在我们Testing Library系列拥有针对现在所有流行的Javascript框架和测试工具（基于DOM，甚至有些不是）的实现。

随着时间推移，我们对 API 进行了一些小修改，并且发现了一些使用非最优解的实现模式（suboptimal patterns）。尽管，我们努力将“better way”使用方法来文档化，但我仍旧看到按照次优模式（suboptimal patterns）编写的测试代码在一些博客和其他地方，我想通过其中的一些，来解释为什么不是很好，以及如何来改进，从而避免这些陷阱。

提示: 按照重要性来标识:
-   低：这仅仅是我个人的想法，如果无所谓的，你可以忽略。
-   中：这可能会导致bug或者无效的工作。
-   高：一定要参考这个建议！因为你很可能将会编写有问题的测试代码。

## 没有使用 Testing Library ESLint 插件

> 重要度：中

使用如下官方的 ESLint 插件可以帮你避免类似很多的常见错误：

-   [eslint-plugin-testing-library](https://github.com/testing-library/eslint-plugin-testing-library)
-   [eslint-plugin-jest-dom](https://github.com/testing-library/eslint-plugin-jest-dom)

> 如果使用 create-react-app 来创建工程，`eslint-plugin-testing-library`已经导入在依赖中

**建议：安装并使用 ESLint 插件**

## 使用了 `wrapper` 作为 `render` 对象的返回值的变量

> 重要度：低


```js
// ❌
const wrapper = render(<Example prop="1" />)
wrapper.rerender(<Example prop="2" />)

// ✅
const {rerender} = render(<Example prop="1" />)
rerender(<Example prop="2" />)
```

我们不需要 `wrapper` 这个变量命名，这只是来自于之前 `enzyme` 的定义，`render` 对象返回的内容并没有被 `wrapping`，即包装。它返回的只是一些公共方法集合，而且并不是频繁的会用到。

**建议：可以通过结构来获取需要的方法或者命名为 `view` **

## 使用了 `cleanup`

> 重要度：中

```js
// ❌
import {render, screen, cleanup} from '@testing-library/react'

afterEach(cleanup)

// ✅
import {render, screen} from '@testing-library/react
```

已经很长一段时间，大多数的测试框架都支持自动 `cleanup`，所以不用再担心

**建议：不用手动 `cleanup` **

没有使用 `screen` 对象

> 重要度：中

```js
// ❌
const {getByRole} = render(<Example />)
const errorMessageNode = getByRole('alert')

// ✅
render(<Example />)
const errorMessageNode = screen.getByRole('alert')
```

在 [DOM Testing Library v6.11.0](https://github.com/testing-library/dom-testing-library/releases/tag/v6.11.0) 已经添加了 `screen`，也就是说只要 `@testing-library/react@>=9` 就可以了。可以通过和 render 一样的方式解构获取，例如：

```js
import {render, screen} from '@testing-library/react'
```

使用 `screen` 的好处是不需要在调用 `render` 的时候反复去加减需要声明的解构的方法，你只需要导入 `screen`，然后编辑器会完成后面的注入。

唯一的例外是，如果是结构了 `container` 或者 `baseElement` 这两个本应该避免的方法（老实说，我也想不到任何合理来使用这两个方法的理由，因为它们只是历史遗留的方法）。

**建议：使用 `screen` 来查询和调试元素**

## 使用了错误的断言

> **重要度：高**

```js
const button = screen.getByRole('button', {name: /disabled button/i})

// ❌
expect(button.disabled).toBe(true)
// error message:
//  expect(received).toBe(expected) // Object.is equality
//
//  Expected: true
//  Received: false

// ✅
expect(button).toBeDisabled()
// error message:
//   Received element is not disabled:
//     <button />
```

`toBeDisabled` 来源于 `jest-dom`， 强烈推荐使用 `jest-dom` 中提供的方法， 这样可以使错误信息更直接。

**建议：安装并使用 [`@testing-library/jest-dom`](https://github.com/testing-library/jest-dom#tobedisabled)**

## 没必要的使用了 `act`

> 重要度：中

```js
// ❌
act(() => {
  render(<Example />)
})

const input = screen.getByRole('textbox', {name: /choose a fruit/i})
act(() => {
  fireEvent.keyDown(input, {key: 'ArrowDown'})
})

// ✅
render(<Example />)
const input = screen.getByRole('textbox', {name: /choose a fruit/i})
fireEvent.keyDown(input, {key: 'ArrowDown'})
```

我看到有人会用像上面的错误例子展示的那样来使用 `act`，因为他们一直看到 `act` 发出的警告信息，然后拼命地尝试，想去解决这些警告，但是他们不知道的是，比如 `render` 和 `fireEvent` 这两个方法其实在内部已经被 `act` 所包装。

大多数时候，你看到了一个关于 `act` 的警告，这不是代表可以置之不理，实际上，是在告诉你一些非期望，或多余的用法存在你测试中。可以从 [Fix the "not wrapped in act(...)" warning](https://kentcdodds.com/blog/fix-the-not-wrapped-in-act-warning) 这篇文章了解更多。

**建议：了解 `act` 的使用场景，避免非必要的使用**

## 错误的使用了元素的查询方法

> 重要程度：高

```js
// ❌
// assuming you've got this DOM to work with:
// <label>Username</label><input data-testid="username" />
screen.getByTestId('username')

// ✅
// change the DOM to be accessible by associating the label and setting the type
// <label for="username">Username</label><input id="username" type="text" />
screen.getByRole('textbox', {name: /username/i})
```

在 ["Which query should I use?"](https://testing-library.com/docs/guide-which-query) 这里，列出了查询方法和使用的顺序。如果你编写测试的目标与我们一致，那么你会对应用有足够的信心，当用户在使用应用时其可以正常工作，那么尽可能的使用基于 DOM 的查询方法，因为那样更加贴近于你的最终用户如何使用应用。这些查询方法可以帮助你达成这个目的，但是并不是所有的都有同样的效果。

### 使用了 `container` 来查询所有的元素

作为一个子章节，我想直接说说 container 的使用场景

```
// ❌
const {container} = render(<Example />)
const button = container.querySelector('.btn-primary')
expect(button).toHaveTextContent(/click me/i)

// ✅
render(<Example />)
screen.getByRole('button', {name: /click me/i})
```

我们想确保你的用户和UI能够正确的交互，如果你到处使用 `querySelector`，则会造成测试代码难以阅读，以及更多的失败，这些会在下个小节体现：

### 没有使用基于字符来查询

同样作为一个子章节，我想谈谈我推为何荐大家在所有地方使用实际的字符来进行元素的查询（在需要实现本地化的情况，建议使用默认地域配置），而不是 test IDs 或其他方式。

```js
// ❌
screen.getByTestId('submit-button')

// ✅
screen.getByRole('button', {name: /submit/i})
```

如果不是根据实际的文本内容来查询，那你不得不进行额外的工作来确保转化的过程是正确的，但是我也听过对此的抱怨是，其他人更改了文本内容会导致你的测试单元被破坏。我首先需要反驳的是，如果任何开发人员将 “Username” 更改为 “Email”， 我相信我有理由知道，因为我也需要去对应修改我的具体实现。此外，如果在某种情况下，发生了测试用例被破坏的情况，也会是十分容易去分类和修复的，不需要太多时间。

所以，修复的成本是很低的，好处是测试用例将会轻松覆盖到需要转换的组件，并且这也会是测试代码更加容易编写和阅读。

我在这里也提到，不是每个人都会同意我的观点，请随时在推文中[in this tweet thread](https://twitter.com/kentcdodds/status/1203179007644012544)阅读更多的信息。

### 没有大多数时候首先使用  `*ByRole` 
同样作为一个子章节，这里我想谈谈 `*ByRole`。最近的一些版本，`*ByRole`的查询方法很认真的进行了改进（这里十分感谢 [Sebastian Silbermann](https://twitter.com/sebsilbermann) 做出的努力），现在它们是查询组件的推荐的首选方法。下面是我最喜欢的功能。

`name` 选项允许你通过 **“Accessible Name”**（用来支持视觉障碍的用户访问页面） 来查询，甚至在你的文本内容被划分到了不同的地方，比如：

```js
// assuming we've got this DOM structure to work with
// <button><span>Hello</span> <span>World</span></button>

screen.getByText(/hello world/i)
// ❌ fails with the following error:
// Unable to find an element with the text: /hello world/i. This could be
// because the text is broken up by multiple elements. In this case, you can
// provide a function for your text matcher to make your matcher more flexible.

screen.getByRole('button', {name: /hello world/i})
// ✅ works!
```

其中一个原因大家不使用 `*ByRole` 是因为，不熟悉元素上无障碍属性的角色，可以参考：[Here's a list of Roles on MDN](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles)。另外一个我很喜欢的地方是，如果当我们无法查询到对应角色的元素的时候，`*ByRole`不会像普通的 `get*` 或者 `find*` 返回整个DOM结构，只是会返回当前你可以到的所有的可用的角色。

```js
// assuming we've got this DOM structure to work with
// <button><span>Hello</span> <span>World</span></button>
screen.getByRole('blah')
```

当无法找到符合要求的元素时，返回的错误信息如下：


```js
TestingLibraryElementError: Unable to find an accessible element with the role "blah"

Here are the accessible roles:

  button:

  Name "Hello World":
  <button />

  --------------------------------------------------

<body>
  <div>
    <button>
      <span>
        Hello
      </span>

      <span>
        World
      </span>
    </button>
  </div>
</body>
```

可以注意到，我们不用显式地将 `role=button` 添加到按钮中来让其获得button这个角色。因为这是一个隐含的角色，可以完美的引领我们去下一章节。

**建议：阅读 [The "Which Query Should I Use" Guide](https://testing-library.com/docs/guide-which-query) 并根据文章中的推荐方法来执行**

## 错误的添加了，例如 `aria-`，`role` 或其他一些 **accessibilty** 属性

> 重要性：高

```
// ❌
render(<button role="button">Click me</button>)

// ✅
render(<button>Click me</button>)
```

比如上面的例子，胡乱的添加辅助阅读属性不仅是没要的，而且会给屏幕阅读器和阅读器用户带来困惑。应该在只有HTML的语义无法满足你的用例的时候，才来使用 accessibility，比如你正在构建一个非原生的UI，但是你希望其可以像 [autocomplete](https://github.com/downshift-js/downshift) 组件来访问。如果那是你正在做的，请务必使用一个现成的组件库来实现可访问性或者遵循WAI-ARIA标准（这里有一些[例子](https://www.w3.org/TR/wai-aria-practices/examples/accordion/accordion.html)）

> 注意：要使得 `input` 来通过 `role` 可访问，你需要另外指定 `type` 属性

**建议：避免添加没必要的或者错误添加 accessibility 属性**

## 没有使用 `@testing-library/user-event`

> 重要性：中

```
// ❌
fireEvent.change(input, {target: {value: 'hello world'}})

// ✅
userEvent.type(input, 'hello world')
```

`@testing-library/user-event` 是一个建立在 `fireEvent` 之上的包，它提供了几个更加契合用户交互行为的方法。比如上面的例子，`fireEvent` 将会在用户进行输入的时候触发 `change event`，然而在这个 `type` 调用，在每个字符输入的时候都将会触发 `keyDown` `keyPress` 和 `keyUp` 这些事件。这更加接近于用户的真实行为。这样的好处在于如果你使用的库没有监听 `change event`，也依旧不会收到影响。

我们依旧在更新 `@testing-library/user-event` 来确保，它能够在用户执行特定操作时会触发所有相同的事件。我认为我们还没有完全实现，所以还没有合并到 `@testing-library/dom` 中（不过可能时未来某个时候）。但是我也有足够的信心推荐你看下这个包，并且更多的使用它，胜过使用 `fireEvent`。

**建议：在任何之前使用 `fireEvent` 的地方使用 `@testing-library/user-event`**

## 对检查任何不存在的内容之外的功能使用了`query*`

> 重要性：高

```
// ❌
expect(screen.queryByRole('alert')).toBeInTheDocument()

// ✅
expect(screen.getByRole('alert')).toBeInTheDocument()
expect(screen.queryByRole('alert')).not.toBeInTheDocument()
```

*唯一*使用 `query*` 相关查询的原因是，如果没有任何的元素匹配到查询，你调用的这个函数不会报错（如果没有找到，返回 `null` ）。这里*唯一*有用的地方在于验证元素没有被渲染到页面上。这里的概念如此重要，因为 `get*` 和 `find*` 在没有元素匹配时会抛出异常，并打印出当前渲染的文档结构，这样你可以检查哪些元素已经渲染，以及为什么你的查询无法查到到条件中的内容。而 `query*` 只会返回 `null`，所以 `toBeInTheDocument` 也就只可以打印 “null isn't in the document”，对排查过程没有什么帮助。

**建议：仅使用 `query*` 来断言检查元素没有存在**

## 使用了 `waitFor` 去等待元素查找

> 重要性：高

```
// ❌
const submitButton = await waitFor(() =>
  screen.getByRole('button', {name: /submit/i}),
)

// ✅
const submitButton = await screen.findByRole('button', {name: /submit/i})
```

这两段代码基本是等效的（`find*`查询会在其内使用`waitFor`），但是第二种逻辑更加简单，并且报错信息更加友好。

**建议：任何时候，直接使用 `find*` 来进行对于无法立即返回结果的内容的查询**

## 传了没有返回值的回调传给 `waitFor`

> 重要性：高

```
// ❌
await waitFor(() => {})
expect(window.fetch).toHaveBeenCalledWith('foo')
expect(window.fetch).toHaveBeenCalledTimes(1)

// ✅
await waitFor(() => expect(window.fetch).toHaveBeenCalledWith('foo'))
expect(window.fetch).toHaveBeenCalledTimes(1)
```

使用 `waitFor` 的目的是可以让你等待特定的事情发生。如果你传了一个空回调，可能今天正常，因为你现在只需要的是等待 “one tick of the event loop”，但是你会在这里留下一个不够健壮的测试，未来当你重构异步逻辑时将十分容易被影响而测试失败。

**建议：在 `waitFor` 定义一个assertion来包装具体要的等待**

## 在一个 `waitFor` 回调内有多个 assertions

> 重要性：低

```
// ❌
await waitFor(() => {
  expect(window.fetch).toHaveBeenCalledWith('foo')
  expect(window.fetch).toHaveBeenCalledTimes(1)
})

// ✅
await waitFor(() => expect(window.fetch).toHaveBeenCalledWith('foo'))
expect(window.fetch).toHaveBeenCalledTimes(1)
```

让我们看看上面的例子，`window.fetch` 被调用了两次，所以 `waitFor` 的调用会报错，然后，我们将不得不等到超时后才能看到报错。通过只在 `waitFor` 中放置单个assertion，我们可以等待期待的UI加载完毕，并且还可以让那个会失败的断言更快的失败，而不用等待超时。

**建议：在回调方法中仅放一个 assertion**

## 在 `waitFor` 中执行 side-effects

> 重要性：高

```
// ❌
await waitFor(() => {
  fireEvent.keyDown(input, {key: 'ArrowDown'})
  expect(screen.getAllByRole('listitem')).toHaveLength(3)
})

// ✅
fireEvent.keyDown(input, {key: 'ArrowDown'})
await waitFor(() => {
  expect(screen.getAllByRole('listitem')).toHaveLength(3)
})
```

`waitFor` 适用于你运行的实际的逻辑和assertion有非确定性时间的情况，因此，回调会被调用（或检查错误）不确定的次数和频率（当DOM变化的间隔的时候两者都会被调用），所以，这意味着 side-effects 会被运行多次。

所以，你不能在snapshot生成的逻辑中使用 waitFor，如果你确实是需要使用snapshot，则先等待 assertion 完成，然后那之后再获取snapshot。

**建议：将 side-effects 放在 `waitFor` 之外，并仅保留assertions给回调**

## 使用了 `get*` 开头的查找方法而替代了 assertions 的作用

> 重要性：低

```
// ❌
screen.getByRole('alert', {name: /error/i})

// ✅
expect(screen.getByRole('alert', {name: /error/i})).toBeInTheDocument()
```

这个实际上也没什么大不了的，但是我想我应该提出来并会给出我的观点。如果 `get*` 查询无法成功找到该元素，它将抛出一条非常有用的错误消息，并显示完整的 DOM 结构（带有语法高亮），这将在调试期非常有用。因此，assertions 可能永远不会失败（因为异常会提前抛出而跳出 assertions 的逻辑）。

所以，许多人会跳过 assertion，不得不说，这也是可以的，但是我个人我会写 assertions，来明确表达检查的意图，这样可以更好的把这个信息传递个给阅读者，并不是让其以为这是在重构后让遗弃的一个查询。

**建议：如果你需要 assert 来测试它存在，则声明它**

## 结论

作为testing library系列的维护者，我们尽最大努力来维护API，让大家用起来更加顺手，如果有存在不足，我们会记录下来。但这有时可能非常困难（特别是当API修改和更新等），希望这对你有所帮助，真心希望你能充满信心地成功交付产品。

Good luck!




















