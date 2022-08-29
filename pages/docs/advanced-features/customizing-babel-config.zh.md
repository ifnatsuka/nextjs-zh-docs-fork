# 自定义 Babel 配置

<details>
  <summary><b>示例</b></summary>
  <ul>
    <li><a href="https://github.com/vercel/next.js/tree/canary/examples/with-custom-babel-config">自定义 babel 配置</a></li>
  </ul>
</details>

Next.js 包含应用程序的 `next/babel` 预设，包含编译 React 应用程序和服务器端代码所需的一切。但是如果您想扩展默认的 Babel 配置，也是可以的。

您只需要在应用程序根目录创建一个 `.babelrc` 文件（或 `babel.config.js`），并添加 `next/babel` 预设。

`.babelrc` 示例：

```json
{
  "presets": ["next/babel"],
  "plugins": []
}
```
您可以 [查看此文件](https://github.com/vercel/next.js/blob/canary/packages/next/build/babel/preset.ts) 了解 `next/babel` 包含的预设。

添加 预设/插件 而不对其进行配置，可以采用以下方式：

```json
{
  "presets": ["next/babel"],
  "plugins": ["@babel/plugin-proposal-do-expressions"]
}
```
要添加具有自定义配置的 预设/插件，请在 `next/babel` 预设上配置，如下所示：

```json
{
  "presets": [
    [
      "next/babel",
      {
        "preset-env": {},
        "transform-runtime": {},
        "styled-jsx": {},
        "class-properties": {}
      }
    ]
  ],
  "plugins": []
}
```

要了解有关每个配置的可用选项的更多信息，请访问其文档站点。

> Next.js 使用当前的 Node.js 版本进行服务器端编译。

> `"preset-env"` 上的 `modules` 选项应保持为 `false`，否则 webpack 代码拆分将关闭。