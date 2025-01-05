---
summary: 了解 AdonisJS 框架核心和官方包所分派的事件。
---

# 事件参考

在本指南中，我们将查看框架核心和官方包所分派的事件列表。请参阅 [emitter](../digging_deeper/emitter.md) 文档，以了解更多关于其用法的信息。

## http\:request_completed

[`http:request_completed`](https://github.com/adonisjs/http-server/blob/main/src/types/server.ts#L65) 事件在 HTTP 请求完成后分派。该事件包含一个 [HttpContext](../concepts/http_context.md) 的实例和请求持续时间。`duration` 值是 `process.hrtime` 方法的输出。

```ts
import emitter from '@adonisjs/core/services/emitter'
import string from '@adonisjs/core/helpers/string'

emitter.on('http:request_completed', (event) => {
  const method = event.ctx.request.method()
  const url = event.ctx.request.url(true)
  const duration = event.duration

  console.log(`${method} ${url}: ${string.prettyHrTime(duration)}`)
})
```

## http\:server_ready

该事件在 AdonisJS HTTP 服务器准备好接受传入请求时分派。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('http:server_ready', (event) => {
  console.log(event.host)
  console.log(event.port)

  /**
   * 启动应用程序和启动
   * HTTP 服务器所用的时间。
   */
  console.log(event.duration)
})
```

## container_binding\:resolved

该事件在 IoC 容器解析绑定或构造类实例后分派。`event.binding` 属性将是一个字符串（绑定名称）或类构造函数，`event.value` 属性是解析后的值。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('container_binding:resolved', (event) => {
  console.log(event.binding)
  console.log(event.value)
})
```

## session\:initiated

`@adonisjs/session` 包在 HTTP 请求期间会话存储被初始化时分派该事件。`event.session` 属性是 [Session class](https://github.com/adonisjs/session/blob/main/src/session.ts) 的一个实例。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('session:initiated', (event) => {
  console.log(`Initiated store for ${event.session.sessionId}`)
})
```

## session\:committed

`@adonisjs/session` 包在 HTTP 请求期间会话数据被写入会话存储时分派该事件。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('session:committed', (event) => {
  console.log(`Persisted data for ${event.session.sessionId}`)
})
```

## session\:migrated

`@adonisjs/session` 包在使用 `session.regenerate()` 方法生成新的会话 ID 时分派该事件。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('session:migrated', (event) => {
  console.log(`Migrating data to ${event.toSessionId}`)
  console.log(`Destroying session ${event.fromSessionId}`)
})
```

## i18n\:missing\:translation

当特定键和区域设置的翻译缺失时，`@adonisjs/i18n` 包分派该事件。你可以监听此事件以查找给定区域设置中缺失的翻译。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('i18n:missing:translation', function (event) {
  console.log(event.identifier)
  console.log(event.hasFallback)
  console.log(event.locale)
})
```

## mail\:sending

`@adonisjs/mail` 包在发送电子邮件之前分派该事件。在调用 `mail.sendLater` 方法的情况下，当邮件队列处理作业时，将分派该事件。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('mail:sending', (event) => {
  console.log(event.mailerName)
  console.log(event.message)
  console.log(event.views)
})
```

## mail\:sent

发送电子邮件后，`@adonisjs/mail` 包分派该事件。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('mail:sent', (event) => {
  console.log(event.response)

  console.log(event.mailerName)
  console.log(event.message)
  console.log(event.views)
})
```

## mail\:queueing

`@adonisjs/mail` 包在将发送电子邮件的作业排队之前分派该事件。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('mail:queueing', (event) => {
  console.log(event.mailerName)
  console.log(event.message)
  console.log(event.views)
})
```

## mail\:queued

电子邮件已排队后，`@adonisjs/mail` 包分派该事件。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('mail:queued', (event) => {
  console.log(event.mailerName)
  console.log(event.message)
  console.log(event.views)
})
```

