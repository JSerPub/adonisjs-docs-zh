---
summary: 了解如何在 AdonisJS 中实现 CORS 以保护你的应用程序。
---

# CORS

[CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) 帮助你保护应用程序免受浏览器环境中脚本触发的恶意请求的攻击。

例如，如果从不同域向你的服务器发送 AJAX 或 fetch 请求，浏览器会因跨源资源共享（CORS）错误而阻止该请求，如果您认为该请求应被允许，则需要您设置 CORS 策略。。

在 AdonisJS 中，你可以使用 `@adonisjs/cors` 包来实现 CORS 策略。该包附带了一个 HTTP 中间件，用于拦截传入请求并返回正确的 CORS 头。

## 安装

使用以下命令安装并配置该包：

```sh
node ace add @adonisjs/cors
```

:::disclosure{title="查看 add 命令执行的步骤"}

1. 使用检测到的包管理器安装 `@adonisjs/cors` 包。

2. 在 `adonisrc.ts` 文件中注册以下服务提供者。

    ```ts
    {
      providers: [
        // ...其他提供者
        () => import('@adonisjs/cors/cors_provider')
      ]
    }
    ```

3. 创建 `config/cors.ts` 文件。该文件包含 CORS 的配置设置。

4. 在 `start/kernel.ts` 文件中注册以下中间件。

    ```ts
    server.use([
      () => import('@adonisjs/cors/cors_middleware')
    ])
    ```

:::

## 配置

CORS 中间件的配置存储在 `config/cors.ts` 文件中。

```ts
import { defineConfig } from '@adonisjs/cors'

const corsConfig = defineConfig({
  enabled: true,
  origin: true,
  methods: ['GET', 'HEAD', 'POST', 'PUT', 'DELETE'],
  headers: true,
  exposeHeaders: [],
  credentials: true,
  maxAge: 90,
})

export default corsConfig
```

<dl>

<dt>

enabled

</dt>

<dd>

在不从中间件堆栈中移除的情况下，临时启用或禁用中间件。

</dd>

<dt>

origin

</dt>

<dd>

`origin` 属性控制 [Access-Control-Allow-Origin](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) 头的值。

通过将值设置为 `true` 来允许请求的当前源，或通过将值设置为 `false` 来禁止请求的当前源。

```ts
{
  origin: true
}
```

你可以指定一个硬编码的源列表，以允许一组域名。

```ts
{
  origin: ['adonisjs.com']
}
```

使用通配符表达式 `*` 来允许所有源。阅读 [MDN 文档](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin#directives) 以了解通配符表达式的工作原理。

当 `credentials` 属性设置为 `true` 时，通配符表达式会自动被当作 `布尔值（true）` 来处理。

```ts
{
  origin: '*'
}
```

你可以在 HTTP 请求期间使用函数计算 `origin` 值。例如：

```ts
{
  origin: (requestOrigin, ctx) => {
    return true
  }
}
```

</dd>

<dt>

methods

</dt>

<dd>

`methods` 属性控制在预检请求期间允许的方法。[Access-Control-Request-Method](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Request-Method) 头的值会与允许的方法进行比较。

```sh
{
  methods: ['GET', 'HEAD', 'POST', 'PUT', 'DELETE']
}
```

</dd>

<dt>

headers

</dt>

<dd>

`headers` 属性控制在预检请求期间允许的请求头。[Access-Control-Request-Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Request-Headers) 头的值会与 headers 属性进行比较。

将值设置为 `true` 将允许所有头。而将值设置为 `false` 将禁止所有头。

```ts
{
  headers: true
}
```

你可以通过定义字符串数组来指定允许的头列表。

```ts
{
  headers: [
    'Content-Type',
    'Accept',
    'Cookie'
  ]
}
```

你可以在 HTTP 请求期间使用函数计算 `headers` 配置值。例如：

```ts
{
  headers: (requestHeaders, ctx) => {
    return true
  }
}
```

</dd>

<dt>

exposeHeaders

</dt>

<dd>

`exposeHeaders` 属性控制在预检请求期间通过 [Access-Control-Expose-Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Expose-Headers) 头暴露的头。

```ts
{
  exposeHeaders: [
    'cache-control',
    'content-language',
    'content-type',
    'expires',
    'last-modified',
    'pragma',
  ]
}
```

</dd>

<dt>

credentials

</dt>

<dd>

`credentials` 属性控制在预检请求期间是否设置 [Access-Control-Allow-Credentials](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials) 头。

```ts
{
  credentials: true
}
```

</dd>

<dt>

maxAge

</dt>

<dd>

`maxAge` 属性控制 [Access-Control-Max-Age](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Max-Age) 响应头。该值的单位为秒。

- 将值设置为 `null` 将不设置该头。
- 而将其设置为 `-1` 会设置该头但禁用缓存。

```ts
{
  maxAge: 90
}
```

</dd>

</dl>

## 调试 CORS 错误

调试 CORS 问题是一项具有挑战性的任务。然而，除了理解 CORS 的规则并调试响应头以确保一切就绪之外，没有捷径可走。

以下是一些文章链接，你可以阅读以更好地理解 CORS 的工作原理。

- [How to Debug Any CORS Error](https://httptoolkit.com/blog/how-to-debug-cors-errors/)
- [Will it CORS?](https://httptoolkit.com/will-it-cors/)
- [MDN in-depth explanation of CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
