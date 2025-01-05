---
summary: 使用 @adonisjs/limiter 包实现速率限制，以保护你的 Web 应用程序或 API 服务器免受滥用。
---

# 速率限制

AdonisJS 提供了一个官方包，用于在你的 Web 应用程序或 API 服务器中实现速率限制。速率限制器提供 `redis`、`mysql`、`postgresql` 和 `memory` 作为存储选项，并具备 [创建自定义存储提供者](#creating-a-custom-storage-provider) 的能力。

`@adonisjs/limiter` 包建立在 [node-rate-limiter-flexible](https://github.com/animir/node-rate-limiter-flexible) 包之上，该包提供了最快的速率限制 API 之一，并使用原子递增来避免竞态条件。

## 安装

使用以下命令安装并配置该包：

```sh
node ace add @adonisjs/limiter
```

:::disclosure{title="查看 add 命令执行的步骤"}

1. 使用检测到的包管理器安装 `@adonisjs/limiter` 包。

2. 在 `adonisrc.ts` 文件中注册以下服务提供者。
    ```ts
    {
      providers: [
        // ...其他提供者
        () => import('@adonisjs/limiter/limiter_provider')
      ]
    }
    ```

3. 创建 `config/limiter.ts` 文件。

4. 创建 `start/limiter.ts` 文件。该文件用于定义 HTTP 节流中间件。

5. 在 `start/env.ts` 文件中定义以下环境变量及其验证。
   ```ts
   LIMITER_STORE=redis
   ```

6. 如果使用 `database` 存储，请可选地创建 `rate_limits` 表的数据库迁移。

:::

## 配置

速率限制器的配置存储在 `config/limiter.ts` 文件中。

另请参阅：[速率限制器配置存根](https://github.com/adonisjs/limiter/blob/main/stubs/config/limiter.stub)

```ts
import env from '#start/env'
import { defineConfig, stores } from '@adonisjs/limiter'

const limiterConfig = defineConfig({
  default: env.get('LIMITER_STORE'),

  stores: {
    redis: stores.redis({}),

    database: stores.database({
      tableName: 'rate_limits'
    }),

    memory: stores.memory({}),
  },
})

export default limiterConfig

declare module '@adonisjs/limiter/types' {
  export interface LimitersList extends InferLimiters<typeof limiterConfig> {}
}
```

<dl>

<dt>

default

</dt>

<dd>

用于应用速率限制的默认存储。该存储在同一配置文件中的 `stores` 对象下定义。

</dd>

<dt>

stores

</dt>

<dd>

你计划在应用程序中使用的存储集合。我们建议在测试期间始终配置 `memory` 存储。

</dd>

</dl>

---

### 环境变量

默认速率限制器使用 `LIMITER_STORE` 环境变量定义，因此你可以在不同的环境中切换不同的存储。例如，在测试期间使用 `memory` 存储，在开发和生产环境中使用 `redis` 存储。

此外，环境变量必须经过验证，以允许使用预配置的存储之一。验证在 `start/env.ts` 文件中使用 `Env.schema.enum` 规则定义。

```ts
{
  LIMITER_STORE: Env.schema.enum(['redis', 'database', 'memory'] as const),
}
```

### 共享选项

以下是所有捆绑存储共享的选项列表。

<dl>

<dt>

keyPrefix

</dt>

<dd>

为存储在数据库存储中的键定义前缀。数据库存储会忽略 `keyPrefix`，因为可以使用不同的数据库表来隔离数据。

</dd>

<dt>

execEvenly

</dt>

<dd>

`execEvenly` 选项在节流请求时添加延迟，以便所有请求在提供的持续时间结束时耗尽。

例如，如果你允许用户每分钟进行 **10 次请求**，所有请求都将有一个人为延迟，以便第十个请求在 1 分钟结束时完成。阅读 `rate-limiter-flexible` 存储库上的 [smooth out traffic peaks](https://github.com/animir/node-rate-limiter-flexible/wiki/Smooth-out-traffic-peaks) 文章，以了解更多关于 `execEvenly` 选项的信息。

</dd>

<dt>

inMemoryBlockOnConsumed

</dt>

<dd>

定义在内存中阻塞键的请求数量。例如，你允许用户每分钟进行 **10 次请求**，并且他们在前 10 秒内消耗了所有请求。

然而，他们继续向服务器发出请求，因此，速率限制器必须在拒绝请求之前与数据库进行检查。

为了减少数据库负载，你可以定义在停止查询数据库并在内存中阻塞键之前应允许的请求数量。

```ts
{
  duration: '1 minute',
  requests: 10,

  /**
   * 在 12 个请求后，在内存中阻塞键并停止咨询数据库。
   */
  inMemoryBlockOnConsumed: 12,
}
```

</dd>

<dt>

inMemoryBlockDuration

</dt>

<dd>

在内存中阻塞键的持续时间。此选项将减少数据库负载，因为后端存储将首先在内存中检查某个键是否被阻塞。

```ts
{
  inMemoryBlockDuration: '1 min'
}
```

</dd>

</dl>

---

### Redis 存储

`redis` 存储对 `@adonisjs/redis` 包有对等依赖，因此在使用 redis 存储之前，必须配置该包。

以下是 redis 存储接受的选项列表（与共享选项一起）。

```ts
{
  redis: stores.redis({
    connectionName: 'main',
    rejectIfRedisNotReady: false,
  }),
}
```

<dl>

<dt>

connectionName

</dt>

<dd>

`connectionName` 属性引用在 `config/redis.ts` 文件中定义的连接。我们建议使用单独的 redis 数据库进行速率限制。

</dd>

<dt>

rejectIfRedisNotReady

</dt>

<dd>

当 Redis 连接的状态不是 `ready` 时，拒绝速率限制请求。

</dd>

</dl>

---

### 数据库存储

`database` 存储对 `@adonisjs/lucid` 包有对等依赖，因此在使用数据库存储之前，必须配置该包。

以下是数据库存储接受的选项列表（与共享选项一起）。

:::note

数据库存储仅支持 MySQL 和 PostgreSQL 数据库。

:::

```ts
{
  database: stores.database({
    connectionName: 'mysql',
    dbName: 'my_app',
    tableName: 'rate_limits',
    schemaName: 'public',
    clearExpiredByTimeout: false,
  }),
}
```

<dl>

<dt>

connectionName

</dt>

<dd>

引用在 `config/database.ts` 文件中定义的数据库连接。如果未定义，我们将使用默认数据库连接。

</dd>

<dt>

dbName

</dt>

<dd>

用于执行 SQL 查询的数据库。我们尝试从 `config/database.ts` 文件中定义的连接配置推断 `dbName` 的值。但是，如果使用连接字符串，则必须通过此属性提供数据库名称。

</dd>

<dt>

tableName

</dt>

<dd>

用于存储速率限制的数据库表。

</dd>

<dt>

schemaName

</dt>

<dd>

用于执行 SQL 查询的架构（仅适用于 PostgreSQL）。

</dd>

<dt>

clearExpiredByTimeout

</dt>

<dd>

启用时，数据库存储将每 5 分钟清除已过期的键。请注意，只有已过期超过 1 小时的键才会被清除。

</dd>

</dl>

## 节流 HTTP 请求

配置好速率限制器后，你可以使用 `limiter.define` 方法创建 HTTP 节流中间件。`limiter` 服务是使用 `config/limiter.ts` 文件中定义的配置创建的 [LimiterManager](https://github.com/adonisjs/limiter/blob/main/src/limiter_manager.ts) 类的单例实例。

如果你打开 `start/limiter.ts` 文件，会发现一个预定义的全局节流中间件，你可以将其应用于路由或路由组。类似地，你可以在应用程序中根据需要创建任意数量的节流中间件。

在以下示例中，全局节流中间件允许用户根据他们的 IP 地址每分钟进行 **10 次请求**。

```ts
// title: start/limiter.ts
import limiter from '@adonisjs/limiter/services/main'

export const throttle = limiter.define('global', () => {
  return limiter.allowRequests(10).every('1 minute')
})
```

你可以将 `throttle` 中间件应用于路由，如下所示。

```ts
// title: start/routes.ts
import router from '@adonisjs/core/services/router'
// highlight-start
import { throttle } from '#start/limiter'
// highlight-end

router
  .get('/', () => {})
  // highlight-start
  .use(throttle)
  // highlight-end
```

### 动态速率限制

让我们创建另一个中间件来保护 API 端点。这次，我们将根据请求的身份验证状态应用动态速率限制。

```ts
// title: start/limiter.ts
export const apiThrottle = limiter.define('api', (ctx) => {
  /**
   * 允许已登录用户根据其用户 ID 每分钟进行 100 次请求
   */
  if (ctx.auth.user) {
    return limiter
      .allowRequests(100)
      .every('1 minute')
      .usingKey(`user_${ctx.auth.user.id}`)
  }

  /**
   * 允许访客用户根据其 IP 地址每分钟进行 10 次请求
   */
  return limiter
    .allowRequests(10)
    .every('1 minute')
    .usingKey(`ip_${ctx.request.ip()}`)
})
```

```ts
// title: start/routes.ts
import { apiThrottle } from '#start/limiter'

router
  .get('/api/repos/:id/stats', [RepoStatusController])
  .use(apiThrottle)
```

### 切换后端存储

你可以使用 `store` 方法为节流中间件指定特定的后端存储。例如：

```ts
limiter
  .allowRequests(10)
  .every('1 minute')
  // highlight-start
  .store('redis')
  // highlight-end
```

### 使用自定义键

默认情况下，请求会根据用户的 IP 地址进行速率限制。但是，你可以使用 `usingKey` 方法指定自定义键。

```ts
limiter
  .allowRequests(10)
  .every('1 minute')
  // highlight-start
  .usingKey(`user_${ctx.auth.user.id}`)
  // highlight-end
```

### 阻塞用户

如果用户在使用完配额后继续发出请求，你可以使用 `blockFor` 方法将其阻塞指定的持续时间。该方法接受以秒为单位的时间表达式或持续时间。

```ts
limiter
  .allowRequests(10)
  .every('1 minute')
  // highlight-start
  /**
   * 如果他们在 1 分钟内发送超过 10 个请求，将被阻塞 30 分钟
   */
  .blockFor('30 mins')
  // highlight-end
```

## 处理 ThrottleException

当用户在指定时间范围内用完所有请求时，节流中间件将抛出 [E_TOO_MANY_REQUESTS](../references/exceptions.md#e_too_many_requests) 异常。异常将使用以下内容协商规则自动转换为 HTTP 响应。

- 带有 `Accept=application/json` 头的 HTTP 请求将收到一个错误消息数组。每个数组元素将是一个带有 `message` 属性的对象。

- 带有 `Accept=application/vnd.api+json` 头的 HTTP 请求将收到一个根据 JSON API 规范格式化的错误消息数组。

- 所有其他请求将收到纯文本响应消息。但是，你可以使用 [status pages](../basics/exception_handling.md#status-pages) 为速率限制器错误显示自定义错误页面。

你也可以在 [global exception handler](../basics/exception_handling.md#handling-exceptions) 中自行处理该错误。

```ts
import { errors } from '@adonisjs/limiter'
import { HttpContext, ExceptionHandler } from '@adonisjs/core/http'

export default class HttpExceptionHandler extends ExceptionHandler {
  protected debug = !app.inProduction
  protected renderStatusPages = app.inProduction

  async handle(error: unknown, ctx: HttpContext) {
    // highlight-start
    if (error instanceof errors.E_TOO_MANY_REQUESTS) {
      const message = error.getResponseMessage(ctx)
      const headers = error.getDefaultHeaders()

      Object.keys(headers).forEach((header) => {
        ctx.response.header(header, headers[header])
      })

      return ctx.response.status(error.status).send(message)
    }
    // highlight-end

    return super.handle(error, ctx)
  }
}
```

### 自定义错误消息

你可以使用 `limitExceeded` 钩子而不是全局处理异常，来自定义错误消息、状态和响应头。

```ts
import limiter from '@adonisjs/limiter/services/main'

export const throttle = limiter.define('global', () => {
  return limiter
    .allowRequests(10)
    .every('1 minute')
    // highlight-start
    .limitExceeded((error) => {
      error
        .setStatus(400)
        .setMessage('Cannot process request. Try again later')
    })
    // highlight-end
})
```

### 为错误消息使用翻译

如果你已配置 [@adonisjs/i18n](../digging_deeper/i18n.md) 包，可以使用 `errors.E_TOO_MANY_REQUESTS` 键为错误消息定义翻译。例如：

```json
// title: resources/lang/fr/errors.json
{
  "E_TOO_MANY_REQUESTS": "Trop de demandes"
}
```

最后，你可以使用 `error.t` 方法定义自定义翻译键。

```ts
limitExceeded((error) => {
  error.t('errors.rate_limited', {
    limit: error.response.limit,
    remaining: error.response.remaining,
  })
})
```

## 直接使用

除了节流 HTTP 请求外，你还可以将速率限制器用于应用程序的其他部分。例如，如果用户多次提供无效的凭据，则在登录时阻塞用户。或者限制用户可以运行的并发作业数量。

### 创建速率限制器

在将速率限制应用于操作之前，你必须使用 `limiter.use` 方法获取 [Limiter](https://github.com/adonisjs/limiter/blob/main/src/limiter.ts) 类的实例。`use` 方法接受后端存储的名称和以下速率限制选项。

- `requests`：在给定持续时间内允许的请求数。
- `duration`：以秒为单位的时间表达式或 [time expression](../references/helpers.md#seconds) 字符串。
- `block (可选)`：在请求耗尽后阻塞键的持续时间。
- `inMemoryBlockOnConsumed (可选)`：请参阅 [shared options](#shared-options)
- `inMemoryBlockDuration (可选)`：请参阅 [shared options](#shared-options)

```ts
import limiter from '@adonisjs/limiter/services/main'

const reportsLimiter = limiter.use('redis', {
  requests: 1,
  duration: '1 hour'
})
```

如果要使用默认存储，可以省略第一个参数。例如：

```ts
const reportsLimiter = limiter.use({
  requests: 1,
  duration: '1 hour'
})
```

### 对操作应用速率限制

创建速率限制器实例后，可以使用 `attempt` 方法对操作应用速率限制。

该方法接受以下参数：

- 用于速率限制的唯一键。
- 要执行的回调函数，直到所有尝试都已耗尽。

`attempt` 方法返回回调函数的结果（如果已执行）。否则，返回 `undefined`。

```ts
const key = 'user_1_reports'

/**
 * 尝试为给定键运行操作。
 * 如果执行了回调函数，结果将是回调函数的返回值；否则，结果为 undefined。
 */ 
const executed = reportsLimiter.attempt(key, async () => {
  await generateReport()
  return true
})

/**
 * 通知用户他们已超过限制
 */
if (!executed) {
  const availableIn = await reportsLimiter.availableIn(key)
  return `请求过多。请在 ${availableIn} 秒后重试`
}

return '报告已生成'
```

### 防止多次登录失败

直接使用的另一个示例是禁止某个 IP 地址在登录表单上进行多次无效尝试。

在以下示例中，我们使用 `limiter.penalize` 方法在用户提供无效凭据时消耗一次请求，并在所有尝试耗尽后将其阻塞 20 分钟。

`limiter.penalize` 方法接受以下参数：

- 用于速率限制的唯一键。
- 要执行的回调函数。如果函数抛出错误，将消耗一次请求。

`penalize` 方法返回回调函数的结果或 `ThrottleException` 的实例。你可以使用异常来查找直到下一次尝试剩余的时间。

```ts
import User from '#models/user'
import { HttpContext } from '@adonisjs/core/http'
import limiter from '@adonisjs/limiter/services/main'

export default class SessionController {
  async store({ request, response, session }: HttpContext) {
    const { email, password } = request.only(['email', 'passwords'])

    /**
     * 创建速率限制器
     */
    const loginLimiter = limiter.use({
      requests: 5,
      duration: '1 min',
      blockDuration: '20 mins'
    })

    /**
     * 使用 IP 地址 + 电子邮件组合。这确保如果攻击者滥用电子邮件，我们不会阻止实际用户登录，而只会惩罚攻击者的 IP 地址。
     */
    const key = `login_${request.ip()}_${email}`

    /**
     * 将 User.verifyCredentials 包装在 "penalize" 方法中，以便每次无效凭据错误消耗一次请求
     */
    const [error, user] = await loginLimiter.penalize(key, () => {
      return User.verifyCredentials(email, password)
    })

    /**
     * 在 ThrottleException 上，将用户重定向回带有自定义错误消息的页面
     */
    if (error) {
      session.flashAll()
      session.flashErrors({
        E_TOO_MANY_REQUESTS: `登录请求过多。请在 ${error.response.availableIn} 秒后重试`
      })
      return response.redirect().back()
    }

    /**
     * 否则，登录用户
     */
  }
}
```

## 手动消耗请求

除了 `attempt` 和 `penalize` 方法外，你还可以直接与速率限制器交互，以检查剩余请求并手动消耗它们。

在以下示例中，我们使用 `remaining` 方法检查给定键是否已消耗所有请求。否则，使用 `increment` 方法消耗一次请求。

```ts
import limiter from '@adonisjs/limiter/services/main'

const requestsLimiter = limiter.use({
  requests: 10,
  duration: '1 minute'
})

// highlight-start
if (await requestsLimiter.remaining('unique_key') > 0) {
  await requestsLimiter.increment('unique_key')
  await performAction()
} else {
  return '请求过多'
}
// highlight-end
```

在上面的示例中，调用 `remaining` 和 `increment` 方法之间可能会出现竞态条件。因此，你可能希望使用 `consume` 方法。`consume` 方法将增加请求计数，并在所有请求被消耗时抛出异常。

```ts
import { errors } from '@adonisjs/limiter'

try {
  await requestsLimiter.consume('unique_key')
  await performAction()
} catch (error) {
  if (error instanceof errors.E_TOO_MANY_REQUESTS) {
    return '请求过多'
  }
}
```

## 阻塞键

除了消耗请求外，如果用户在所有尝试耗尽后继续发出请求，你还可以将键阻塞更长时间。

在创建带有 `blockDuration` 选项的速率限制器实例时，`consume`、`attempt` 和 `penalize` 方法会自动执行阻塞。例如：

```ts
import limiter from '@adonisjs/limiter/services/main'

const requestsLimiter = limiter.use({
  requests: 10,
  duration: '1 minute',
  // highlight-start
  blockDuration: '30 mins'
  // highlight-end
})

/**
 * 用户可以在一分钟内进行 10 次请求。但是，如果他们发送第 11 个请求，我们将阻塞该键 30 分钟。
 */ 
await requestLimiter.consume('a_unique_key')

/**
 * 与 consume 行为相同
 */
await requestLimiter.attempt('a_unique_key', () => {
})

/**
 * 允许 10 次失败，然后将键阻塞 30 分钟。
 */
await requestLimiter.penalize('a_unique_key', () => {
})
```

最后，你可以使用 `block` 方法将键阻塞给定持续时间。

```ts
const requestsLimiter = limiter.use({
  requests: 10,
  duration: '1 minute',
})

await requestsLimiter.block('a_unique_key', '30 mins')
```

## 重置尝试

你可以使用以下方法之一减少请求数量或从存储中删除整个键。

`decrement` 方法将请求计数减少 1，`delete` 方法删除键。请注意，`decrement` 方法不是原子的，当并发性过高时，可能会将请求计数设置为 `-1`。

```ts
// title: 减少请求计数
import limiter from '@adonisjs/limiter/services/main'

const jobsLimiter = limiter.use({
  requests: 2,
  duration: '5 mins',
})

await jobsLimiter.attempt('unique_key', async () => {
  await processJob()

  /**
   * 处理完作业后，减少已消耗的请求数。这将允许其他工作者使用该槽位。
   */
  // highlight-start
  await jobsLimiter.decrement('unique_key')
  // highlight-end
})
```

```ts
// title: 删除键
import limiter from '@adonisjs/limiter/services/main'

const requestsLimiter = limiter.use({
  requests: 2,
  duration: '5 mins',
})

await requestsLimiter.delete('unique_key')
```

## 测试

如果你使用单个（即默认）存储进行速率限制，可以在 `.env.test` 文件中定义 `LIMITER_STORE` 环境变量，在测试期间切换到 `memory` 存储。

```dotenv
// title: .env.test
LIMITER_STORE=memory
```

你可以使用 `limiter.clear` 方法在测试之间清除速率限制存储。`clear` 方法接受一个存储名称数组并刷新数据库。

在使用 Redis 时，建议使用单独的数据库进行速率限制。否则，`clear` 方法将刷新整个数据库，这可能会影响应用程序的其他部分。

```ts
import limiter from '@adonisjs/limiter/services/main'

test.group('Reports', (group) => {
  // highlight-start
  group.each.setup(() => {
    return () => limiter.clear(['redis', 'memory'])
  })
  // highlight-end
})
```

或者，你可以不带任何参数调用 `clear` 方法，所有配置的存储都将被清除。

```ts
test.group('Reports', (group) => {
  group.each.setup(() => {
    // highlight-start
    return () => limiter.clear()
    // highlight-end
  })
})
```

## 创建自定义存储提供者

自定义存储提供者必须实现 [LimiterStoreContract](https://github.com/adonisjs/limiter/blob/main/src/types.ts#L163) 接口，并定义以下属性/方法。

你可以在任何文件/文件夹中编写实现。创建自定义存储不需要服务提供者。

```ts
import string from '@adonisjs/core/helpers/string'
import { LimiterResponse } from '@adonisjs/limiter'
import {
  LimiterStoreContract,
  LimiterConsumptionOptions
} from '@adonisjs/limiter/types'

/**
 * 你想要接受的自定义选项集。
 */
export type MongoDbLimiterConfig = {
  client: MongoDBConnection
}

export class MongoDbLimiterStore implements LimiterStoreContract {
  readonly name = 'mongodb'
  declare readonly requests: number
  declare readonly duration: number
  declare readonly blockDuration: number

  constructor(public config: MongoDbLimiterConfig & LimiterConsumptionOptions) {
    this.request = this.config.requests
    this.duration = string.seconds.parse(this.config.duration)
    this.blockDuration = string.seconds.parse(this.config.blockDuration)
  }

  /**
   * 为给定键消耗一次请求。当所有请求都已消耗时，
   * 此方法应抛出错误。
   */
  async consume(key: string | number): Promise<LimiterResponse> {
  }

  /**
   * 为给定键消耗一次请求，但在所有请求都已消耗时不抛出错误。
   */
  async increment(key: string | number): Promise<LimiterResponse> {}

  /**
   * 为给定键奖励一次请求。如果可能，不要将请求计数设置为负值。
   */
  async decrement(key: string | number): Promise<LimiterResponse> {}

  /**
   * 将键阻塞指定持续时间。
   */ 
  async block(
    key: string | number,
    duration: string | number
  ): Promise<LimiterResponse> {}
  
  /**
   * 为给定键设置已消耗请求的数量。如果未提供显式持续时间，
   * 则应从配置中推断持续时间。
   */ 
  async set(
    key: string | number,
    requests: number,
    duration?: string | number
  ): Promise<LimiterResponse> {}

  /**
   * 从存储中删除键
   */
  async delete(key: string | number): Promise<boolean> {}

  /**
   * 从数据库中刷新所有键
   */
  async clear(): Promise<void> {}

  /**
   * 获取给定键的速率限制器响应。如果键不存在，则返回 `null`。
   */
  async get(key: string | number): Promise<LimiterResponse | null> {}
}
```

### 定义配置助手

编写完实现后，必须创建一个配置助手，以便在 `config/limiter.ts` 文件中使用提供者。配置助手应返回一个 `LimiterManagerStoreFactory` 函数。

你可以在与 `MongoDbLimiterStore` 实现相同的文件中编写以下函数。

```ts
import { LimiterManagerStoreFactory } from '@adonisjs/limiter/types'

/**
 * 在配置文件中使用 mongoDb 存储的配置助手
 */
export function mongoDbStore(config: MongoDbLimiterConfig) {
  const storeFactory: LimiterManagerStoreFactory = (runtimeOptions) => {
    return new MongoDbLimiterStore({
      ...config,
      ...runtimeOptions
    })
  }
}
```

### 使用配置助手

完成后，可以按如下方式使用 `mongoDbStore` 助手。

```ts
// title: config/limiter.ts
import env from '#start/env'
// highlight-start
import { mongoDbStore } from 'my-custom-package'
// highlight-end
import { defineConfig } from '@adonisjs/limiter'

const limiterConfig = defineConfig({
  default: env.get('LIMITER_STORE'),

  stores: {
    // highlight-start
    mongodb: mongoDbStore({
      client: mongoDb // 创建 mongoDb 客户端
    })
    // highlight-end
  },
})
```

### 包装 rate-limiter-flexible 驱动

如果你计划包装 [node-rate-limiter-flexible](https://github.com/animir/node-rate-limiter-flexible?tab=readme-ov-file#docs-and-examples) 包中的现有驱动，则可以使用 [RateLimiterBridge](https://github.com/adonisjs/limiter/blob/main/src/stores/bridge.ts) 进行实现。

这次让我们使用桥接重新实现相同的 `MongoDbLimiterStore`。

```ts
import { RateLimiterBridge } from '@adonisjs/limiter'
import { RateLimiterMongo } from 'rate-limiter-flexible'

export class MongoDbLimiterStore extends RateLimiterBridge {
  readonly name = 'mongodb'

  constructor(public config: MongoDbLimiterConfig & LimiterConsumptionOptions) {
    super(
      new RateLimiterMongo({
        storeClient: config.client,
        points: config.requests,
        duration: string.seconds.parse(config.duration),
        blockDuration: string.seconds.parse(this.config.blockDuration)
        // ... 提供其他选项
      })
    )
  }

  /**
   * 自我实现 clear 方法。理想情况下，使用
   * config.client 发出删除查询
   */
  async clear() {}
}
```
