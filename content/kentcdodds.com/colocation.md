# Colocation


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09e842679200480983ebc921559e4fe7~tplv-k3u1fbpfcp-watermark.image?)

> This is a translation of the original post [Colocation](https://kentcdodds.com/blog/colocation) by [Kent C. Dodds](https://kentcdodds.com/) on July 17th, 2019.

> 原文来自于[Kent C. Dodds](https://kentcdodds.com/)发布于2019年7月17日的文章 [Colocation](https://kentcdodds.com/blog/colocation).

我们都希望拥有易于维护的代码库，因此我们以这个作为最好的目标开始，使我们的代码库（或代码库的角落）可维护且易于理解。随着时间的过去，伴随着代码的增长，工程的管理（JS，CSS，图片等等）变得越来越困难，随着项目的发展，越来越多的代码变成了 “部落知识”（只有你或者其他几个人知道的知识），而这种知识会导致“技术债务”（无论这个术语是[否](https://twitter.com/ryanflorence/status/747983065738153985)准确）。

我习惯让我的代码库不仅仅可以我自己（代码的作者）维护，也包括团队的其他人，或者未来其他的人，甚至是6个月后的我，我相信我们都会为了代码而同意这个好主意来而努力，我们可以使用许多不同的工具和技术来实现这一目标。

## Let's talk about Code Comments（谈谈代码的注释）

我不想讨论是否需要对代码进行注释（你应该注释），以及应该写什么注释（你在注释中解释了你正在做的一些意想不到的事情，那么后来的人就能够理解那些导致意外或者奇怪的代码的决策），（好吧，也许我想稍许谈谈这个），相反，我想把重点放在这些代码注释的位置。我们通常将这些注释与它们所解释的代码“co-location 放在一起”，使其尽可能接近相关代码。

考虑一下，如果我们以不同的方式做到这一点，如果我们把这些注释放在一个完全独立的文件中呢？一个巨大的 `DOCUMENTATION.md` 文件或可能一个 `docs/` 目录映射到 `src/` 目录，听起来很有趣吗？不，对我来说也不是，如果不能将注释与它所解释的代码放在一起，我们会遇到一些严重的问题。

- 可维护性 Maintainability：它们将会更快的因为没有同步更新而过时不可用，我们会移动或者删除某个 `src/` 下的文件，而不会同步更新对应的 `docs/` 文件。
- 实用性 Applicability：人们在查看 `src/` 的代码时可能会忽略一个在 `docs/` 的很重要的注释，或者因为不知道有一个对应 `src/` 的 `docs/` 注释文件，没有去添加对应代码的注释。
- 上手容易的问题 Ease of use：从一个位置切换到另一个位置的上下文对于这种设置来说也是一个挑战，必须处理多个文件在多个不同的位置，可能会难以确保进行了完成组件所需的一切。

我们绝对可以为这种代码注释风格提出一个规范，但是我们为什么要这样做呢？将注释与它们所解释的代码放在一起不是更简单吗？

## So-what? （那又如何？）

你可能回想：是的，这就是为什么没有人把注释写到 `docs/`，并且所有人都把注释写在需要注解的代码旁边，这很明显，你的观点是什么？我的观点是，co-location 的好处无处不在。

## HTML/View

以HTML为例子，将注释以 co-location 的形式放置的好处也可以将注释转为模板，在出现的现代的框架，比如React之前，你需要把视图的模板和逻辑放在不同的文件，也会有备受上面的问题的折磨，如今，在 React 和 VUE 中会把这些放在同一个文件中，对于 Angular，如果没有在同一个文件，但模板文件至少也会放在关联的JS文件旁。

## CSS

另一个适应于此的便是CSS了，我不是和你争论CSS-in-JS（非常优秀的库）的优点，看[这里](https://medium.com/seek-blog/a-unified-styling-language-d0c208de2660)了解更多的优点。

## Tests

对于单元测试来说这个概念同样适用，是不是很常见，比如一个项目包含 `src/` 和 `test/` 目录并且测试文件试图镜像到对应的 src/ 目录，上述所有陷阱也适用于这里，我可能不会把单元测试放在完全相同的文件中，但我也不完全排除这是一个有趣的想法（可以留给大家思考）。

为了让代码的维护性更加优异，我们应该把测试文件与它需要去测试的文件和文件组放在尽可一起，这确保了新人（或者6个月内的我）看到这份代码，他们能够很快就看到这些模块已经被测试了，并且可以通过这些测试逻辑来了解代码逻辑，当修改代码的时候，测试用例也会被提示需要更新（或者新增/删除/修改）。

## State

应用或者组件的状态同样适用，与使用状态的 UI 不相关/间接距离越大，维护它就越困难。本地化状态有更多好处，它还提高了应用程序的性能。与位于组件顶部的状态变化相比，在应用的组件下的状态变化会触发更少的组件重渲染，请本地化你的状态。

## "Reusable" utility files 可重用的工具文件

这也同样适用于通用的文件和功能，可以想象当你在写一个组件，看到一段很好的代码，可以被提取到公共的方法中，你这么做了并且想，我打赌很多人将会用到这个方法，所以你把方法放到了应用中的 `utils/` 目录。

过后，你的组件被删除了，但是这个通用的功能将会被无视，它会存在（以及它的测试），因为删除它的人认为这个功能被其他地方使用，年复一年，工程师们努力工作，以确保该功能及其测试用例继续正常运行，甚至没有意识到已经不再需要它了，这多么浪费精力。

相反，如果您只是将该函数直接留在使用它的文件中，则将完全不同，我不是说不要去担心那些复杂的通用函数，而是尽可能的将他们放在需要的地方，这样可以少些问题。

> 请删除这个[ESLINT RULE](https://github.com/yannickcr/eslint-plugin-react/blob/e6b4c33a1db4cc94c3e9223b09fb92b1dbddc00d/docs/rules/no-multi-comp.md)，以及类似的

## The principle 原则

co-location 的概念可以归结为以下这个基本原则：

> 将代码尽可能靠近其相关的位置

你可能会说：“Things that change together should be located as close as reasonable. 共同变化的事物应该尽可能接近。”。（[Dan Abramov](https://twitter.com/dan_abramov)也曾静对我这么说过）。

## Open Source made easy(-er) 更加容易转为开源组件

除了避免上面讨论的问题，以这种方式构建项目还有其他好处。把一个组件转为开源项目，就像将文件夹复制/粘贴到另一个项目并将其发布到npm一样简单，然后你可以安装它，然后使用 ’require‘/’import‘ 导入即可。

## Exceptions

当然，对于跨越整个或部分系统的文档以及事物如何集成在一起的文档来说，有一个很好的论据。你会把跨组件的集成功能或端到端测试放在哪里？可能你会想这是个特殊情况，但他们实际上可以很好的实现上面的原则。

如果我的应用有一部分与用户身份验证相关联，并且我想记录该功能的流程，则可以将 README.md 文件放在包含与用户身份验证关联的所有模块的文件夹中。如果我需要为该功能编写集成测试，我可以将这些测试的文件放在同一文件夹中。

说到 end-to-end 测试，放在项目的根目录也是很合理的，它们超越了项目本身，进入了系统的其他部分，所以对我来说，将它们放在一个单独的目录中是有意义的，有时他们没有完整映射到 'src/' 下的文件，实际上，E2E 测试不关注 'src/' 下的文件是如何组织的，重构和移动文件不会需要去修改 E2E 测试。

## 结论

我们的目标是尽可能轻松的来维护项目，这样我们可以这样放置代码的注释，或其他文件来获得可维护性/适用性，以及易用的好处。如果你还没有试过，我建议你试试。

另外，如果你关注于违反 "separation of concerns" （分离关注），我建议你听听(Peter Hunter)[https://twitter.com/floydophone]的(这个)[https://youtu.be/x7cQ3mrcKaY]视频，来重新评估具体是什么意思。

我也需要再次提到，这也使用于图片和其他的资源文件，当你使用 webpack 的时候，将资源 co-locating 也十分容易，实话说，这也是 webpack IMO 核心价值的主张之一。
