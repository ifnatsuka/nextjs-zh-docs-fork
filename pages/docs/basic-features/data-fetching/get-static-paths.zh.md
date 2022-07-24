# getStaticPaths

如果一个页面具有[动态路由](/docs/routing/dynamic-routes)并使用了 `getStaticProps`，这时你需要定义一个**路径的列表**来进行静态生成。

当你从一个使用动态路由的页面导出一个名为 `getStaticPaths`（静态站点生成）的函数时，Next.js 将静态地预渲染 `getStaticPaths` 指定的所有路径。

```jsx
// pages/posts/[id].js
// Generates `/posts/1` and `/posts/2`
export async function getStaticPaths() {
  return {
    paths: [{ params: { id: '1' } }, { params: { id: '2' } }],
    fallback: false, // can also be true or 'blocking'
  }
}
// `getStaticPaths` requires using `getStaticProps`
export async function getStaticProps(context) {
  return {
    // Passed to the page component as props
    props: { post: {} },
  }
}
export default function Post({ post }) {
  // Render post...
}
```

[`getStaticPaths` API 参考](/docs/api-reference/data-fetching/get-static-paths)包含了所有 `getStaticPaths` 可用的参数和属性。

## 何时使用 getStaticPaths ？

如果你需要静态地预渲染使用动态路由的页面，你应该使用 `getStaticPaths`：

- 数据来自于**无头 CMS（Headless CMS）**
- 数据来自于一个**数据库**
- 数据来自于**文件系统**
- 数据可以**被公开缓存**（不针对用户）
- 页面**必须**预渲染（为了SEO），并且需要非常快 - `getStaticProps` 生成 `HTML` 和 `JSON` 文件，这两个文件可以被 CDN 缓存以提高性能

## getStaticPaths 运行的时间

`getStaticPaths` 只在生产中的构建过程运行，在运行时不会被调用。你可以[用这个工具](https://next-code-elimination.vercel.app/)验证写在 `getStaticPaths` 内的代码是否从客户端捆绑中移除。

### getStaticProps 与 getStaticPaths 的关系

- `getStaticProps` 在 `next build` 时运行，用于构建过程返回的所有 `paths`
- 当定义 `fallback: true` 时，`getStaticProps` 在后台运行
- 当定义 `fallback: blocking` 时，`getStaticProps` 在初始渲染前被调用

##  getStaticPaths 可用的位置

- `getStaticPaths` **必须**与 `getStaticProps` 一起使用
- 你**不能**将 `getStaticPaths` 与 [`getServerSideProps`](/docs/basic-features/data-fetching/get-server-side-props) 一起使用
- 你也可以从使用 `getStaticProps` 的 [动态路由](docsroutingdynamic-routes) 导出 `getStaticPaths`
- 你**不能**将 `getStaticPaths` 从一个非页面文件导出（如 `components` 文件夹）
- 您必须将 `getStaticPaths` 导出为独立函数，而不是页面组件的属性

## 开发环境响应

在开发环境中（`next dev`），`getStaticPaths` 将在**每次请求**时运行。

## 按需生成路径

`getStaticProps` 允许您控制在构建期间生成哪些页面，而不是使用 [`fallback`](/docs/api-reference/data-fetching/get-static-paths#fallback-blocking) 按需生成。在构建期间生成更多页面将导致构建速度变慢。

你可以通过为 `paths` 返回一个空数组来推迟按需生成所有页面。这在将 Next.js 应用程序部署到多个环境时特别有用。例如，你可以通过按需生成所有页面以供预览（但不是生产构建）来加快构建速度。这对于拥有数十万个静态页面的网站很有帮助。

```jsx
// pages/posts/[id].js
export async function getStaticPaths() {
  // When this is true (in preview environments) don't
  // prerender any static pages
  // (faster builds, but slower initial page load)
  if (process.env.SKIP_BUILD_STATIC_GENERATION) {
    return {
      paths: [],
      fallback: 'blocking',
    }
  }
  // Call an external API endpoint to get posts
  const res = await fetch('https://.../posts')
  const posts = await res.json()
  // Get the paths we want to prerender based on posts
  // In production environments, prerender all pages
  // (slower builds, but faster initial page load)
  const paths = posts.map((post) => ({
    params: { id: post.id },
  }))
  // { fallback: false } means other routes should 404
  return { paths, fallback: false }
}
```

## 相关

关于下一步该做什么的更多信息，我们建议阅读以下章节：

- [**getStaticPaths API 参考**](/docs/api-reference/data-fetching/get-static-paths)
