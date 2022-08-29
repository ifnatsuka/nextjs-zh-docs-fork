# Next.js 使用 MDX

MDX 是 Markdown 的超集，允许您直接在 Markdown 文件中写入 JSX。这是一种添加动态交互性并在内容中嵌入组件的强大方式，可帮助您使页面栩栩如生。

Next.js 通过多种不同的方式支持 MDX，本页介绍将 MDX 集成到 Next.js 项目中的一些方法。

## 为什么使用 MDX?

Markdown 是一种直观的编写内容的方式，其简洁的语法能够编写既可读又可维护的内容。由于可以在 Markdown 中使用 `HTML` 元素，所以在设置 Markdown 页面的样式时也可以发挥创意。

但是，由于 Markdown 本质上是静态内容，因此不能基于用户交互创建动态内容。MDX 的亮点在于它能够直接在标记中创建和使用 React 组件，构建具有交互的网站页面成为可能。

## MDX 插件

MDX 内部使用 Remark 和 Rehype。

Remark 是一个由插件生态系统支持的 Markdown 处理器。这个插件生态系统允许您解析代码、转换 `HTML` 元素、更改语法、提取 frontmatter 等等。

Rehype 是一个 `HTML` 处理器，也由插件生态系统提供支持。类似于 remark，这些插件允许您操作、清理、编译和配置所有类型的数据、元素和内容。

要使用 remark 或 rehype 中的插件，您需要将其添加到 MDX packages 配置中。

## `@next/mdx`

`@next/mdx` 包在项目根目录的 `next.config.js` 文件中配置。**它从本地文件中获取数据**，允许您直接在 `/pages` 目录中创建扩展名为 `.mdx` 的页面。

### 在 Next.js 中设置 `@next/mdx`

以下步骤概述了如何在 Next.js 项目中设置 `@next/mdx`：

1. 安装所需的包：

   ```bash
     npm install @next/mdx @mdx-js/loader
   ```

2. 支持顶级 `.mdx` 页面。下面添加了 `options` 对象，允许您传入任何插件：

   ```js
   // next.config.js

   const withMDX = require('@next/mdx')({
     extension: /\.mdx?$/,
     options: {
       remarkPlugins: [],
       rehypePlugins: [],
       // 如果要使用 `MDXProvider`, 就取消下一行的注释.
       // providerImportSource: "@mdx-js/react",
     },
   })
   module.exports = withMDX({
     // 用 md 扩展名附加默认值
     pageExtensions: ['ts', 'tsx', 'js', 'jsx', 'md', 'mdx'],
   })
   ```

3. 在 `/pages` 目录创建一个新的 MDX 页面：

   ```bash
     - /pages
       - my-mdx-page.mdx
     - package.json
   ```

## 使用 Components, Layouts 和 Custom Elements

现在，您可以直接在 MDX 页面中导入 React 组件：

```md
import { MyComponent } from 'my-components'

# My MDX page

This is a list in Markdown:

- One
- Two
- Three

Checkout my React component:

<MyComponent/>
```

### Frontmatter

Frontmatter 是一种类似于YAML的 键/值 对，可用于存储关于页面的数据。默认情况下，`@next/mdx` 不支持 frontmatter，尽管有许多将 frontmatter 添加到 MDX 内容的解决方案，例如 [gray-matter](https://github.com/jonschlinkert/gray-matter)

要使用 `@next/mdx` 访问页面元数据，您可以从 `.mdx` 文件中导出元对象：

```md
export const meta = {
author: 'Rich Haines'
}

# My MDX page
```

### Layouts


要将布局添加到 MDX 页面，请创建新组件并将其导入 MDX 页面。然后，您可以使用布局组件包装 MDX 页面：

```md
import { MyComponent, MyLayoutComponent } from 'my-components'

export const meta = {
author: 'Rich Haines'
}

# My MDX Page with a Layout

This is a list in Markdown:

- One
- Two
- Three

Checkout my React component:

<MyComponent/>

export default ({ children }) => <MyLayoutComponent meta={meta}>{children}</MyLayoutComponent>
```

### Custom Elements

使用 Markdown 的一个令人愉快的方面是，它映射到本地的 `HTML` 元素，使书写快速直观：

```md
# H1 heading

## H2 heading

This is a list in Markdown:

- One
- Two
- Three
```

上面生成了以下 `HTML`：

```html
<h1>H1 heading</h1>

<h2>H2 heading</h2>

<p>This is a list in Markdown:</p>

<ul>
  <li>One</li>
  <li>Two</li>
  <li>Three</li>
</ul>
```
当您想为自己的元素设置样式以给您的网站或应用程序带来自定义感觉时，您可以传入短代码。这些是您自己的自定义组件，映射到 `HTML` 元素。为此，使用 `MDXProvider` 并将组件对象作为 prop 传递。组件对象中的每个对象键都映射到 `HTML` 元素名称。

要启用此功能，需要在 `next.config.js` 中指定 `providerImportSource: "@mdx-js/react"`。

```js
// next.config.js

const withMDX = require('@next/mdx')({
  // ...
  options: {
    providerImportSource: '@mdx-js/react',
  },
})
```

然后在页面中设置 provider

```jsx
// pages/index.js

import { MDXProvider } from '@mdx-js/react'
import Image from 'next/image'
import { Heading, InlineCode, Pre, Table, Text } from 'my-components'

const ResponsiveImage = (props) => (
  <Image alt={props.alt} layout="responsive" {...props} />
)

const components = {
  img: ResponsiveImage,
  h1: Heading.H1,
  h2: Heading.H2,
  p: Text,
  pre: Pre,
  code: InlineCode,
}

export default function Post(props) {
  return (
    <MDXProvider components={components}>
      <main {...props} />
    </MDXProvider>
  )
}
```
如果您在整个站点中使用它，您可能希望将 provider 添加到 `_app.js`，以便所有 MDX 页面都选择自定义元素配置。

## 有用的链接

- [MDX](https://mdxjs.com)
- [`@next/mdx`](https://www.npmjs.com/package/@next/mdx)
- [remark](https://github.com/remarkjs/remark)
- [rehype](https://github.com/rehypejs/rehype)
