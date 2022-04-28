# How I built a modern website in 2021

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fd9440de79c54ea9aa7015791d1cf0e7~tplv-k3u1fbpfcp-watermark.image?)

> This is a translation of the original post [How I built a modern website in 2021](https://kentcdodds.com/blog/how-i-built-a-modern-website-in-2021) by [Kent C. Dodds](https://kentcdodds.com/) on September 29th, 2020.

> 原文来自于[Kent C. Dodds](https://kentcdodds.com/)发布于2021年9月29日的文章 [How I built a modern website in 2021](https://kentcdodds.com/blog/how-i-built-a-modern-website-in-2021).

在2021年大半年的时间里，我完全重写了我的个人网站 kentcdodds.com。你现在浏览完成后的网站，有没有试试 dark 和 light 模式，有没有注册账号并且选择一个小队加入？你有没有试着参加 [the Call Kent Podcast](https://kentcdodds.com/calls)。这篇文章是来讲述我是如何构建它的，而不是它有什么新功能或者其他，这里有太多的内容值得深入讨论，这在一篇博客内无法详尽，我尽可能的大致上介绍下构建这个网站所使用的技术和第三方库。

如果你还没有开始，请先阅读网站的[介绍](https://kentcdodds.com/blog/introducing-the-new-kentcdodds.com)，可以从更高的层次来了解这个网站对于一个软件工程师，比如你，会带来哪些学习体验和提升。

## 来龙去脉

在我们开始深入之前，我想明确一件事。如果这里只是一个开发人员的简单的内容博客网站，我会表示同意，建站的技术选择是过度工程化了。然后，我想去构建一个在此之上并且可以说是超越的网站，所以我必须要深思熟虑后做出架构设计的决策。我在这里所完成的内容绝对不是说用wordpress加CDN就能搞定的。

如果你只是个人新手打算去建立你的个人网站，那这里肯定不是参考的地方，若我要建立一个简单的个人站点，那我会直接用 [Remix.run](https://remix.run)，那我大概率会将它部署在netlify的serverless服务上，而且内容采用remix已经内置支持了的markdown格式，这将简单的多。

但是，再一次，这不是kentcdodds.com所表达的。你对如何使用现代工具有效地建立一个可维护的网站感兴趣，那么请继续。

在下面的视频也可以去了解这个网站还能够做什么：

https://youtu.be/7H2VsJb8LgY

## 综合的统计概览

首先，让你们先了解下这里谈到的网站规模，这里有一些统计：

在2021年10月，也就是写这篇文章的时候，运行 [cloc](https://github.com/AlDanial/cloc)的结果：

```
$ npx cloc ./app ./types ./tests ./styles ./mocks ./cypress ./prisma ./.github
     266 text files.
     257 unique files.
      15 files ignored.

github.com/AlDanial/cloc v 1.90  T=0.16 s (1601.9 files/s, 194240.7 lines/s)
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
TypeScript                     219           2020            583          21582
CSS                             10            198            301           4705
JSON                             7              0              0            609
YAML                             2             43             13            232
SQL                              7             20             25             52
JavaScript                       4              2              3             42
Markdown                         1              0              0              2
TOML                             1              0              2              1
-------------------------------------------------------------------------------
SUM:                           251           2283            927          27225
-------------------------------------------------------------------------------
```

来看看网站关于内容方面的数量，这里有一个字数统计：
```
$ find ./content -type f | xargs wc -w | tail -1
  280801 total
```

这个数量比哈利波特前三本的总字数都多。

加上四个季度 [the Chats with Kent Podcast](https://kentcdodds.com/chats) 博客的总播放量，包括了35个小时的内容，以及全新的节目 [Call Kent Podcast](https://kentcdodds.com/calls) 的3小时内容，并且时长还在不断增加，顺便说一句，这也已经超过了 Jim Dale 的阅读哈利波特的节目的总时长（除非你像我一样爱用3倍速#subtlebrag 🙃）。

27k行的代码量，这可不是一个拥有6个人的团队，兢兢业业工作8年后的产出，更加也不像你的人博客那般。这是一个真正的全栈项目，包含了数据库，缓存，用户认证等待，我十分相信当时这个网站是基于Remix的最大的应用。

这些也并不是我一个人完成的。你可以查看 [the credits page](https://kentcdodds.com/credits) 这个页面来查看贡献者的信息。我是最主要的贡献人，并且做出整个网站架构设计的决策（当然也包含错误？时间会说明一切 😅）。

第一次提交在2020年11月，大部分的内容在最近的4-5个月完成。

这是到目前为止~945次提交。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f4bc81454d234ed9b947d2b28bb7e3fb~tplv-k3u1fbpfcp-watermark.image?)

## 技术选型

下面是项目中的主要技术选型（排名不分线后）：

-   [React](https://reactjs.org/): For the UI
-   [Remix](https://remix.run/): Framework for the Client/Server/Routing
-   [TypeScript](https://www.typescriptlang.org/): Typed JavaScript (necessary for any project you plan to maintain)
-   [XState](https://xstate.js.org/): State machine tool making complex component state management simple
-   [Prisma](https://www.prisma.io/): Fantastic ORM with stellar migrations and TypeScript client support
-   [Express](https://expressjs.com/): Node server framework
-   [Cypress](https://cypress.io/): E2E testing framework
-   [Jest](https://jestjs.io/): Unit/Component testing framework
-   [Testing Library](https://testing-library.com/): Simple utilities for testing DOM-based user interfaces
-   [MSW](https://mswjs.io/): Fantastic tool for mocking HTTP requests in the browser/node
-   [Tailwind CSS](https://tailwindcss.com/): Utility classes for consistent/maintainable styling
-   [Postcss](https://postcss.org/): CSS processor (pretty much just use it for autoprefixer and tailwind)
-   [Reach UI](https://reach.tech/): A set of accessible UI components every app needs (accordion/tabs/dialog/etc...)
-   [ESBuild](https://esbuild.github.io/): JavaScript bundler (used by Remix and mdx-bundler).
-   [mdx-bundler](https://github.com/kentcdodds/mdx-bundler): Tool for compiling and bundling MDX for my site content (blog posts and some simple pages).
-   [Octokit](https://github.com/octokit/octokit.js): Library making interacting with the GitHub API easier.
-   [Framer Motion](https://www.framer.com/motion/): Great React Animation library
-   [Unified](https://unifiedjs.com/): Markdown/HTML parser/transformer/compiler system.
-   [Postgres](https://www.postgresql.org/): Battle tested SQL database
-   [Redis](https://redis.io/): In-memory database–key/value store.

下面是项目选用的服务：

-   [Fly.io](https://fly.io/): Super hosting platform
-   [GitHub Actions](https://github.com/features/actions): Hosted CI pipeline service
-   [Sentry](https://sentry.io/): Error reporting service
-   [Cloudinary](https://cloudinary.com/): Fantastic image hosting and transformation service.
-   [Fathom](https://kcd.im/fathom): Privacy-focused ethical analytics service.
-   [Metronome](https://metronome.sh/): Remix metrics service

## 架构设计

### 部署流水线


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55f89aba6fa94101b78c12cf2f1056b1~tplv-k3u1fbpfcp-watermark.image?)

我相信通过描述app的部署方式对整体架构的理解很有指导意义，上图通过 [Excalidraw](https://excalidraw.com/)绘制直观的进行了表现，也请让我通过文字的形式来详述：

首先，我提交了代码，然后推送到Github（open source）。在Github上，有两个Action的服务会在代码push到 `main` 分支后自动运行。

在 Discord 中安装了 GitHub 的 webhook，这样当有任何的成功/失败的构建信息都会推送给discord中定义好的频道，这样，在任何时候，我都能轻松掌握事件进展的情况。

### GitHub Actions: 🥬 Refresh Content（内容刷新）

第一个Github的任务为 “🥬 Refresh Content”， 目的在于当 content 变化的时候进行刷新。在了解它做了什么之前，让我先解释下它解决了什么问题，我的站点kentcdodds.com，之前的版本是基于 Gatsby 的，由于 Gatsby 的 SSG 的特性，当我仅仅想修改content的时候，不得不编译整个站点（这可能需要10-25分钟）。

但是现在的服务器支持 SSG，所以更新 content 则不需要等待一个完整的rebuild，服务端能够通过通过 Github API 来直接访问存储在 Github 上的所有content，这里的问题是，在博客文章的请求上增加了许多额外的开销，再加上编译MDX的时间，你只能得到一个相应很慢的博客，所以我不得不使用了Redis来进行缓存，然后这个问题就变成了缓存无效，即invalidation，我需要确保我对某些content更新后，redis也能够刷新。

那么，这就是第一个Github action的用处。首先，它能够判断在当前使用的版本和最近一次提交的版本是否一致来确定是否需要刷新（这些东西都存在redis，然后服务会暴露一个接口以便action来获取）。如果任何变化的文件来源于 ./content 目录，然后这个action会发出经过了身份验证的post请求来告知所有变化的文件。接着，服务端会通过 Github API 来获取所有的content，然后重新编译 MDX 页面，推送这些内容到Redis，紧接着，Fly.io会自动推送到其他节点。

这将之前10-25分钟的任务时间缩减到了8秒，并且也同样节省了计算资源，当为了修复一些拼写错误的时候不至于需要重新来编译/部署/更新缓存整个站点。

> 我承认使用 Github 来作为 CMS 是有点奇怪，但因此，优秀的贡献者们一直可以通过这样来提供对这些开源内容的维护，所以我会继续这样。（注意每篇博文底部的编辑链接）

### GitHub Actions: 🚀 Deploy （部署）

第二个 Github action 是用来部署，首先，它会决定更改是否需要部署，如果变化的只是content，那么没有理由因为只是刷新content而费心去重新部署网站，在之前旧站的大多数主要的提交都仅仅是content，这样拆分action也更加环保。

一旦我们发现有需要部署的变更，就会并行启动如下几个步骤：

- ⬣ ESLint: 运行EsLint检查错误
- ʦ TypeScript: 检查类型错误
- 🃏 Jest: 运行单元测试
- ⚫️ Cypress: 运行自动化测试
- 🐳 Build: 编译成docker镜像

在自动化测试，即Cypress那步，我进一步的将E2E的测试用例拆分成三分并放入不同的容器，这样可以同时运行这些，节省时间。

> 你可能注意到了，在上面的部署图上，Cypress上没有箭头，这是故意和暂时的。当前，当E2E测试失败并不会导致编译失败退出，迄今为止，我还不曾担心部署过程失败，并且我也不想挂起部署的过程，因为我偶然发现because I accidentally busted something for the 0 users who expect things to be working。而且E2E测试也是整个测试流程中最慢的部分，我希望能够迅速的完成发布，总的来说，我会关注是不是把网站整崩了，但是现在我希望整个部署过程更快的完成，而且我也知道什么时候有错误产生。

等到 ESLint，Typescript，Jest和Build成功完成，接下来我们进入到部署这一步，这块是相当简单的，我使用了 Fly CLI 来部署之前编译好的docker container，到了这里 Fly 会处理剩下的步骤，它会在 Dallas, Santiago, Sydney, Hong Kong, Chennai, and Amsterdam 这些区域依次启动Node服务， 当启动完成，Fly 将会把流量切换到新的服务，然后关闭旧的服务，如果在某个区域启动失败，服务会自动进行回滚。

另外，部署的过程中，使用了 prisma 的 migrate 功能来应用任何增量的数据库更新（migrations，它在 Postgres DB 中存储了上一次的改变内容）。Prisma 在 Dallas 的 Postgrs 的集群上应用变更，接着 Fly 会立即讲变更应用到其他区域的对应服务。

当我运行 `git push` 或者 在 Github 上点击 Merge 按钮的时候，上面的这一切就会发生了😄。

### 数据库的访问关系

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1dd422dfb09b4e599de9f9525d0d8715~tplv-k3u1fbpfcp-watermark.image?)

Fly.io 最酷的一点，就是它可以讲你的应用部署到全世界的不同区域（这也是我选择 Fly 而不是选用 Node 备用服务节点的原因）。我根据之前网站的统计分析，选择了他们提供的6个节点，当然他们可以提供更多。

不过呢，将Node服务部署到多个区域节点只是这个故事的一部分。想要在托管主机上，获得更好的网络性能，你需要将你的数据也部署在节点附近。因此，Fly 也提供 Postgres 和 Redis 集群服务在各个子区域，这意味着一个授权用户在柏林访问 [The Call Kent Podcast](https://kentcdodds.com/calls)，访问会命中离他们最近的服务器，即阿姆斯特丹，同样也是同一个节点内 Postgres 和 Redis 的缓存来提供数据访问，这样，在世界任何地方的访问体验都是一样飞速。

更重要的是，我也不需要在供应商的选定做出权衡，任何时候，我都能搬回家，然后部署到任何支持Docker的地方，这也是我没有选择比如 Cloudflare Workers 或者 FaunaDB 这些服务。另外，我也需要来改造以来符合这些服务商的限制，Fly 很棒，我希望能一直用下去。

这也不是说没有缺点，所有的跨区域部署都会带来数据一致性（consistency）的问题，架构上每个数据库都拥有一样的数据，我也不像根据区域来切割应用，每个数据都要拥有一样的数据，那么如何来做到数据一致性？这样，我会选择其中一个区域作为主区域，然后设置其他的区域的数据库为只读模式，所以，柏林的用户的数据不会写入到位于阿姆斯特丹的数据库，不用担心，所有的Node服务只会建立只读连接到本地区域的数据库来获取数据（最常见的操作），以此保证读是很快的，然后他们会创建写连接到主区域的数据库，一旦写入操作发生的时候，Fly 就会立即自动地将数据变更到其他区域，这个模式工作的十分完美。

Fly 让这种操作变得十分简易，我超爱它，虽然如此，但是这也产生的了个新的问题。

### Fly （数据同步）请求回放

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5304ca9fecb4f26a17fc7d9d28b20ba~tplv-k3u1fbpfcp-watermark.image?)

对于这个多区域部署的独写连接模式的问题在于，如果一个柏林的用户尝试去读取他们刚刚写入的数据，在 Fly 将变更的数据广播下来之前，有可能读取的依旧是旧数据。数据的广播一般来说在毫秒级别完成，但是如果写入的数据过大（比如你[提交](https://kentcdodds.com/calls/record/new)了一段音频到 [ The Call Kent Podcast](https://kentcdodds.com/calls) ），那读取很有可能是延迟的。

避免此问题的一种方法是在完成写入后，其余的读取操作将针对主数据库来执行，不幸的是，这样做会比较复杂。

另外一种是，可以使用 Fly 的 “Replay”，在读写连接都建立在主区域的时候，可以发送这个 “replay” 请求到主区域，你可以发送一个携带 “fly-replay: REGION=dfw” 请求头的请求，接着 Fly 会进行拦截，然后将这个请求重新发送到指定的区域（‘dfw’是达拉斯，也就是我的主区）。

所以在 express app 中有一个中间件来负责将所有的非 get 请求自动 replay，这确实意味着，在柏林的用户的访问将会等待更长的时间来处理，而且，这些操作也不是高频的操作，老实说，我不知道还有没有更好的方案。🙃

我对这个解决方案非常满意。

## 本地开发（Local Development with MSW） 

当我在本地开发的时候，我可以通过 `docker-compose.yml` 来启动本地的 postgres 和 redis 数据库的coker容器。但是我也依旧需要与一些第三方的API打交道，在写这篇文章的时候，也就是2021年9月，app需要与如下的API交互数据：

1.  `api.github.com`
2.  `oembed.com`
3.  `api.twitter.com`
4.  `api.tito.io`
5.  `api.transistor.fm`
6.  `s3.amazonaws.com`
7.  `discord.com/api`
8.  `api.convertkit.com`
9.  `api.simplecast.com`
10.  `api.mailgun.net`
11.  `res.cloudinary.com`
12.  `www.gravatar.com/avatar`
13.  `verifier.meetchopra.com`

我坚定的认为支持完全的离线工作是很重要的😅，比如当你去到深山中，没有网络的时候，但是你依旧需要工作（比如我现在就在飞机上，没有网络），但是，这么多第三方的API，可能支持完全的离线调试开发吗？

很简单：我使用MSW来mock了这些API

MSW是一个可以同时模拟浏览器和node的一个网络请求模拟工具。对我来说，所有的第三方请求通过 Remix loaders 在服务端发出，所以我只需要在node server安装NSW，我喜欢NSW的原因是它完全不会耦合我的代码。运行它的方法非常的简单，这是我启动服务器的方法：

```
node .
```

这是我启动服务器加模拟请求的模式：
```
node --require ./mocks .
```

就是这样，在 `./mocks` 目录中保存了所有的NSW的 handlers 并且初始化了NSW去拦截服务器发出的HTTP请求，现在开始，我不会说说编写这些mock的代码很简单，这是相当多的代码，花了我不少时间，但是，这仍旧帮了我大忙。Mock的服务访问速度很快而且完全不依赖于网络，我强烈推荐使用它。

对于一些mock API，我使用 `fake.js` 来产生随机的模拟的符合接口类型要求的数据，但是对于 Github API，我实际上知道响应的内容，因为毫不夸张地说，我的工作就是让仓库来返回具体请求的内容。所以 Github API 是直接请求文件系统然后返回返回实际的内容，这就是我如何在本地来开发这些content。并且不需要在代码中做什么花哨的操作，就我的app而言，就是对需要的内容发出请求，接着NSW会进行拦截，并使用文件系统上的内容进行相应。

更进一步，Remix会在文件发生变化的时候时候自动加载页面，并且我已经设置了，Redis会自动刷新缓存，当content有变化的时候，这样就触发Remix去重新加载页面。我认为这整个过程相当酷。

> NSW 是一个强大的生产力工具

## Redis缓存和缓存LRU

正如之前介绍过的系统的架构图，我将Redis也部署在 Fly.io，并且我编写了Redis服务访问的抽象层，我认为这里也有一些有趣的地方来值得讨论。

首个问题：我希望我的网站访问速度非常的快，但是我也想在请求的过程做些其他的事情，这样就导致了更多的时间花费，而且有一些情况是会导致更慢和不稳定，所以我使用了Redis来缓存，这样可以使得有些请求的时间从350ms较低到5ms，但是带来的问题是何时将缓存失效，上面我说过了我是如何缓存content的，但是缓存的可不止这些地方，比如许多的第三方的查询和一些的Postgres的查询都进行了缓存（Postgres本身的性能是十分不错的，但是在我的博客上，每个页面会需要差不多30个查询）。

并非所有的内容都缓存在Redis，一些存放在 `lru-cache` 模块（lru是“least-recently-used”，帮你避免内存溢出的问题），我使用非持久化的LRU来处理那些时效性较低的数据，比如Postgres查询。

因为大量的内容需要被缓存，所以一个抽象层是很有意义的，能够来处理缓存更简单和一致，我没耐心去找一个所以自己造了个轮子。

这里是API：

```js
type CacheMetadata = {
  createdTime: number
  maxAge: number | null
}
// it's the value/null/undefined or a promise that resolves to that
type VNUP<Value> = Value | null | undefined | Promise<Value | null | undefined>

async function cachified<
  Value,
  Cache extends {
    name: string
    get: (key: string) => VNUP<{
      metadata: CacheMetadata
      value: Value
    }>
    set: (
      key: string,
      value: {
        metadata: CacheMetadata
        value: Value
      },
    ) => unknown | Promise<unknown>
    del: (key: string) => unknown | Promise<unknown>
  },
>(options: {
  key: string
  cache: Cache
  getFreshValue: () => Promise<Value>
  checkValue?: (value: Value) => boolean
  forceFresh?: boolean | string
  request?: Request
  fallbackToCache?: boolean
  timings?: Timings
  timingType?: string
  maxAge?: number
}): Promise<Value> {
  // do the stuff...
}

// here's an example of the cachified credits.yml that powers the /credits page:
async function getPeople({
  request,
  forceFresh,
}: {
  request?: Request
  forceFresh?: boolean | string
}) {
  const allPeople = await cachified({
    cache: redisCache,
    key: 'content:data:credits.yml',
    request,
    forceFresh,
    maxAge: 1000 * 60 * 60 * 24 * 30,
    getFreshValue: async () => {
      const creditsString = await downloadFile('content/data/credits.yml')
      const rawCredits = YAML.parse(creditsString)
      if (!Array.isArray(rawCredits)) {
        console.error('Credits is not an array', rawCredits)
        throw new Error('Credits is not an array.')
      }

      return rawCredits.map(mapPerson).filter(typedBoolean)
    },
    checkValue: (value: unknown) => Array.isArray(value),
  })
  return allPeople
}
```

上面一大堆😶，但是别担心，我将一一道来，让我们以 generic type 开始:

- `Value` 类型是指值需要被从缓存存取
- `Cache` 类型是一个对象，有 `name`（用于日志）属性，以及`get`，`set`和`del`三个方法
- `CacheMetadata` 是一个描述信息连同对应的值，来决定某个值什么时候需要刷新


然后现在是选项：

- `key` 是值的id
- `cache` 可以用的缓存
- `getFreshValue` 是用来获取获取最新的值的方法，当没有缓存的时候，这个方法将会被调用，当拿到了最新值后，会被存到对应 `key` 的 `cache` 
- `checkValue` 是一个function用来验证从`cache`/`getFreshValue`获取的值是正确的，有这么一种可能性，就是我发布的一个更新，`getFreshValue` 会去变更 `value`，当缓存的值不是正确的，这时候我们希望确保 `getFreshValue` 调用成功来避免运行时错误。我们也使用这个方法去检查从 `getFreshValue` 拿的值是正确的，如果失败了，则抛出一个异常来帮助排查（这样肯定比只抛出type异常更直观）。
- `forceFresh` 允许你跳过查询缓存（缓存都没有失效）而直接调用 `getFreshValue`，当你传入字符串，会以 “,” 逗号切分，并且检查是否有 `key` 存在，有的话就会去调用 `key`。这很有，当一个 cachified（使其可以缓存） 的方法去尝试访问另一个 cachified 的方法（假设这个方法可以获取博客所有的mdx文件），只会刷新部分缓存，而不是所有。
- `request` 用来决定 `forceFresh` 的默认值，如果这个请求带有 `?fresh` 的查询参数，并且用户是 ADMIN 角色（比如我），然后 `forceFresh` 会被设置为 **true**，这让我可以在任何页面来刷新所有资源的缓存，不过，我需要经常这么做。你也可以提供一个可以逗号进行分割的键值对来刷新指定的值。
- `fallbackToCache` 如果我们使用了 `forceFresh`（跳过了缓存的值），但是操作失败，然后我们不希望此时抛出错误，可以退回选择获取缓存中的值，这个就是控制上述行为，并且默认打开。
- `timings` 和 `timingsType` 用来另外一个很实用的功能，可以记录下花费的时间，并存在 `Server-Timing` 的头部信息中（用来排查性能瓶颈）。
- `maxAge` 用来控制缓存的最大生命周期，以此来自动刷新。

当值来自缓存，直接返回。当请求过来，我们判断缓存已经是失效，则我们会调用 `cachified`，并将其中的属性 `forceRefresh` 设置为 **true**。

> 这样的好处是，用户不用等待 `getFreshValue`，但是这样做的问题是，在刷新之前（也就是在数据过期后）的最后一个请求用户会拿到缓存的旧值，我认为这是一个合理的权衡。

我很满意这个抽象层，很可能我会把这部分独立为一个开源项目，并将这段放进工程的 `README.md`。😅

## Image optimization with Cloudinary （使用Cloudinary来进行图片优化）

好吧，大伙们，Cloudinary 太令人赞叹了，这里所有的图片都存储在 Cloudinary，然后能够将图片以最合适的大小和尺寸发送到你的设备中的浏览器，使用它只需要一点点工作（费用不低...Cloudinary不便宜），可以节省大量的网络带宽，而且让图片加载超快。

我之前的网站，之所以花费很长的时间去编译，其中一个原因也是 gatsby 每次都不得不去将所有的图片重新生成需要的尺寸。Gatsby 的开发团队也帮助我来把这些放在一个可以持久化的缓存，但是如果我需要bust 缓存（比如刷新缓存），那我不得不运行 Netlify 几次（有可能超时）来填入新的缓存，以便我可以重新部署网站。

在 Cloudinary，我没有这个问题了，我只需要上传图片，然后在我的 mdx 文件上使用 cloudinary ID即可，然后他们会在 `<img />` 标签上生成对应的 `sizes` 和 `srcset` 属性。因为 Cloudinary 也支持通过 URL 来传递参数，这样我可以以这种方式来获取对应尺寸的图片。

另外，我使用 Cloudinary 去动态的生成所有的社交图片。在 [The Call Kent Podcast](https://kentcdodds.com/calls)中也是用了同样的方式，太疯狂了。

我正在做的另一件很酷的事情，您可能已经在博客文章中注意到了，在服务器端，我请求的 banner 图片，宽度仅 `100px` 和包含 `blur` 属性，接着我把图片转为base64，这与文章的其他metadata一起被缓存，接着当我运行server-render这篇文章的时候，会server-render这个base64模糊的图片去等比例放大（我也会使用CSS属性 `backdrop-filter` 让它在放大的时候更加平滑），当加载完成的时候fade-in（淡入）全尺寸的图片，我对这种模式很满意。

> Cloudinary 拓展了我的思路，所以花的钱也值得了。

## MDX Compilation with mdx-bundler （编译MDX文件）

自从我不在 [since I left Medium](https://kentcdodds.com/blog/goodbye-medium)发博客后我就一直用MDX，我真的喜欢这种方式，可以让我轻松的在博客文章加入交互内容，而不用特殊的方式来处理它们。

当我从采用 Gatsby 的构建时编译MDX，到 Remix 的按需编译，我需要找到合适的方法来制定如何按需编译，在 `xdm` 创建的时候是一个合适的时候（它是一个更快的非运行时编译器），但是它只是一个编译器（compiler），并非一个bundler，如果在MDX文件中你引入了外部的组件，在你跑这些代码的时候你需要确保这些引入是正确的，所以我觉得我需要的不仅仅是个compiler，而是bundler。

但是没有找到具体的实现，所以我自己造了个，命名为 `mdx-bundler`。使用rollup来打包，后来尝试了使用esbuild，发现简直快到飞起（虽然还不够完全支持 bundle-on-demand，所以我对于已经编译完成的文件放入了缓存）。

正如人们期待那样，我有一些unified的插件（remark/rehype），可以在mdx编译的时候去自动化一些任务。其中一个可以给amazon和egghead的链接自动附加查询参数，另外一个可以将链接转换为自定义的推文以放在文章内（比直接用twitter widget更快），还有一个可以将egghead的视频链接转为视频放在文章中，我还有一个基于Shiki的自定义语法高亮插件（借鉴了Ryan Florence的一个神秘的库），以及一个优化inline的cloudinary图片插件。

Unified十分强大，我很乐意将它用在基于markdown的内容处理。

## Database interaction with Prisma（使用 Prisma 来操作数据库）

好的，朋友们，让我们来谈谈 Prisma，我并不是数据库的专家，后端服务并不是我的强项，有趣的是，Remix使得后端开发相当平易近人，最近几个月我一直都在做后端的事情😆，我十分开心使用 Prisma 和数据库交互是相当的简单，不仅仅是在 Postgres 进行数据，还包括数据合并，Prsima真的令人惊叹，接下来让我们来谈谈它。

### Migrations

在Prsima，通过 `schema.prisma` 这个文件来描述数据库模型，然后，您可以告诉 Prisma 使用它来更新数据库以反映数据结构，运行 `prisma migrate dev --name <descriptive-name>`, 接着 Prisma 会生成SQL来变更结构。

如果你小心处理，你可以进行零停机变更，当然，零停机变更不是Prisma独有的，但prisma确实使创建这些迁移对我来说变得更加简单，因为太多年没接触，而且在开发我的网站期间，我从来没有真正喜欢做SQL的开发😬。

### TypeScript

`schema.prisma` 也可以用来根据数据库生成模型的类型描述文件，这有个简单的例子：

```js
const users = await prisma.user.findMany({
  select: {
    id: true,
    email: true,
    firstName: true,
  },
})
```

```js
// This is users type. To be clear, I don't have to write this myself,
// the call above returns this type automatically:
const users: Array<{
  id: string
  email: string
  firstName: string
}>
```

如果接下来我需要获取 `team` 字段：

```js
const users = await prisma.user.findMany({
  select: {
    id: true,
    email: true,
    firstName: true,
    team: true, // <-- just add the field I want
  },
})
```

然后 `users` 这个数组就变成了这样：

```js
const users: Array<{
  id: string
  email: string
  firstName: string
  team: Team
}>
```

那么，如果我想知道这个用户读了哪些文章，该如何操作呢？我需要graphql吗？不！看下面：

```js
const users = await prismaRead.user.findMany({
  select: {
    id: true,
    email: true,
    firstName: true,
    team: true,
    postReads: {
      select: {
        postSlug: true,
      },
    },
  },
})
```

现在，`users` 数组是这样：

```js
const users: Array<{
  firstName: string
  email: string
  id: string
  team: Team
  postReads: Array<{
    postSlug: string
  }>
}>
```

现在才是我要说的，在 Remix，我能直接在 `loader` 中进行查询，然后在页面组件中轻易的拿到需要的typed的数据。

```js
type LoaderData = Await<ReturnType<typeof getLoaderData>>

async function getLoaderData() {
  const users = await prismaRead.user.findMany({
    select: {
      id: true,
      email: true,
      firstName: true,
      team: true,
      postReads: {
        select: {
          postSlug: true,
        },
      },
    },
  })
  return {users}
}

export const loader: LoaderFunction = async ({request}) => {
  return json(await getLoaderData())
}

export default function UsersPage() {
  const data = useLoaderData<LoaderData>()
  return (
    <div>
      <h1>Users</h1>
      <ul>
        {/* all this auto-completes and type checks!! */}
        {data.users.map(user => (
          <li key={user.id}>
            <div>{user.firstName}</div>
          </li>
        ))}
      </ul>
    </div>
  )
}
```

如果我需要在前端去除某些字段，只需更新prsima的查询语句，那么Typescript会确保我不会忘记了什么，这简直太棒了。

> Prisma 让我一个前端开发人员，觉得有能力直接使用数据库。

## Authentication with Magic Links （身份验证）

不久前，我发了下面的推文：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5bc90b743f64a99ae2be10784ea38b4~tplv-k3u1fbpfcp-watermark.image?)
https://twitter.com/kentcdodds/status/1299758509312024576

是的，没错，在这里，我实现了内建的用户验证。我的理由是，还记得上面关于通过在用户附近配置节点服务器来让访问变得极快的讨论，但是如果我使用了第三方的验证服务，那等于要回滚之前的所有工作，每个请求都需要去到验证服务所在的区域去验证用户登录状态，这太无法接受了。

在我处理登录这个问题的时候，Ryan Florence做了些直播 [live](https://www.youtube.com/watch?v=f1OFqldJ9FE) [streams](https://www.youtube.com/watch?v=rU_C2pg_P40) 来介绍他如何在Remix中集成鉴权服务，它看起来并不是那么复杂，他十分热心的给我整理了一个大纲，让我在短短一天内就完成了大部分工作。

使用 magic link 来进行身份验证真的帮了很大忙，这样做意味着我不用担心如何存储密码或者是处理忘记密码和改密码的服务，这并不是一个自私或懒惰的决定，我强烈地认为使用 magic link 对于这个app是最好的选择，请记住，很多应用都有一个类似 “magic-link” 的身份验证系统，像重置密码的流程中发送到你邮箱中的重置密码的链接，你看到的是隐含的超链接，事实上，这样更加安全，因为没有密码可以丢失了。

哦，你会说：

_但如果没有密码，我无法使用密码管理器，我怎么记得在我三十个邮箱地址中，用了其中哪个注册了你网站。_

你的密码管理器，绝对可以存储登录信息，比如只提供了邮件地址，而没有密码这种方式。

好的，让我们来看看身份验证的流程图

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf9e3a52ea4f42d4a8a56b86264ab80c~tplv-k3u1fbpfcp-watermark.image?)

我需要指出在这个流程图没有与数据库的交互，直到用户在实际登录后，并且登录和注册的流程是一样的，对我来说大大简化了相应的工作。

现在，让我们看一下当用户导航到需要验证的页面时会发生什么情况。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f845ec6e5c74499a9cd10e4c1dd52a8~tplv-k3u1fbpfcp-watermark.image?)

其基本原理非常简单：

- 从 session cookie 中拿到 session ID
- 从 session 中拿到 user ID
- 获取用户对象
- 更新过期时间，以便活跃用户几乎不需要重新进行身份验证
- 如果上面任何一步失败，则清理并跳转

老实说，它并不像我多年前在我工作的其他应用程序中实现身份验证时那么复杂。Remix 有助于通过其 cookie 会话抽象使其更加容易。

## Remix

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cbf00e3aed9643458a7a578b168930a6~tplv-k3u1fbpfcp-watermark.image?)

好了，朋友们，在我所用的工具中，Remix 对我的工作效率和网站性能的影响最大。Remix 使我能够完成所有这些很酷的事情，而不至于太复杂。

我肯定会在未来写很多关于Remix的博客文章，所以赶快点击[订阅](https://kentcdodds.com/subscribe)吧，下面的内容说明了 Remix 为何如何出色：

1. 服务端和用户端的通讯的便利性，数据的 over-fetching 将不在是个问题，因为我可以在服务端把不需要的数据过滤掉，只保留前端需要的内容，正因为如此，不需要一个庞大而复杂的 graphql 后端和前端端库来处理这个问题（当然你也可以在 remix 中使用 graphql）。这个功能是很强大的，未来几个月，会有关于这个内容的博客。
2. auto-performance 也是我在使用 Remix 的 web 平台的强大特性，后续也会有跟多的博客来展开。
3. 能够为特定的 router 应用 css，而且不会与其他 router 的 css 冲突，再见了 CSS-in-JS👋。
4. 事实上，我甚至不必考虑服务器上的缓存，因为Remix为我处理了这些（也包括后面的数据转换 mutations），所有组件都可以假设数据已准备就绪。使用声明的方式来管理异常/错误，Remix 没有实现自己的缓存，而是利用浏览器缓存使事情变得超快，即使在重新加载（或在新选项卡中打开链接）之后也是如此。
5. 不用担心，像其他框架那样需要有一个 “Layout” 组件，这可以让我关注于数据进行加载的角度，当然，也也要后续展开说明。

我提到了上面的一些特性需要深入展开，并不是说还需要更多深入来充分利用它，这就是 Remix 的工作模式，我花了更多的时间来确保我的app不会被框架所限制，而不仅仅完成它。

## Conclusion 结论

我无法告诉你在建立这个网站过程中我学习了多少，但是我知道十分有趣，而且我很开心把我学到的内容放在博客文章和 workshop，这样你可以看到我是如何做，实际上，我也准备了一些 workshops 让大家参加，[在这里买票](https://kentcdodds.com/workshops)，十分期待可以在那看到你。

别忘了，如果你还没看过 [ "Introducing the new kentcdodds.com"](https://kentcdodds.com/blog/introducing-the-new-kentcdodds.com)，那赶紧打开看看吧。

