# 绝对导入和模块路径别名

<details>
  <summary><b>示例</b></summary>
  <ul>
    <li><a href="https://github.com/vercel/next.js/tree/canary/examples/with-absolute-imports">绝对导入</a></li>
  </ul>
</details>

从 [Next.js 9.4](https://nextjs.org/blog/next-9-4) 起自动支持 `tsconfig.json` 和 `jsconfig.json`、`"paths"` 和 `"baseUrl"` 选项。

> 注意：`jsconfig.json` 可以在不使用 TypeScript 时使用。

> 注意： 如果修改了 `tsconfig.json` / `jsconfig.json`, 你需要重启开发服务器，才可以生效。

这些选项允许您配置模块别名，例如，一种常见的模式是将某些目录别名为绝对路径。

这些选项的一个有用特性是它们自动集成到某些编辑器中，例如 vscode。

`baseUrl` 配置选项允许您直接从项目根目录导入。

此配置的示例：

```json
// tsconfig.json or jsconfig.json
{
  "compilerOptions": {
    "baseUrl": "."
  }
}
```

```jsx
// components/button.js
export default function Button() {
  return <button>Click me</button>
}
```

```jsx
// pages/index.js
import Button from 'components/button'

export default function HomePage() {
  return (
    <>
      <h1>Hello World</h1>
      <Button />
    </>
  )
}
```
使用 `"paths"` 可以配置模块别名。例如 `@/components/*` 到 `components/*`。

此配置的示例：

```json
// tsconfig.json or jsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/components/*": ["components/*"]
    }
  }
}
```

```jsx
// components/button.js
export default function Button() {
  return <button>Click me</button>
}
```

```jsx
// pages/index.js
import Button from '@/components/button'

export default function HomePage() {
  return (
    <>
      <h1>Hello World</h1>
      <Button />
    </>
  )
}
```
