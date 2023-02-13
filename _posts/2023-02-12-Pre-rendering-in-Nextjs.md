---

layout: post
title: 'Next.js中的预渲染'
date: 2023-02-12
author: 爱学习的lbl
color: rgb(255,146,146)
cover: 'https://www.nextjs.cn/static/images/nextjs-big-logo.png'
subtitle: 'Pre-rendering of Next.js'
tags: 前端 Nextjs
music-id: 5189652

---

在默认情况下，Next.js 会预渲染每个页面。这意味着 Next.js 会提前为每个页面生成 HTML，而不是全部由客户端 JavaScript 完成。预渲染可以带来更好的性能和SEO（Search engine optimization，搜索引擎优化）。

每个生成的 HTML 都与该页面所需的最少 JavaScript 代码相关。当浏览器加载页面时，其 JavaScript 代码将运行并使页面完全交互。（这个过程称为**hydration**，**水合作用**。）

Next.js的预渲染分为两种形式：[Static Generation](https://nextjs.org/docs/basic-features/pages#static-generation-recommended)和[Server-side Rendering](https://nextjs.org/docs/basic-features/pages#server-side-rendering)。其中，
Static Generation 又分为 [with data](https://nextjs.org/docs/basic-features/pages#static-generation-with-data), 和 [without data](https://nextjs.org/docs/basic-features/pages#static-generation-without-data) 两种情况。接下来我会分别讲讲每一种情况。

## Static Generation和 Server-side Rendering

Next.js 有两种pre-rendering形式，Static Generation和 Server-side Rendering，以下是官方的解释：
-   [**Static Generation**](https://nextjs.org/docs/basic-features/pages#static-generation-recommended): generates the HTML at **build time**;
-   [**Server-side Rendering**](https://nextjs.org/docs/basic-features/pages#server-side-rendering): generates the HTML on **each request**;

>*开发模式*下（运行`npm run dev`或`yarn dev`时），页面在每次request的时候预渲染。 这也适用于静态生成，使其更易于开发。
>*投入生产*时，静态生成将在构建时发生一次，而**不是**在每次请求时发生。
> 
> 我们可以分别为每个page设置pre-rendering形式

>你应该问问自己：“我可以在用户请求之前**预渲染**此页面吗？” 如果答案是肯定的，那么你应该选择[Static Generation](https://nextjs.org/docs/basic-features/pages#static-generation-recommended)。

## Static Generation
[Static Generation](https://nextjs.org/docs/basic-features/pages#static-generation-recommended) can be done with and without data.
静态生成有以下两种`pre-rendering`方式：
- **build时**请求外部data；
- **build时不需要**请求外部data，但在用户端需要请求data，比如用户dashboard，其渲染的data是根据用户高度差异化的，并且有时效性要求，无法在build时准备好所有data，所以在build时不请求data，在客户端请求对应的data；

### Static Generation with data

对于某些页面，如果不首先获取一些外部数据，可能无法呈现 HTML。也许需要在构建时访问文件系统、获取外部 API 或查询数据库。Next.js开箱即用地支持这种情况——[**带数据的静态生成**](https://nextjs.org/docs/basic-features/pages#static-generation-with-data)

![Image about Static Generation with Data From Next.js Document](https://nextjs.org/static/images/learn/data-fetching/static-generation-with-data.png)

#### 方法： `getStaticProps`

在 Next.js 中，导出页面组件时，你可以导出一个名为`getStaticProps` 的`async` function（异步函数）。
-   [`getStaticProps`](https://nextjs.org/docs/basic-features/data-fetching#getstaticprops-static-generation)在生产中的 _构建_ 时运行
-   在函数内部，可以获取外部数据并将其作为props发送到page。

例如：
```jsx
export default function Home(props) { ... }

export async function getStaticProps() {
  // Get external data from the file system, API, DB, etc.
  const data = ...

  // The value of the `props` key will be
  //  passed to the `Home` component
  return {
    props: ...
  }
}
```

这是一段官网解释：
>Essentially, `getStaticProps` allows you to tell Next.js: _“Hey, this page has some data dependencies — so when you pre-render this page at build time, make sure to resolve them first!”_
>
>**Note**: In **development mode**, `getStaticProps`  runs on each request instead.

#### 从API拿数据
```js
export async function getSortedPostsData() {
  // Instead of the file system,
  // fetch post data from an external API endpoint
  const res = await fetch('..');
  return res.json();
}
```
#### 从数据库拿数据
```js
import someDatabaseSDK from 'someDatabaseSDK'

const databaseClient = someDatabaseSDK.createClient(...)

export async function getSortedPostsData() {
  // Instead of the file system,
  // fetch post data from a database
  return databaseClient.query('SELECT posts...')
}
```
> `getStaticProps`只**在服务器端运行**。它永远不会在客户端运行。它甚至不会包含在浏览器的 JS 包中。这意味着你可以编写直接数据库查询等代码，而无需将它们发送到浏览器。

> 在**开发**（`npm run dev`或`yarn dev`）中，[`getStaticProps`](https://nextjs.org/docs/basic-features/data-fetching#getstaticprops-static-generation)在 _每个请求（request）_ 上运行。
> 在**生产**中，`getStaticProps`在 _构建时_ 运行。但是，可以使用`getStaticPaths`返回的[`fallback` key ](https://nextjs.org/docs/api-reference/data-fetching/get-static-paths#fallback-false)来增强此行为

> `getStaticProps`只能从[**page**](https://nextjs.org/docs/basic-features/pages)导出。不能从non-page文件中导出它。

### Static Generation with data 总结
由于[静态生成](https://nextjs.org/docs/basic-features/pages#static-generation-recommended)在 _build_ 时发生一次，因此它不适合频繁更新或根据每个用户请求更改的数据。

如果你的数据可能会发生变化，你可以使用**服务器端渲染**。

## Server-side Rendering With Data

![](https://nextjs.org/static/images/learn/data-fetching/server-side-rendering-with-data.png)

为了使用 [Server-side Rendering](https://nextjs.org/docs/basic-features/pages#server-side-rendering) 你需要从页面导出 [`getServerSideProps`](https://nextjs.org/docs/basic-features/data-fetching#getserversideprops-server-side-rendering) 方法 而不是之前 提到的 [`getStaticProps`](https://nextjs.org/docs/basic-features/data-fetching#getstaticprops-static-generation) 方法。

仅当你需要预渲染的数据**必须**在请求时获取，才应使用`getServerSideProps`。
这么做，用户接收到页面第一个字节所花费的时间time to first byte ([TTFB](https://web.dev/time-to-first-byte/)) 会比 [`getStaticProps`](https://nextjs.org/docs/basic-features/data-fetching#getstaticprops-static-generation) 长，因为服务器要计算每一个request的结果， 并且如果没有额外配置， [CDN](https://vercel.com/docs/edge-network/overview)无法缓存结果。
```jsx
export async function getServerSideProps(context) {
  return {
    props: {
      // props for your component
    },
  };
}
```
`getServerSideProps`在请求时被调用，它的参数 ( `context`) 包含请求特定的参数。

如果不需要预渲染data， _指仅预渲染page，可能由于data时效性要求很高，不需要预渲染data_，也可以使用[**Client-side Rendering**](https://nextjs.org/docs/basic-features/data-fetching#fetching-data-on-the-client-side) ，比如，在dashboard用户仪表板场景中，dashboard是私有的、特定于用户的页面，所以 SEO不相关，且不需要在服务器端预渲染（不知道用户信息，预渲染无意义）；数据更新频繁，需要请求时取数据。

## Client-side Rendering

-   静态生成（预渲染）页面时，不需要外部数据。
-   当页面加载时，才使用 JavaScript 从客户端获取外部数据并填充其余部

### 适用场景
与 SEO 无关的私人的、用户特定的页面

### 适应场景举例
  比如，在dashboard用户仪表板场景中，dashboard是私有的、特定于用户的页面，所以 SEO不相关，且不需要在服务器端预渲染（不知道用户信息，预渲染无意义）；数据更新频繁，需要请求时取数据。

![](https://nextjs.org/static/images/learn/data-fetching/client-side-rendering.png)

### SWR
如果在客户端获取数据，可以使用Next.js团队出的SWR，这是一个React Hook，它处理缓存、重新验证、焦点跟踪、间隔重新获取等。

示例：
```jsx
import useSWR from 'swr';

function Profile() {
  const { data, error } = useSWR('/api/user', fetch);

  if (error) return <div>failed to load</div>;
  if (!data) return <div>loading...</div>;
  return <div>hello {data.name}!</div>;
}
```
查看[SWR 文档](https://swr.vercel.app/)以了解更多信息。
