# Next.js 编译器

<details open>
  <summary><b>版本历史</b></summary>

| 版本   | 变化                                                                                                                            |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `v12.2.0` | [SWC 插件](#swc-plugins-Experimental) 添加了实验性支持。                                                         |
| `v12.1.0` | 添加了对 Styled Components，Jest，Relay，Remove React Properties，Legacy Decorators，Remove Console，jsxImportSource 的支持 |
| `v12.0.0` | Next.js 编译器 [介绍](https://nextjs.org/blog/next-12).                                                                    |

</details>
Next.js 编译器使用的是 Rust 编写的 [SWC](http://swc.rs/)，允许 Next.js 转换和缩小 JavaScript 代码以用于生产。替换单个文件的 Babel 和 缩小输出包的 Terser。

使用 Next.js 编译器的编译速度比 Babel 快17倍，并且从 Next.js v12 起默认启用。
如果您有现有的 Babel 配置或正在使用 [不支持的功能](#unsupported-features)，您的应用程序将退出 Next.js 编译器并继续使用 Babel。

## 为什么选择 SWC?

[SWC](http://swc.rs/) 是一个可扩展的基于 Rust 的下一代快速开发工具。

SWC 可以用于编译、缩小、绑定等，并被设计为可扩展的。您可以调用它来执行代码转换（内置或自定义）。运行这些转换是通过像 Next.js 这样的高级工具实现的。

我们选择在 SWC 的基础上进行构建有几个原因：

- **扩展:** 可以用作 Next.js 内部的板条箱，而无需 Fork 这个库或者变通方法设计约束。
- **性能:** 通过切换到 SWC，我们能够在 Next.js 中实现约 3 倍的快速刷新速度和约 5 倍的构建速度，并且还有更多的优化空间仍在进行中。
- **WebAssembly:** Rust 对 WASM 的支持对于支持所有可能的平台和让 Next.js 开发无处不在至关重要。
- **社区:** Rust 社区和生态系统令人惊叹，并且仍在增长。

## 支持的功能

### Styled Components

我们正在将 `babel-plugin-styled-components` 移植到 Next.js 编译器。

首先，更新到 Next.js 的最新版本：`npm install next@latest`。然后，更新 `next.config.js` 文件：

```js
module.exports = {
  compiler: {
    // 有关更多选项的信息，请请查看 https://styled-components.com/docs/tooling#babel-plugin
    styledComponents: boolean | {
      // 在开发中默认启用，在生产中禁用以减小文件大小，
      // 设置此选项将覆盖所有环境的默认值。
      displayName?: boolean,
      // 默认情况下启用。
      ssr?: boolean,
      // 默认情况下启用。
      fileName?: boolean,
      // 默认情况下为空。
      topLevelImportPaths?: string[],
      // 默认为 ["index"].
      meaninglessFileNames?: string[],
      // 默认情况下启用。
      cssProp?: boolean,
      // 默认情况下为空。
      namespace?: string,
      // 尚不支持。
      minify?: boolean,
      // 尚不支持。
      transpileTemplateLiterals?: boolean,
      // 尚不支持。
      pure?: boolean,
    }，
  }，
}
```

`minify`，`transpileTemplateLiterals` 和 `pure` 尚未实现。你可以 [在此处](https://github.com/vercel/Next.js/issues/30802) 跟踪进度。 `ssr` 和 `displayName` 是在 Next.js 中使用 `styled-components` 的主要需求。

### Jest

Jest 支持不仅包括 Babel 之前提供的转换，还简化了与 Next.js 一起配置Jest，包括：

- 自动模拟 `.css`，`.module.css` (以及 `.scss`)，和图像导入
- 使用 SWC 自动设置 `transform`
- 加载 `.env` 到 `process.env`
- 忽略测试解析和转换中的 `node_modules`
- 忽略测试解析中的 `.next`
- 为启用实验 SWC 转换的标志加载 `next.config.js`

首先，更新到 Next.js 的最新版本：`npm install next@latest`。 然后，更新 `jest.config.js` 文件：

```js
// jest.config.js
const nextJest = require('next/jest')

// 提供 Next.js 应用程序的路径， 该路径将启用加载 next.config.js 和 .env 文件
const createJestConfig = nextJest({ dir: './' })

// 要传递给 Jest 的任何自定义配置
const customJestConfig = {
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
}

// createJestConfig 以这种方式导出，以确保 next/jest 可以加载 Next.js 配置，该配置是异步的
module.exports = createJestConfig(customJestConfig)
```

### Relay

启用 [Relay](https://relay.dev/) 支持:

```js
// next.config.js
module.exports = {
  compiler: {
    relay: {
      // 这应该与 relay.config.js 匹配
      src: './',
      artifactDirectory: './__generated__',
      language: 'typescript',
    },
  },
}
```

注意：在 Next.js 中， `pages` 目录中的所有 JavaScript 文件都被视为路由。 因此对于 `relay-compiler`，你需要在 `pages` 之外指定  `artifactDirectory` 配置， 否则 `relay-compiler` 将在 `__generated__` 目录中的源文件旁边生成文件，并且该文件将被视为路由，则将中断生产构建。

### 移除 JSX 属性

允许移除 JSX 属性. 这通常用于测试， 类似于 `babel-plugin-react-remove-properties`。

删除与默认正则表达式 `^data-test` 匹配的属性：

```js
// next.config.js
module.exports = {
  compiler: {
    reactRemoveProperties: true
  }
}
```

删除自定义属性：

```js
// next.config.js
module.exports = {
  compiler: {
    // 这里定义的正则表达式是在 Rust 中处理的， 因此语法与 JavaScript 的 `RegExp` 不同. 
    // 请查看 https://docs.rs/regex.
    reactRemoveProperties: { properties: ['^data-custom$'] },
  },
}
```

### 移除 Console 调用

移除所有在程序代码中调用的 `console.*`（不包括 `node_modules`）。

类似于 `babel-plugin-transform-remove-console`。

删除所有 `console.*` 调用：

```js
// next.config.js
module.exports = {
  compiler: {
    removeConsole: true
  }
}
```

删除所有 `console.*`，排除 `console.error`：

```js
// next.config.js
module.exports = {
  compiler: {
    removeConsole: {
      exclude: ['error'],
    },
  },
}
```

### 传统装饰器

Next.js 将自动检测 `jsconfig.json` / `tsconfig.json` 中的 `experimentalDecorators`。

Legacy decorators 通常与较旧版本的库一起使用 (比如: `mobx`)。

仅当与现有应用程序兼容时才支持此标志。我们不建议在新的应用程序中使用旧的装饰器。

首先，更新到 Next.js 的最新版本：`npm install next@latest`。 然后，更新`jsconfig.json` or `tsconfig.json` 文件：

```js
{
  "compilerOptions": {
    "experimentalDecorators": true
  }
}
```

### 导入源

Next.js 将自动检测 `jsconfig.json` / `tsconfig.json`  中的 `jsxImportSource`，并应用它。

这通常与 Theme UI 等库一起使用。

首先，更新到 Next.js 的最新版本： `npm install next@latest`。 然后，更新 `jsconfig.json` 或者 `tsconfig.json` 文件：

```js
{
  "compilerOptions": {
    "jsxImportSource": 'preact'
  }
}
```

### Emotion

我们正在努力将 `@emotion/babel-plugin` 移植到 Next.js 编译器。

首先，更新到 Next.js 的最新版本: `npm install next@latest`。 然后，更新`next.config.js` 文件：

```js
// next.config.js

module.exports = {
  compiler: {
    emotion: boolean | {
      // 默认值为 true。当构建类型为 production 时，它将被禁用。
      sourceMap?: boolean,
      // 默认值为 'dev-only'.
      autoLabel?: 'never' | 'dev-only' | 'always',
      // 默认值为 '[local]'.
      // 允许的值: `[local]` `[filename]` and `[dirname]`
      // 此选项仅在 autoLabel 设置为 'dev-only' 或 'always' 时有效。
      // 它允许您定义生成的标签的格式。
      // 格式通过字符串定义，其中变量部分用方括号 [] 括起来。
      // 例如，labelFormat: "my-classname--[local]"，其中[local] 将替换为结果分配给的变量的名称。
      labelFormat?: string,
    },
  },
}
```
目前只支持 `@emotion/babel-plugin` 中的 `importMap`。

## 实验功能

### 最小化

您可以选择使用 Next.js 编译器进行缩小。这比 Terser 快7倍。

```js
// next.config.js

module.exports = {
  swcMinify: true
}
```

如果你对 `swcMinify` 有反馈，请在 [这里](https://github.com/vercel/Next.js/discussions/30237) 分享。

### 缩小器调试选项

虽然缩小器（minifier）是实验性的，但我们将以下选项用于调试。一旦缩小器发布稳定版后，它们将不可用。

```js
// next.config.js

module.exports = {
  experimental: {
    swcMinifyDebugOptions: {
      compress: {
        defaults: true,
        side_effects: false
      }
    }
  },
  swcMinify: true
}
```
如果您的应用程序使用上述选项，则表示 `side_effects` 是有问题的选项， 有关详细，请查看 [SWC文档](https://swc.rs/docs/configuration/minification#jscminifycompress)。


### 模块化导入
 
允许模块化导入。类似于 [babel-plugin-transform-imports](https://www.npmjs.com/package/babel-plugin-transform-imports) 。

解构导入：

```js
import { Row,Grid as MyGrid } from 'react-bootstrap'
import { merge } from 'lodash'
```

单一导入：

```js
import Row from 'react-bootstrap/lib/Row'
import MyGrid from 'react-bootstrap/lib/Grid'
import merge from 'lodash/merge'
```

配置上述转换：

```js
// next.config.js
module.exports = {
  experimental: {
    modularizeImports: {
      'react-bootstrap': {
        transform: 'react-bootstrap/lib/{{member}}',
      },
      lodash: {
        transform: 'lodash/{{member}}',
      }
    }
  }
}
```

高级转换：

- 使用正则表达式

类似于 `babel-plugin-transform-imports`，但转换是用  [handlebars](https://docs.rs/handlebars) 模版化的， 正则表达式是 Rust [正则表达式](https://docs.rs/regex/latest/regex/) 语法。

配置：

```js
// next.config.js
module.exports = {
  experimental: {
    modularizeImports: {
      'my-library/?(((\\w*)?/?)*)': {
        transform: 'my-library/{{ matches.[1] }}/{{member}}',
      }
    }
  }
}
```

代码：

```js
import { MyModule } from 'my-library'
import { App } from 'my-library/components'
import { Header,Footer } from 'my-library/components/App'
```

成为：

```js
import MyModule from 'my-library/MyModule'
import App from 'my-library/components/App'
import Header from 'my-library/components/App/Header'
import Footer from 'my-library/components/App/Footer'
```

- Handlebars 模版

此转换使用 [handlebars](https://docs.rs/handlebars) 在 `transform` 字段中模版化替换导入路径。 这些遍历和助手函数可用：

1. `matches`: 类型为`string[]`。 正则表达式匹配的所有组。  `matches.[0]` 是完全匹配。
2. `member`: 类型为 `string`。 成员导入的名称。
3. `lowerCase`，`upperCase`，`camelCase`: 用于将字符串转换为小写、大写或驼峰大小写的辅助函数。

### SWC Trace profiling

您可以将 SWC 的内部转换， 跟踪生成为 chromium 的 [跟踪事件格式](https://docs.google.com/document/d/1CvAClvFfyA5R-PhYUmn5OOQtYMH4h6I0nSsKchNAySU/preview?mode=html#%21=)。

```js
// next.config.js

module.exports = {
  experimental: {
    swcTraceProfiling: true,
  }
}
```

一旦启用，SWC 将在 `.next/` 下生成名为 `swc-trace-profile-${timestamp}.json` 的跟踪。 Chromium 的跟踪查看器 ( chrome://tracing/，https://ui.perfetto.dev/ )，或者兼容的火焰图查看器 ( https://www.speedscope.app ) 可以加载和查看生成的跟踪。

### SWC Plugins (实验)

您可以将 swc 的转换配置为使用 wasm 中编写的 swc 实验插件支持来定制转换行为。

```js
// next.config.js

module.exports = {
  experimental: {
    swcPlugins: [
      [
        'plugin',
        {
          ...pluginOptions,
        },
      ],
    ],
  },
}
```

`swcPlugins` 接受一个用于配置插件的元组数组。插件的元组包含插件的路径和插件配置的对象。插件的路径可以是 npm 模块包名，也可以是 `.wasm` 二进制文件本身的绝对路径。

## 不支持的功能

当您的应用程序有 `.babelrc` 文件时，Next.js将自动返回使用Babel来转换单个文件。这确保了与利用自定义Babel插件的现有应用程序的向后兼容性。


如果您使用的是自定义 Babel 设置，[请共享您的配置](https://github.com/vercel/Next.js/discussions/30174)。我们正在努力移植尽可能多的常用 Babel 转换，并在将来支持插件。