---
summary: 了解如何使用 AdonisJS 的 Transmit 包通过 SSE 发送实时更新
---

# Transmit

Transmit 是为 AdonisJS 构建的一个原生且带有观点的服务器发送事件 (SSE) 模块。它是一种简单高效的方式，用于向客户端发送实时更新，例如通知、实时聊天消息或任何其他类型的实时数据。

:::note
数据传输仅从服务器到客户端，反之则不行。要实现客户端到服务器的通信，您必须使用表单或 fetch 请求。
:::

## 安装

使用以下命令安装并配置该包：

```sh
node ace add @adonisjs/transmit
```

:::disclosure{title="查看 add 命令执行的步骤"}

1. 使用检测到的包管理器安装 `@adonisjs/transmit` 包。
 
2. 在 `adonisrc.ts` 文件中注册 `@adonisjs/transmit/transmit_provider` 服务提供者。
 
3. 在 `config` 目录中创建一个新的 `transmit.ts` 文件。
 
:::

您还需要安装 Transmit 客户端包，以便在客户端监听事件。

```sh
npm install @adonisjs/transmit-client
```

## 配置

Transmit 包的配置存储在 `config/transmit.ts` 文件中。

另请参阅：[配置存根](https://github.com/adonisjs/transmit/blob/main/stubs/config/transmit.stub)

```ts
import { defineConfig } from '@adonisjs/transmit'

export default defineConfig({
  pingInterval: false,
  transport: null,
})
```

<dl>

<dt>

pingInterval

</dt>

<dd>

用于向客户端发送 ping 消息的时间间隔。该值以毫秒为单位或使用字符串 `Duration` 格式（例如：`10s`）。设置为 `false` 以禁用 ping 消息。

</dd>

<dt>

transport

</dt>

<dd>

Transmit 支持在多个服务器或实例之间同步事件。您可以通过引用所需的传输层来启用此功能（目前仅支持 `redis`）。设置为 `null` 以禁用同步。

```ts
import env from '#start/env'
import { defineConfig } from '@adonisjs/transmit'
import { redis } from '@adonisjs/transmit/transports'

export default defineConfig({
  transport: {
    driver: redis({
      host: env.get('REDIS_HOST'),
      port: env.get('REDIS_PORT'),
      password: env.get('REDIS_PASSWORD'),
      keyPrefix: 'transmit',
    })
  }
})
```

:::note
使用 `redis` 传输时，请确保已安装 `ioredis`。
:::

</dd>

</dl>

## 注册路由

您必须注册 transmit 路由，以允许客户端连接到服务器。路由需要手动注册。

```ts
// title: start/routes.ts
import transmit from '@adonisjs/transmit/services/main'

transmit.registerRoutes()
```

您也可以通过手动绑定控制器来手动注册每个路由。

```ts
// title: start/routes.ts
const EventStreamController = () => import('@adonisjs/transmit/controllers/event_stream_controller')
const SubscribeController = () => import('@adonisjs/transmit/controllers/subscribe_controller')
const UnsubscribeController = () => import('@adonisjs/transmit/controllers/unsubscribe_controller')

router.get('/__transmit/events', [EventStreamController])
router.post('/__transmit/subscribe', [SubscribeController])
router.post('/__transmit/unsubscribe', [UnsubscribeController])
```

如果您想修改路由定义，例如使用 [`Rate Limiter`](../security/rate_limiting.md) 和身份验证中间件来避免某些 transmit 路由被滥用，您可以更改路由定义或向 `transmit.registerRoutes` 方法传递一个回调函数。

```ts
// title: start/routes.ts
import transmit from '@adonisjs/transmit/services/main'

transmit.registerRoutes((route) => {
  // 确保您已认证才能注册客户端
  if (route.getPattern() === '__transmit/events') {
    route.middleware(middleware.auth())
    return
  }

  // 为其他 transmit 路由添加限流中间件
  route.use(throttle)
})
```

## 频道

频道用于对事件进行分组。例如，您可以有一个用于通知的频道，另一个用于聊天消息的频道，依此类推。当客户端订阅它们时，频道会动态创建。

### 频道名称

频道名称用于标识频道。它们是区分大小写的，并且必须是字符串。在频道名称中，除了 `/` 之外，不能使用任何特殊字符或空格。以下是一些有效的频道名称示例：

```ts
import transmit from '@adonisjs/transmit/services/main'

transmit.broadcast('global', { message: 'Hello' })
transmit.broadcast('chats/1/messages', { message: 'Hello' })
transmit.broadcast('users/1', { message: 'Hello' })
```

:::tip
频道名称使用与 AdonisJS 中路由相同的语法，但与它们无关。您可以自由地为 HTTP 路由和频道定义相同的键。
:::

### 频道授权

您可以使用 `authorize` 方法授权或拒绝连接到频道。该方法接收频道名称和 `HttpContext`，并且必须返回布尔值。

```ts
// title: start/transmit.ts

import transmit from '@adonisjs/transmit/services/main'
import Chat from '#models/chat'
import type { HttpContext } from '@adonisjs/core/http'

transmit.authorize<{ id: string }>('users/:id', (ctx: HttpContext, { id }) => {
  return ctx.auth.user?.id === +id
})

transmit.authorize<{ id: string }>('chats/:id/messages', async (ctx: HttpContext, { id }) => {
  const chat = await Chat.findOrFail(+id)
  
  return ctx.bouncer.allows('accessChat', chat)
})
```

## 广播事件

您可以使用 `broadcast` 方法向频道广播事件。该方法接收频道名称和要发送的数据。

```ts
import transmit from '@adonisjs/transmit/services/main'

transmit.broadcast('global', { message: 'Hello' })
```

您还可以使用 `broadcastExcept` 方法向除一个之外的任何频道广播事件。该方法接收频道名称、要发送的数据以及您想要忽略的 UID。

```ts
transmit.broadcastExcept('global', { message: 'Hello' }, 'uid-of-sender')
```

### 在多个服务器或实例之间同步

默认情况下，广播事件仅在 HTTP 请求的上下文中工作。但是，如果您在配置中注册了 `transport`，则可以使用 `transmit` 服务在后台广播事件。

传输层负责在多个服务器或实例之间同步事件。它通过广播任何事件（如广播事件、订阅和取消订阅）到所有连接的服务器或实例来实现，使用 `Message Bus`。

负责您客户端连接的服务器或实例将接收事件并将其广播给客户端。

## Transmit 客户端

您可以使用 `@adonisjs/transmit-client` 包在客户端监听事件。该包提供了一个 `Transmit` 类。客户端默认使用 [`EventSource`](https://developer.mozilla.org/en-US/docs/Web/API/EventSource) API 连接到服务器。

```ts
import { Transmit } from '@adonisjs/transmit-client'

export const transmit = new Transmit({
  baseUrl: window.location.origin
})
```

:::tip
您应该只创建 `Transmit` 类的一个实例，并在整个应用程序中重复使用它。
:::

### 配置 Transmit 实例

`Transmit` 类接受一个包含以下属性的对象：

<dl>

<dt>

baseUrl

</dt>

<dd>

服务器的基本 URL。URL 必须包括协议（http 或 https）和域名。

</dd>

<dt>

uidGenerator

</dt>

<dd>

一个为客户端生成唯一标识符的函数。该函数必须返回一个字符串。默认值为 `crypto.randomUUID`。

</dd>

<dt>

eventSourceFactory

</dt>

<dd>

一个创建新的 `EventSource` 实例的函数。默认值为 WebAPI [`EventSource`](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)。如果您想在 `Node.js`、`React Native` 或任何其他不支持 `EventSource` API 的环境中使用客户端，则需要提供自定义实现。

</dd>

<dt>

eventTargetFactory 

</dt>

<dd>

一个创建新的 `EventTarget` 实例的函数。默认值为 WebAPI [`EventTarget`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget)。如果您想在 `Node.js`、`React Native` 或任何其他不支持 `EventTarget` API 的环境中使用客户端，则需要提供自定义实现。返回 `null` 以禁用 `EventTarget` API。

</dd>

<dt>

httpClientFactory

</dt>

<dd>

一个创建新的 `HttpClient` 实例的函数。主要用于测试目的。

</dd>

<dt>

beforeSubscribe

</dt>

<dd>

在订阅频道之前调用的函数。它接收频道名称和发送到服务器的 `Request` 对象。使用此函数添加自定义头或修改请求对象。

</dd>

<dt>

beforeUnsubscribe

</dt>

<dd>

在取消订阅频道之前调用的函数。它接收频道名称和发送到服务器的 `Request` 对象。使用此函数添加自定义头或修改请求对象。

</dd>

<dt>

maxReconnectAttempts

</dt>

<dd>

最大重连尝试次数。默认值为 `5`。

</dd>

<dt>

onReconnectAttempt

</dt>

<dd>

在每次重连尝试之前调用的函数，并接收迄今为止已进行的尝试次数。使用此函数添加自定义逻辑。

</dd>

<dt>

onReconnectFailed

</dt>

<dd>

当重连尝试失败时调用的函数。使用此函数添加自定义逻辑。

</dd>

<dt>

onSubscribeFailed

</dt>

<dd>

当订阅失败时调用的函数。它接收 `Response` 对象。使用此函数添加自定义逻辑。

</dd>

<dt>

onSubscription

</dt>

<dd>

当订阅成功时调用的函数。它接收频道名称。使用此函数添加自定义逻辑。

</dd>

<dt>

onUnsubscription

</dt>

<dd>

当取消订阅成功时调用的函数。它接收频道名称。使用此函数添加自定义逻辑。

</dd>

</dl>

### 创建订阅

您可以使用 `subscription` 方法创建对频道的订阅。该方法接收频道名称。

```ts
const subscription = transmit.subscription('chats/1/messages')
await subscription.create()
```

`create` 方法在服务器上注册订阅。它返回一个 promise，您可以 `await` 或 `void`。

:::note
如果您不调用 `create` 方法，订阅将不会在服务器上注册，并且您将不会收到任何事件。
:::

### 监听事件

您可以使用 `onMessage` 方法监听订阅上的事件，该方法接收一个回调函数。您可以多次调用 `onMessage` 方法以添加不同的回调函数。

```ts
subscription.onMessage((data) => {
  console.log(data)
})
```

您还可以使用 `onMessageOnce` 方法仅监听频道一次，该方法接收一个回调函数。

```ts
subscription.onMessageOnce(() => {
  console.log('I will be called only once')
})
```

### 停止监听事件

`onMessage` 和 `onMessageOnce` 方法返回一个函数，您可以调用该函数以停止监听特定的回调。

```ts
const stopListening = subscription.onMessage((data) => {
  console.log(data)
})

// 停止监听
stopListening()
```

### 删除订阅

您可以使用 `delete` 方法删除订阅。该方法返回一个 promise，您可以 `await` 或 `void`。此方法将在服务器上注销订阅。

```ts
await subscription.delete()
```

## 避免 GZip 干扰

在部署使用 `@adonisjs/transmit` 的应用程序时，重要的是确保 GZip 压缩不会干扰服务器发送事件 (SSE) 使用的 `text/event-stream` 内容类型。对 `text/event-stream` 应用的压缩可能导致连接问题，从而导致频繁断开连接或 SSE 失败。

如果您的部署使用反向代理（如 Traefik 或 Nginx）或其他应用 GZip 的中间件，请确保对 `text/event-stream` 内容类型禁用压缩。

### Traefik 的示例配置

```plaintext
traefik.http.middlewares.gzip.compress=true
traefik.http.middlewares.gzip.compress.excludedcontenttypes=text/event-stream
traefik.http.routers.my-router.middlewares=gzip
```
