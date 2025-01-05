---
summary: 使用 @adonisjs/static 包从给定目录提供静态文件服务。
---

# 静态文件服务器

你可以使用 `@adonisjs/static` 包从给定目录提供静态文件服务。该包附带了一个中间件，你必须在 [服务器中间件堆栈](./middleware.md#server-middleware-stack) 中注册它，以拦截 HTTP 请求并提供文件服务。

## 安装

该包在 `web` 初学者工具包中已预配置。但是，你可以按照以下步骤在其他初学者工具包中安装和配置它。

使用以下命令安装和配置该包：

```sh
node ace add @adonisjs/static
```

:::disclosure{title="查看 add 命令执行的步骤"}

1. 使用检测到的包管理器安装 `@adonisjs/static` 包。

2. 在 `adonisrc.ts` 文件中注册以下服务提供者。

    ```ts
    {
      providers: [
        // ...其他提供者
        () => import('@adonisjs/static/static_provider')
      ]
    }
    ```

3. 创建 `config/static.ts` 文件。

4. 在 `start/kernel.ts` 文件中注册以下中间件。

    ```ts
    server.use([
      () => import('@adonisjs/static/static_middleware')
    ])
    ```

:::

## 配置

静态中间件的配置存储在 `config/static.ts` 文件中。

```ts
import { defineConfig } from '@adonisjs/static'

const staticServerConfig = defineConfig({
  enabled: true,
  etag: true,
  lastModified: true,
  dotFiles: 'ignore',
})

export default staticServerConfig
```

<dl>

<dt>

  enabled

</dt>

<dd>

临时启用或禁用中间件，而不从中间件堆栈中移除它。

</dd>

<dt>

  acceptRanges

</dt>

<dd>

`Accept-Range` 头允许浏览器恢复中断的文件下载，而不是尝试重新开始下载。通过将 `acceptsRanges` 设置为 `false`，可以禁用可恢复下载。

默认为 `true`。

</dd>

<dt>

  cacheControl

</dt>

<dd>

启用或禁用 [Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) 头。当 `cacheControl` 被禁用时，将忽略 `immutable` 和 `maxAge` 属性。


```ts
{
  cacheControl: true
}
```
</dd>


<dt>

  dotFiles

</dt>

<dd>

定义如何处理 `public` 目录中对点文件的请求。你可以设置以下选项之一。

- `allow`：像其他文件一样提供点文件服务。
- `deny`：使用 `403` 状态码拒绝请求。
- `ignore`：假装文件不存在，并返回 `404` 状态码。

```ts
{
  dotFiles: 'ignore'
}
```

</dd>


<dt>

  etag

</dt>

<dd>

启用或禁用 [etag](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag) 生成。

```ts
{
  etag: true,
}
```

</dd>

<dt>

  lastModified

</dt>

<dd>

启用或禁用 [Last-Modified](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Last-Modified) 头。文件的 [stat.mtime](https://nodejs.org/api/fs.html#statsmtime) 属性用作头的值。

```ts
{
  lastModified: true,
}
```

</dd>


<dt>

  immutable

</dt>

<dd>

启用或禁用 `Cache-Control` 头的 [immutable](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control#immutable) 指令。默认情况下，`immutable` 属性是禁用的。

如果启用了 `immutable` 属性，你必须定义 `maxAge` 属性以启用缓存。

```ts
{
  immutable: true
}
```

</dd>

<dt>

  maxAge

</dt>

<dd>

为 `Cache-Control` 头定义 [max-age](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control#max-age) 指令。值应为毫秒或时间表达式字符串。

```ts
{
  maxAge: '30 mins'
}
```

</dd>

<dt>

  headers

</dt>

<dd>

一个函数，返回一个对象，其中包含要在响应上设置的头。该函数接收文件路径作为第一个参数，接收 [文件状态](https://nodejs.org/api/fs.html#class-fsstats) 对象作为第二个参数。

```ts
{
  headers: (path, stats) => {
    if (path.endsWith('.mc2')) {
      return {
        'content-type': 'application/octet-stream'
      }
    }
  }
}
```

</dd>


</dl>

## 提供静态文件服务

注册中间件后，你可以在 `public` 目录中创建文件，并通过浏览器使用文件路径访问它们。例如，可以通过 `http://localhost:3333/css/style.css` URL 访问 `./public/css/style.css` 文件。

`public` 目录中的文件不会使用资源打包工具进行编译或构建。如果你想编译前端资源，必须将它们放置在 `resources` 目录中，并使用 [资源打包工具](../basics/vite.md)。

## 将静态文件复制到生产构建中

运行 `node ace build` 命令时，存储在 `/public` 目录中的静态文件会自动复制到 `build` 文件夹中。

复制公共文件的规则在 `adonisrc.ts` 文件中定义。

```ts
{
  metaFiles: [
    {
      pattern: 'public/**',
      reloadServer: false
    }
  ]
}
```
