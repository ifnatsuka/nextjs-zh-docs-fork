# 自定义 Webpack 配置

> 注意：semver（语义化版本号规范） 不包括对 webpack 配置的更改，因此请自行承担风险

在继续添加自定义 webpack 配置之前，请确保 Next.js 尚未支持您的配置：

- [CSS imports](/docs/basic-features/built-in-css-support#adding-a-global-stylesheet)
- [CSS modules](/docs/basic-features/built-in-css-support#adding-component-level-css)
- [Sass/SCSS imports](/docs/basic-features/built-in-css-support#sass-support)
- [Sass/SCSS modules](/docs/basic-features/built-in-css-support#sass-support)
- [preact](https://github.com/vercel/next.js/tree/canary/examples/using-preact)
- [自定义 babel 配置](/docs/advanced-features/customizing-babel-config)

一些常见的功能可以作为插件提供：

- [@next/mdx](https://github.com/vercel/next.js/tree/canary/packages/next-mdx)
- [@next/bundle-analyzer](https://github.com/vercel/next.js/tree/canary/packages/next-bundle-analyzer)

为了扩展我们对 `webpack` 的使用，您可以定义一个函数，在 `next.config.js` 内部扩展其配置，如下所示：

```js
module.exports = {
  webpack: (
    config,
    { buildId, dev, isServer, defaultLoaders, nextRuntime, webpack }
  ) => {
    // 重要提示：返回修改后的配置
    return config
  },
}
```

> `webpack` 函数执行两次，一次用于服务器，另一次用于客户端。这允许您使用 `isServer` 属性区分客户端和服务器配置。

`webpack` 函数的第二个参数是具有以下属性的对象：

- `buildId`：`String` - 生成 ID，用作生成之间的唯一标识符
- `dev`：`Boolean` - 指示是否将在 development 中完成编译
- `isServer`：`Boolean` - 服务器端编译为 `true`，客户端编译为 `false`
- `nextRuntime`：`String | undefined` - 服务器端编译的目标运行时；无论是 `"edge"` 还是 `"nodejs"`，在客户端编译的都是 `undefined`。
- `defaultLoaders`：`Object` - Next.js 内部使用的默认加载程序
  - `babel`：`Object` - 默认的 `babel-loader` 配置

`defaultLoaders.babel` 的用法示例:

```js
// 用于添加依赖于 babel-loader 的 loader 的示例配置 
// 此代码取自 @next/mdx 插件源代码:
// https://github.com/vercel/next.js/tree/canary/packages/next-mdx
module.exports = {
  webpack: (config, options) => {
    config.module.rules.push({
      test: /\x/,
      use: [
        options.defaultLoaders.babel,
        {
          loader: '@mdx-js/loader',
          options: pluginOptions.options,
        },
      ],
    })

    return config
  },
}
```

#### `nextRuntime`

请注意，当 `nextRuntime` 是 `"edge"` 或 `"nodejs"` 时，`isServer` 是 `true`，nextRuntime "`edge`" 当前仅用于 edge runtime 的中间件和服务器组件。

## 相关
- [**next.config.js 简介** / 了解 Next.js 使用的配置文件的更多信息](/docs/api-reference/next-config-js/introduction)
