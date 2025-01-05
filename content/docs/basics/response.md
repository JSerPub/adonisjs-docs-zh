---
summary: Response 类用于发送 HTTP 响应。它支持发送 HTML 片段、JSON 对象、流等。
---

# 响应

[响应类](https://github.com/adonisjs/http-server/blob/main/src/response.ts)的实例用于响应 HTTP 请求。AdonisJS 支持发送 **HTML 片段**、**JSON 对象**、**流** 等。可以通过 `ctx.response` 属性访问响应实例。

## 发送响应

发送响应的最简单方法是从路由处理程序返回一个值。

```ts
import router from '@adonisjs/core/services/router'

router.get('/', async () => {
  /** 纯字符串 */
  return 'This is the homepage.'

  /** HTML 片段 */
  return '<p> This is the homepage </p>'

  /** JSON 响应 */
  return { page: 'home' }

  /** 转换为 ISO 字符串 */
  return new Date()
})
```

除了从路由处理程序返回一个值外，还可以使用 `response.send` 方法显式设置响应体。但是，多次调用 `response.send` 方法会覆盖旧的响应体，只保留最新的响应体。

```ts
import router from '@adonisjs/core/services/router'

router.get('/', async ({ response }) => {
  /** 纯字符串 */
  response.send('This is the homepage')

  /** HTML 片段 */
  response.send('<p> This is the homepage </p>')

  /** JSON 响应 */
  response.send({ page: 'home' })

  /** 转换为 ISO 字符串 */
  response.send(new Date())
})
```

可以使用 `response.status` 方法为响应设置自定义状态码。

```ts
response.status(200).send({ page: 'home' })

// 发送空的 201 响应
response.status(201).send('')
```

## 流式内容

`response.stream` 方法允许将流管道传输到响应中。该方法在流完成后会自动销毁流。

`response.stream` 方法不会设置 `content-type` 和 `content-length` 头；你必须在流式传输内容之前显式设置它们。

```ts
import router from '@adonisjs/core/services/router'
import fs from 'fs'

router.get('/', async ({ response }) => {
  const image = fs.createReadStream('./some-file.jpg')
  response.stream(image)
})
```

如果出现错误，将向客户端发送 500 状态码。但是，你可以通过将回调作为第二个参数来自定义错误码和消息。

```ts
const image = fs.createReadStream('./some-file.jpg')

response.stream(image, () => {
  const message = 'Unable to serve file. Try again'
  const status = 400

  return [message, status]
})
```

## 下载文件

当你想从磁盘流式传输文件时，建议使用 `response.download` 方法而不是 `response.stream` 方法。这是因为 `download` 方法会自动设置 `content-type` 和 `content-length` 头。

```ts
import app from '@adonisjs/core/services/app'
import router from '@adonisjs/core/services/router'

router.get('/uploads/:file', async ({ response, params }) => {
  const filePath = app.makePath(`uploads/${params.file}`)

  response.download(filePath)
})
```

可选地，你可以为文件内容生成一个 [Etag](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag)。使用 Etag 将帮助浏览器重用之前请求中的缓存响应（如果有）。

```ts
const filePath = app.makePath(`uploads/${params.file}`)
const generateEtag = true

response.download(filePath, generateEtag)
```

与 `response.stream` 方法类似，你可以通过将回调作为最后一个参数来发送自定义错误消息和状态码。

```ts
const filePath = app.makePath(`uploads/${params.file}`)
const generateEtag = true

response.download(filePath, generateEtag, (error) => {
  if (error.code === 'ENOENT') {
    return ['File does not exists', 404]
  }

  return ['Cannot download file', 400]
})
```

### 强制下载文件

`response.attachment` 方法与 `response.download` 方法类似，但它通过设置 [Content-Disposition](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition) 头强制浏览器将文件保存在用户的计算机上。

```ts
import app from '@adonisjs/core/services/app'
import router from '@adonisjs/core/services/router'

router.get('/uploads/:file', async ({ response, params }) => {
  const filePath = app.makePath(`uploads/${params.file}`)

  response.attachment(filePath, 'custom-filename.jpg')
})
```

## 设置响应状态和头

### 设置状态

可以使用 `response.status` 方法设置响应状态。调用此方法将覆盖现有的响应状态（如果有）。但是，可以使用 `response.safeStatus` 方法仅在状态为 `undefined` 时设置状态。

```ts
import router from '@adonisjs/core/services/router'

router.get('/', async ({ response }) => {
  /**
   * 将状态设置为 200
   */
  response.safeStatus(200)

  /**
   * 不设置状态，因为它已经设置
   */
  response.safeStatus(201)
})
```

### 设置头

可以使用 `response.header` 方法设置响应头。此方法将覆盖现有的头值（如果已存在）。但是，可以使用 `response.safeHeader` 方法仅在头为 `undefined` 时设置头。

```ts
import router from '@adonisjs/core/services/router'

router.get('/', async ({ response }) => {
  /**
   * 定义 content-type 头
   */
  response.safeHeader('Content-type', 'text/html')

  /**
   * 不设置 content-type 头，因为它已经设置
   */
  response.safeHeader('Content-type', 'text/html')
})
```

可以使用 `response.append` 方法将值附加到现有头值。

```ts
response.append('Set-cookie', 'cookie-value')
```

`response.removeHeader` 方法删除现有的头。

```ts
response.removeHeader('Set-cookie')
```

### X-Request-Id 头

如果当前请求中存在该头，或者启用了 [生成请求 ID](./request#generating-request-ids)，则该头将存在于响应中。

## 重定向

`response.redirect` 方法返回一个 [Redirect](https://github.com/adonisjs/http-server/blob/main/src/redirect.ts) 类的实例。重定向类使用流畅的 API 来构建重定向 URL。

执行重定向的最简单方法是使用 `redirect.toPath` 方法并传递重定向路径。

```ts
import router from '@adonisjs/core/services/router'

router.get('/posts', async ({ response }) => {
  response.redirect().toPath('/articles')
})
```

重定向类还允许从预注册的路由构建 URL。`redirect.toRoute` 方法接受 [路由标识符](./routing.md#route-identifier) 作为第一个参数，路由参数作为第二个参数。

```ts
import router from '@adonisjs/core/services/router'

router.get('/articles/:id', async () => {}).as('articles.show')

router.get('/posts/:id', async ({ response, params }) => {
  response.redirect().toRoute('articles.show', { id: params.id })
})
```

### 重定向回上一页

在表单提交期间出现验证错误时，你可能希望将用户重定向回上一页。可以使用 `redirect.back` 方法实现这一点。

```ts
response.redirect().back()
```

### 重定向状态码

重定向响应的默认状态码是 `302`；可以通过调用 `redirect.status` 方法来更改它。

```ts
response.redirect().status(301).toRoute('articles.show', { id: params.id })
```

### 带查询字符串的重定向

可以使用 `withQs` 方法将查询字符串附加到重定向 URL。该方法接受一个键值对对象并将其转换为字符串。

```ts
response.redirect().withQs({ page: 1, limit: 20 }).toRoute('articles.index')
```

要将当前请求 URL 中的查询字符串转发，请在不传递任何参数的情况下调用 `withQs` 方法。

```ts
// 转发当前 URL 查询字符串
response.redirect().withQs().toRoute('articles.index')
```

在重定向回上一页时，`withQs` 方法将转发上一页的查询字符串。

```ts
// 转发当前 URL 查询字符串
response.redirect().withQs().back()
```

## 使用错误中止请求

可以使用 `response.abort` 方法通过引发异常来结束请求。该方法将抛出 `E_HTTP_REQUEST_ABORTED` 异常并触发 [异常处理](./exception_handling.md) 流程。

```ts
router.get('posts/:id/edit', async ({ response, auth, params }) => {
  const post = await Post.findByOrFail(params.id)

  if (!auth.user.can('editPost', post)) {
    response.abort({ message: 'Cannot edit post' })
  }

  // 继续执行其余逻辑
})
```

默认情况下，异常将创建一个具有 `400` 状态码的 HTTP 响应。但是，可以将自定义状态码作为第二个参数指定。

```ts
response.abort({ message: 'Cannot edit post' }, 403)
```

## 在响应完成后运行操作

可以使用 `response.onFinish` 方法监听 Node.js 将响应写入 TCP 套接字的事件。在内部，我们使用 [on-finished](https://github.com/jshttp/on-finished) 包，因此你可以参考该包的 README 文件以获取深入的技术解释。

```ts
router.get('posts', ({ response }) => {
  response.onFinish(() => {
    // 清理逻辑
  })
})
```

## 访问 Node.js `res` 对象

可以使用 `response.response` 属性访问 [Node.js res 对象](https://nodejs.org/dist/latest-v19.x/docs/api/http.html#class-httpserverresponse)。

```ts
router.get('posts', ({ response }) => {
  console.log(response.response)
})
```

## 响应体序列化

使用 `response.send` 方法设置的响应体在 [写入响应](https://nodejs.org/dist/latest-v18.x/docs/api/http.html#responsewritechunk-encoding-callback)到传出消息流之前会被序列化为字符串。

以下是支持的数据类型及其序列化规则。

- 数组和对象使用 [安全字符串化函数](https://github.com/poppinss/utils/blob/main/src/json/safe_stringify.ts)进行字符串化。该方法类似于 `JSON.stringify`，但会移除循环引用并序列化 `BigInt(s)`。
- 数字和布尔值被转换为字符串。
- Date 类的实例通过调用 `toISOString` 方法转换为字符串。
- 正则表达式和错误对象通过调用 `toString` 方法转换为字符串。
- 其他任何数据类型都会导致异常。

### 内容类型推断

序列化响应后，响应类会自动推断并设置 `content-type` 和 `content-length` 头。

以下是设置 `content-type` 头的规则列表。

- 数组和对象的内容类型设置为 `application/json`。
- HTML 片段的内容类型设置为 `text/html`。
- JSONP 响应使用 `text/javascript` 内容类型发送。
- 其他所有内容的内容类型设置为 `text/plain`。

## 扩展 Response 类

你可以使用宏或 getter 向 Response 类添加自定义属性。如果你对宏的概念不熟悉，请务必先阅读 [扩展 AdonisJS 指南](../concepts/extending_the_framework.md)。

```ts
import { Response } from '@adonisjs/core/http'

Response.macro('property', function (this: Response) {
  return value
})
Response.getter('property', function (this: Response) {
  return value
})
```

由于宏和 getter 是在运行时添加的，因此你必须向 TypeScript 告知它们的类型。

```ts
declare module '@adonisjs/core/http' {
  export interface Response {
    property: valueType
  }
}
```