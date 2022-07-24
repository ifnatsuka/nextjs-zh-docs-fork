# 重写

<details open>
  <summary><b>示例</b></summary>
  <ul>
    <li><a href="https://github.com/vercel/next.js/tree/canary/examples/rewrites">Rewrites</a></li>
  </ul>
</details>

<details>
  <summary><b>Version History</b></summary>

| 版本        | 更改       |
|-----------|----------|
| `v10.2.0` | 添加 `has` |
| `v9.5.0`  | 添加覆写配置   |

</details>

重写允许您将传入请求路径映射到不同的目标路径。

重写充当 URL 代理并掩盖目标路径，使用户看起来没有更改他们在网站上的位置。相反，[重写](/docs/api-reference/next-config-js/redirects) 将重新路由到新页面并显示 URL 更改。

要使用重写，您可以在 `next.config.js` 中配置 `rewrites`：

```js
module.exports = {
  async rewrites() {
    return [
      {
        source: '/about',
        destination: '/',
      },
    ]
  },
}
```

重写应用于客户端路由，上面的示例中 `<Link href="about">` 将应用重写。

`rewrites` 是一个异步函数，它返回一个数组，其中包含 `source` 和 `destination` 属性的对象：

- `source`：`String` - 是传入的请求的路径
- `destination`：`String` 是你想要路由到的路径
- `basePath`：`false` 或 `undefined` - 如果是 false，匹配时将不包括 basePath，只能用于外部重写
- `locale`：`false` 或 `undefined` - 匹配时是否包括 locale
- `has` 是一个 [has 对象](#header-cookie-and-query-matching) 的数组，具有 `type`、`key` 和 `value`属性

默认情况下，重写在检查文件系统（页面和 `/public` 文件）之后及动态路由之前被应用。从 Next.js `v10.1` 开始，可以通过从 `rewrites` 函数返回一个对象而不是数组来更改此行为：

```js
module.exports = {
  async rewrites() {
    return {
      beforeFiles: [
        // These rewrites are checked after headers/redirects
        // and before all files including _next/public files which
        // allows overriding page files
        {
          source: '/some-page',
          destination: '/somewhere-else',
          has: [{ type: 'query', key: 'overrideMe' }],
        },
      ],
      afterFiles: [
        // These rewrites are checked after pages/public files
        // are checked but before dynamic routes
        {
          source: '/non-existent',
          destination: '/somewhere-else',
        },
      ],
      fallback: [
        // These rewrites are checked after both pages/public files
        // and dynamic routes are checked
        {
          source: '/:path*',
          destination: `https://my-old-site.com/:path*`,
        },
      ],
    }
  },
}
```

注意：`beforeFiles` 中的重写并不是在匹配到一个源后立即检查文件系统/动态路由，而是继续匹配，直到所有 `beforeFiles` 都被检查过。

Next.js 路由的检查顺序是：

1. [标头]（/docs/api-reference/next-config-js/headers）被检查/应用
2. [重定向](/docs/api-reference/next-config-js/redirects) 被检查/应用
3. `beforeFiles` 重写被检查/应用
4. 来自 [公共目录](/docs/basic-features/static-file-serving) 的静态文件、`_next/static` 文件和非动态页面被检查/提供服务
5. 检查/应用 `beforeFiles` 重写，如果这些重写之一被匹配，我们在每次匹配后检查动态路由/静态文件。
6. `fallback` 重写被检查/应用，这些重写在渲染 404 页面之前和动态路由/所有静态资产被检查之后应用。如果你在 `getStaticPaths` 中使用 [fallback: true/'blocking'](/docs/api-reference/data-fetching/get-static-paths#fallback-true)，你的 `next.config.js` 中定义的 fallback `rewrites` 将不被运行

## 重写参数

在重写中使用参数时，如果在 `destination` 中未使用任何参数，则默认情况下将在查询中传递参数。

```js
module.exports = {
  async rewrites() {
    return [
      {
        source: '/old-about/:path*',
        destination: '/about', // The :path parameter isn't used here so will be automatically passed in the query
      },
    ]
  },
}
```

如果在 `destination` 中使用参数，则不会在查询中自动传递任何参数。

```js
module.exports = {
  async rewrites() {
    return [
      {
        source: '/docs/:path*',
        destination: '/:path*', // The :path parameter is used here so will not be automatically passed in the query
      },
    ]
  },
}
```

如果某个参数已在 `destination` 中使用，您仍然可以在查询中手动传递参数，方法是在 `destination` 中指定查询。

```js
module.exports = {
  async rewrites() {
    return [
      {
        source: '/:first/:second',
        destination: '/:first?second=:second',
        // Since the :first parameter is used in the destination the :second parameter
        // will not automatically be added in the query although we can manually add it
        // as shown above
      },
    ]
  },
}
```

注意：对于来自[自动静态优化](/docs/advanced-features/automatic-static-optimization)或[预渲染](/docs/basic-features/data-fetching/get-static-props)的静态页面，重写的参数将在水合后在客户端解析并在查询（query）中提供。

## 路径匹配

允许路径匹配，例如 `blog/:slug` 将匹配 `blog/hello-world`（无嵌套路径）：

```js
module.exports = {
  async rewrites() {
    return [
      {
        source: '/blog/:slug',
        destination: '/news/:slug', // Matched parameters can be used in the destination
      },
    ]
  },
}
```

### 通配符路径匹配

要匹配通配符路径，您可以在参数后使用 `*`，例如 `blog/:slug*` 将匹配 `blog/a/b/c/d/hello-world`：

```js
module.exports = {
  async rewrites() {
    return [
      {
        source: '/blog/:slug*',
        destination: '/news/:slug*', // Matched parameters can be used in the destination
      },
    ]
  },
}
```

### 正则表达式路径匹配

要匹配正则表达式路径，您可以将正则表达式包在参数后面的括号中，例如 `blog\:slug(\\d{1,})` 将匹配  `blog123` 但不匹配 `blogabc`：

```js
module.exports = {
  async rewrites() {
    return [
      {
        source: '/old-blog/:post(\\d{1,})',
        destination: '/blog/:post', // Matched parameters can be used in the destination
      },
    ]
  },
}
```

以下字符 `(`、`)`、`{`, `}`、`:`、`*`、`+`、`?` 用于路径匹配，所以当在 `source` 中作为非特殊值使用时，必须在它们前面加上 `\` 来转义。

```js
module.exports = {
  async rewrites() {
    return [
      {
        // this will match `/english(default)/something` being requested
        source: '/english\\(default\\)/:slug',
        destination: '/en-us/:slug',
      },
    ]
  },
}
```

## Header, Cookie, and Query Matching

To only match a rewrite when header, cookie, or query values also match the `has` field can be used. Both the `source` and all `has` items must match for the rewrite to be applied.

`has` items have the following fields:

- `type`: `String` - must be either `header`, `cookie`, `host`, or `query`.
- `key`: `String` - the key from the selected type to match against.
- `value`: `String` or `undefined` - the value to check for, if undefined any value will match. A regex like string can be used to capture a specific part of the value, e.g. if the value `first-(?<paramName>.*)` is used for `first-second` then `second` will be usable in the destination with `:paramName`.

```js
module.exports = {
  async rewrites() {
    return [
      // if the header `x-rewrite-me` is present,
      // this rewrite will be applied
      {
        source: '/:path*',
        has: [
          {
            type: 'header',
            key: 'x-rewrite-me',
          },
        ],
        destination: '/another-page',
      },
      // if the source, query, and cookie are matched,
      // this rewrite will be applied
      {
        source: '/specific/:path*',
        has: [
          {
            type: 'query',
            key: 'page',
            // the page value will not be available in the
            // destination since value is provided and doesn't
            // use a named capture group e.g. (?<page>home)
            value: 'home',
          },
          {
            type: 'cookie',
            key: 'authorized',
            value: 'true',
          },
        ],
        destination: '/:path*/home',
      },
      // if the header `x-authorized` is present and
      // contains a matching value, this rewrite will be applied
      {
        source: '/:path*',
        has: [
          {
            type: 'header',
            key: 'x-authorized',
            value: '(?<authorized>yes|true)',
          },
        ],
        destination: '/home?authorized=:authorized',
      },
      // if the host is `example.com`,
      // this rewrite will be applied
      {
        source: '/:path*',
        has: [
          {
            type: 'host',
            value: 'example.com',
          },
        ],
        destination: '/another-page',
      },
    ]
  },
}
```

## Rewriting to an external URL

<details>
  <summary><b>Examples</b></summary>
  <ul>
    <li><a href="https://github.com/vercel/next.js/tree/canary/examples/custom-routes-proxying">Incremental adoption of Next.js</a></li>
    <li><a href="https://github.com/vercel/next.js/tree/canary/examples/with-zones">Using Multiple Zones</a></li>
  </ul>
</details>

Rewrites allow you to rewrite to an external url. This is especially useful for incrementally adopting Next.js. The following is an example rewrite for redirecting the `/blog` route of your main app to an external site.

```js
module.exports = {
  async rewrites() {
    return [
      {
        source: '/blog',
        destination: 'https://example.com/blog',
      },
      {
        source: '/blog/:slug',
        destination: 'https://example.com/blog/:slug', // Matched parameters can be used in the destination
      },
    ]
  },
}
```

If you're using `trailingSlash: true`, you also need to insert a trailing slash in the `source` parameter. If the destination server is also expecting a trailing slash it should be included in the `destination` parameter as well.

```js
module.exports = {
  trailingSlash: true,
  async rewrites() {
    return [
      {
        source: '/blog/',
        destination: 'https://example.com/blog/',
      },
      {
        source: '/blog/:path*/',
        destination: 'https://example.com/blog/:path*/',
      },
    ]
  },
}
```

### Incremental adoption of Next.js

You can also have Next.js fall back to proxying to an existing website after checking all Next.js routes.

This way you don't have to change the rewrites configuration when migrating more pages to Next.js

```js
module.exports = {
  async rewrites() {
    return {
      fallback: [
        {
          source: '/:path*',
          destination: `https://custom-routes-proxying-endpoint.vercel.app/:path*`,
        },
      ],
    }
  },
}
```

See additional information on incremental adoption [in the docs here](/docs/migrating/incremental-adoption).

### Rewrites with basePath support

When leveraging [`basePath` support](/docs/api-reference/next-config-js/basepath) with rewrites each `source` and `destination` is automatically prefixed with the `basePath` unless you add `basePath: false` to the rewrite:

```js
module.exports = {
  basePath: '/docs',

  async rewrites() {
    return [
      {
        source: '/with-basePath', // automatically becomes /docs/with-basePath
        destination: '/another', // automatically becomes /docs/another
      },
      {
        // does not add /docs to /without-basePath since basePath: false is set
        // Note: this can not be used for internal rewrites e.g. `destination: '/another'`
        source: '/without-basePath',
        destination: 'https://example.com',
        basePath: false,
      },
    ]
  },
}
```

### Rewrites with i18n support

When leveraging [`i18n` support](/docs/advanced-features/i18n-routing) with rewrites each `source` and `destination` is automatically prefixed to handle the configured `locales` unless you add `locale: false` to the rewrite. If `locale: false` is used you must prefix the `source` and `destination` with a locale for it to be matched correctly.

```js
module.exports = {
  i18n: {
    locales: ['en', 'fr', 'de'],
    defaultLocale: 'en',
  },

  async rewrites() {
    return [
      {
        source: '/with-locale', // automatically handles all locales
        destination: '/another', // automatically passes the locale on
      },
      {
        // does not handle locales automatically since locale: false is set
        source: '/nl/with-locale-manual',
        destination: '/nl/another',
        locale: false,
      },
      {
        // this matches '/' since `en` is the defaultLocale
        source: '/en',
        destination: '/en/another',
        locale: false,
      },
      {
        // it's possible to match all locales even when locale: false is set
        source: '/:locale/api-alias/:path*',
        destination: '/api/:path*',
        locale: false,
      },
      {
        // this gets converted to /(en|fr|de)/(.*) so will not match the top-level
        // `/` or `/fr` routes like /:path* would
        source: '/(.*)',
        destination: '/another',
      },
    ]
  },
}
```