## queued\:mail\:error

当 `@adonisjs/mail` 包的 [MemoryQueue](https://github.com/adonisjs/mail/blob/main/src/messengers/memory_queue.ts) 实现无法使用 `mail.sendLater` 方法发送已排队的电子邮件时，分派该事件。

如果你使用的是自定义队列实现，你必须捕获作业错误并分派此事件。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('queued:mail:error', (event) => {
  console.log(event.error)
  console.log(event.mailerName)
})
```

## session_auth\:login_attempted

当通过会话守卫直接或内部调用 `auth.login` 方法时，`@adonisjs/auth` 包的 [SessionGuard](https://github.com/adonisjs/auth/blob/main/src/guards/session/guard.ts) 实现会分发此事件。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('session_auth:login_attempted', (event) => {
  console.log(event.guardName)
  console.log(event.user)
})
```

## session_auth\:login_succeeded

用户成功登录后，`@adonisjs/auth` 包的 [SessionGuard](https://github.com/adonisjs/auth/blob/main/src/guards/session/guard.ts) 实现会分发此事件。

你可以使用此事件来跟踪与给定用户关联的会话。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('session_auth:login_succeeded', (event) => {
  console.log(event.guardName)
  console.log(event.sessionId)
  console.log(event.user)
  console.log(event.rememberMeToken) // （如果已创建）
})
```

## session_auth\:authentication_attempted

当尝试验证请求会话并检查已登录用户时，`@adonisjs/auth` 包会分发此事件。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('session_auth:authentication_attempted', (event) => {
  console.log(event.guardName)
  console.log(event.sessionId)
})
```

## session_auth\:authentication_succeeded

请求会话验证通过且用户已登录后，`@adonisjs/auth` 包会分发此事件。你可以通过 `event.user` 属性访问已登录用户。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('session_auth:authentication_succeeded', (event) => {
  console.log(event.guardName)
  console.log(event.sessionId)

  console.log(event.user)
  console.log(event.rememberMeToken) // 如果使用令牌进行身份验证
})
```

## session_auth\:authentication_failed

当身份验证检查失败且在当前 HTTP 请求中用户未登录时，`@adonisjs/auth` 包会分发此事件。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('session_auth:authentication_failed', (event) => {
  console.log(event.guardName)
  console.log(event.sessionId)

  console.log(event.error)
})
```

## session_auth\:logged_out

用户注销后，`@adonisjs/auth` 包会分发此事件。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('session_auth:logged_out', (event) => {
  console.log(event.guardName)
  console.log(event.sessionId)

  /**
   * 如果在请求期间调用注销，而该请求中原本就没有用户登录，
   * 则 user 的值将为 null。
   */
  console.log(event.user)
})
```

## access_tokens_auth\:authentication_attempted

当尝试在 HTTP 请求期间验证访问令牌时，`@adonisjs/auth` 包会分发此事件。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('access_tokens_auth:authentication_attempted', (event) => {
  console.log(event.guardName)
})
```

## access_tokens_auth\:authentication_succeeded

访问令牌验证通过后，`@adonisjs/auth` 包会分发此事件。你可以通过 `event.user` 属性访问已验证用户。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('access_tokens_auth:authentication_succeeded', (event) => {
  console.log(event.guardName)
  console.log(event.user)
  console.log(event.token)
})
```

## access_tokens_auth\:authentication_failed

当身份验证检查失败时，`@adonisjs/auth` 包会分发此事件。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('access_tokens_auth:authentication_failed', (event) => {
  console.log(event.guardName)
  console.log(event.error)
})
```

## authorization\:finished

授权检查执行后，`@adonisjs/bouncer` 包会分发此事件。事件负载包括最终响应，你可以检查该响应以了解检查的状态。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('authorization:finished', (event) => {
  console.log(event.user)
  console.log(event.response)
  console.log(event.parameters)
  console.log(event.action) 
})
```
