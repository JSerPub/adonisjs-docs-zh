---
summary: 使用 @adonisjs/session 包在你的 AdonisJS 应用程序中管理用户会话。
---

# 会话

你可以使用 `@adonisjs/session` 包在你的 AdonisJS 应用程序中管理用户会话。会话包提供了一个统一的 API，用于在不同的存储提供者之间存储会话数据。

**以下是捆绑的存储列表**：

- `cookie`：会话数据存储在加密的 cookie 中。cookie 存储在多服务器部署中效果很好，因为数据存储在客户端。

- `file`：会话数据保存在服务器上的一个文件中。只有通过负载均衡器实现粘性会话，文件存储才能扩展至多服务器部署。

- `redis`：会话数据存储在 Redis 数据库中。对于会话数据量大的应用程序，建议使用 redis 存储，并且可以扩展到多服务器部署。

- `dynamodb`：会话数据存储在 Amazon DynamoDB 表中。DynamoDB 存储适用于需要高度可扩展和分布式会话存储的应用程序，特别是在基础设施建立在 AWS 上时。

- `memory`：会话数据存储在全局内存存储中。内存存储用于测试期间。

除了内置的后端存储，你还可以创建和 [注册自定义会话存储](#creating-a-custom-session-store)。

## 安装

使用以下命令安装并配置包：

```sh
node ace add @adonisjs/session
```

:::disclosure{title="查看 add 命令执行的步骤"}

1. 使用检测到的包管理器安装 `@adonisjs/session` 包。

2. 在 `adonisrc.ts` 文件中注册以下服务提供者。

    ```ts
    {
      providers: [
        // ...其他提供者
        () => import('@adonisjs/session/session_provider')
      ]
    }
    ```

3. 创建 `config/session.ts` 文件。

4. 定义以下环境变量及其验证。

    ```dotenv
    SESSION_DRIVER=cookie
    ```

5. 在 `start/kernel.ts` 文件中注册以下中间件。

    ```ts
    router.use([
      () => import('@adonisjs/session/session_middleware')
    ])
    ```

:::

## 配置

会话包的配置存储在 `config/session.ts` 文件中。

另请参阅：[会话配置存根](https://github.com/adonisjs/session/blob/main/stubs/config/session.stub)

```ts
import env from '#start/env'
import app from '@adonisjs/core/services/app'
import { defineConfig, stores } from '@adonisjs/session'

export default defineConfig({
  age: '2h',
  enabled: true,
  cookieName: 'adonis-session',
  clearWithBrowser: false,

  cookie: {
    path: '/',
    httpOnly: true,
    secure: app.inProduction,
    sameSite: 'lax',
  },

  store: env.get('SESSION_DRIVER'),
  stores: {
    cookie: stores.cookie(),
  }
})
```

<dl>

<dt>

  enabled

</dt>

<dd>

临时启用或禁用中间件，而不从中间件堆栈中移除它。

</dd>

<dt>

  cookieName

</dt>

<dd>

用于存储会话 ID 的 cookie 名称。可以随意重命名。

</dd>

<dt>

  clearWithBrowser

</dt>

<dd>

设置为 true 时，用户关闭浏览器窗口后，会话 ID cookie 将被移除。这种 cookie 在技术上被称为 [会话 cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#define_the_lifetime_of_a_cookie)。

</dd>

<dt>

  age

</dt>

<dd>

`age` 属性控制无用户活动时会话数据的有效性。在给定时长后，会话数据被视为过期。

</dd>

<dt>

  cookie

</dt>

<dd>

控制会话 ID cookie 属性。另请参阅 [cookie 配置](./cookies.md#configuration)。

</dd>

<dt>

  store

</dt>

<dd>

定义要用于存储会话数据的存储。它可以是固定值，也可以从环境变量中读取。

</dd>

<dt>

  stores

</dt>

<dd>

`stores` 对象用于配置一个或多个后端存储。

大多数应用程序将使用单个存储。但是，你可以配置多个存储，并根据应用程序运行的环境在它们之间切换。

</dd>

</dl>

---

### 存储配置

以下是 `@adonisjs/session` 包捆绑的后端存储列表。

```ts
import app from '@adonisjs/core/services/app'
import { defineConfig, stores } from '@adonisjs/session'

export default defineConfig({
  store: env.get('SESSION_DRIVER'),

  // highlight-start
  stores: {
    cookie: stores.cookie(),

    file: stores.file({
      location: app.tmpPath('sessions')
    }),

    redis: stores.redis({
      connection: 'main'
    })

    dynamodb: stores.dynamodb({
      clientConfig: {}
    }),
  }
  // highlight-end
})
```

<dl>

<dt>

  stores.cookie

</dt>

<dd>

`cookie` 存储加密并将会话数据存储在 cookie 中。

</dd>

<dt>

  stores.file

</dt>

<dd>

定义 `file` 存储的配置。该方法接受用于存储会话文件的 `location` 路径。

</dd>

<dt>

  stores.redis

</dt>

<dd>

定义 `redis` 存储的配置。该方法接受用于存储会话数据的 `connection` 名称。

在使用 `redis` 存储之前，请确保首先安装并配置 [@adonisjs/redis](../database/redis.md) 包。

</dd>

<dt>

  stores.dynamodb

</dt>

<dd>

定义 `dynamodb` 存储的配置。你可以通过 `clientConfig` 属性传递 [DynamoDB 配置](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/Package/-aws-sdk-client-dynamodb/Interface/DynamoDBClientConfig/)，或者通过 `client` 属性传递 DynamoDB 的实例。

```ts
// title: 使用 client 配置
stores.dynamodb({
  clientConfig: {
    region: 'us-east-1',
    endpoint: '<database-endpoint>',
    credentials: {
      accessKeyId: '',
      secretAccessKey: '',
    }
  },
})
```

```ts
// title: 使用 client 实例
import { DynamoDBClient } from '@aws-sdk/client-dynamodb'
const client = new DynamoDBClient({})

stores.dynamodb({
  client,
})
```

此外，你可以定义自定义表名和键属性名。

```ts
stores.dynamodb({
  tableName: 'Session'
  keyAttributName: 'key'
})
```

</dd>

</dl>

---

### 更新环境变量验证

如果你决定使用除默认存储以外的会话存储，请确保也更新 `SESSION_DRIVER` 环境变量的验证。

在以下示例中，我们配置了 `cookie`、`redis` 和 `dynamodb` 存储。因此，我们也应该允许 `SESSION_DRIVER` 环境变量是其中之一。

```ts
import { defineConfig, stores } from '@adonisjs/session'

export default defineConfig({
  // highlight-start
  store: env.get('SESSION_DRIVER'),

  stores: {
    cookie: stores.cookie(),
    redis: stores.redis({
      connection: 'main'
    })
  }
  // highlight-end
})
```

```ts
// title: start/env.ts
{
  SESSION_DRIVER: Env.schema.enum(['cookie', 'redis', 'memory'] as const)
}
```

## 基本示例
一旦会话包注册后，你可以从 [HTTP 上下文](../concepts/http_context.md) 中访问 `session` 属性。`session` 属性暴露了用于读写会话存储数据的 API。

```ts
import router from '@adonisjs/core/services/router'

router.get('/theme/:color', async ({ params, session, response }) => {
  // highlight-start
  session.put('theme', params.color)
  // highlight-end
  response.redirect('/')
})

router.get('/', async ({ session }) => {
  // highlight-start
  const colorTheme = session.get('theme')
  // highlight-end
  return `You are using ${colorTheme} color theme`
})
```

请求开始时从会话存储中读取会话数据，并在请求结束时写回存储。因此，所有更改在请求完成前都保存在内存中。

## 支持的数据类型
会话数据使用 `JSON.stringify` 序列化为字符串；因此，你可以将以下 JavaScript 数据类型用作会话值。

- string
- number
- bigInt
- boolean
- null
- object
- array

```ts
// 对象
session.put('user', {
  id: 1,
  fullName: 'virk',
})

// 数组
session.put('product_ids', [1, 2, 3, 4])

// 布尔值
session.put('is_logged_in', true)

// 数字
session.put('visits', 10)

// BigInt
session.put('visits', BigInt(10))

// 数据对象转换为 ISO 字符串
session.put('visited_at', new Date())
```

## 读写数据
以下是可以用于与 `session` 对象中的数据交互的方法列表。

### get
从存储中返回键的值。你可以使用点符号读取嵌套值。

```ts
session.get('key')
session.get('user.email')
```

你还可以定义第二个参数作为默认值。如果键在存储中不存在，将返回默认值。

```ts
session.get('visits', 0)
```

### has
检查键是否存在于会话存储中。

```ts
if (session.has('visits')) {
}
```

### all
返回会话存储中的所有数据。返回值始终是一个对象。

```ts
session.all()
```

### put
向会话存储中添加键值对。你可以使用点符号创建具有嵌套值的对象。

```ts
session.put('user', { email: 'foo@bar.com' })

// 与上面相同
session.put('user.email', 'foo@bar.com')
```

### forget
从会话存储中移除键值对。

```ts
session.forget('user')

// 从 user 对象中移除 email
session.forget('user.email')
```

### pull
`pull` 方法返回键的值并同时从存储中移除它。

```ts
const user = session.pull('user')
session.has('user') // false
```

### increment
`increment` 方法增加键的值。如果键不存在，则定义新的键值。

```ts
session.increment('visits')

// 增加 4
session.increment('visits', 4)
```

### decrement
`decrement` 方法减少键的值。如果键不存在，则定义新的键值。

```ts
session.decrement('visits')

// 减少 4
session.decrement('visits', 4)
```

### clear
`clear` 方法从会话存储中移除所有内容。

```ts
session.clear()
```

## 会话生命周期
AdonisJS 在第一个 HTTP 请求时创建一个空的会话存储，并将其分配给一个唯一的会话 ID，即使请求/响应生命周期不与会话交互。

在每次后续请求中，我们更新会话 ID cookie 的 `maxAge` 属性，以确保其不会过期。会话存储也会被通知任何更改（如果有），以便更新和持久化它们。

你可以使用 `sessionId` 属性访问唯一的会话 ID。访问者的会话 ID 在其过期前保持不变。

```ts
console.log(session.sessionId)
```

### 重新生成会话 ID
重新生成会话 ID 有助于防止应用程序中的 [会话固定](https://owasp.org/www-community/attacks/Session_fixation) 攻击。在将匿名会话与登录用户关联时，必须重新生成会话 ID。

`@adonisjs/auth` 包会自动重新生成会话 ID，因此你无需手动执行此操作。

```ts
/**
 * 在请求结束时分配新的会话 ID
 */
session.regenerate()
```

## 闪存消息
闪存消息用于在两个 HTTP 请求之间传递数据。它们通常用于在特定操作后向用户提供反馈。例如，在表单提交后显示成功消息或显示验证错误消息。

在以下示例中，我们定义了用于显示联系表单和将表单详细信息提交到数据库的路由。表单提交后，我们使用闪存消息将用户重定向回表单，并附带成功通知。

```ts
import router from '@adonisjs/core/services/router'

router.post('/contact', ({ session, request, response }) => {
  const data = request.all()
  // 保存联系数据
  
  // highlight-start
  session.flash('notification', {
    type: 'success',
    message: 'Thanks for contacting. We will get back to you'
  })
  // highlight-end

  response.redirect().back()
})

router.get('/contact', ({ view }) => {
  return view.render('contact')
})
```

你可以在 Edge 模板中使用 `flashMessage` 标签或 `flashMessages` 属性访问闪存消息。

```edge
@flashMessage('notification')
  <div class="notification {{ $message.type }}">
    {{ $message.message }}
  </div>
@end

<form method="POST" action="/contact">
  <!-- 表单的其余部分 -->
</form>
```

你可以在控制器中使用 `session.flashMessages` 属性访问闪存消息。

```ts
router.get('/contact', ({ view, session }) => {
  // highlight-start
  console.log(session.flashMessages.all())
  // highlight-end
  return view.render('contact')
})
```

### 验证错误和闪存消息
会话中间件会自动捕获 [验证异常](./validation.md#error-handling)，并将用户重定向回表单。验证错误和表单输入数据保存在闪存消息中，你可以在 Edge 模板中访问它们。

在以下示例中：

- 我们使用 [`old` 方法](../references/edge.md#old) 访问 `title` 输入字段的值。
- 使用 [`@inputError` 标签](../references/edge.md#inputerror) 访问错误消息。

```edge
<form method="POST" action="/posts">
  <div>
    <label for="title"> Title </label>
    <input 
      type="text"
      id="title"
      name="title"
      value="{{ old('title') || '' }}"
    />

    @inputError('title')
      @each(message in $messages)
        <p> {{ message }} </p>
      @end
    @end
  </div>
</form>
```

### 写入闪存消息
以下是将数据写入闪存消息存储的方法。`session.flash` 方法接受键值对并将其写入会话存储中的闪存消息属性。

```ts
session.flash('key', value)
session.flash({
  key: value
})
```

你可以使用以下方法之一来快速闪存表单数据，而不是手动读取请求数据并将其存储在闪存消息中。

```ts
// title: flashAll
/**
 * 快速闪存请求数据的简写
 */
session.flashAll()

/**
 * 与 "flashAll" 相同
 */
session.flash(request.all())
```

```ts
// title: flashOnly
/**
 * 快速闪存请求数据中选定属性的简写
 */
session.flashOnly(['username', 'email'])

/**
 * 与 "flashOnly" 相同
 */
session.flash(request.only(['username', 'email']))
```

```ts
// title: flashExcept
/**
 * 快速闪存请求数据中选定属性的简写
 */
session.flashExcept(['password'])

/**
 * 与 "flashExcept" 相同
 */
session.flash(request.except(['password']))
```

最后，你可以使用 `session.reflash` 方法重新闪存当前的闪存消息。

```ts
session.reflash()
session.reflashOnly(['notification', 'errors'])
session.reflashExcept(['errors'])
```

### 读取闪存消息

闪存消息仅在重定向后的后续请求中可用。你可以通过 `session.flashMessages` 属性访问它们。

```ts
console.log(session.flashMessages.all())
console.log(session.flashMessages.get('key'))
console.log(session.flashMessages.has('key'))
```

相同的 `flashMessages` 属性也可以在 Edge 模板中使用，访问方式如下：

参见：[Edge 辅助函数参考](../references/edge.md#flashmessages)

```edge
{{ flashMessages.all() }}
{{ flashMessages.get('key') }}
{{ flashMessages.has('key') }}
```

最后，你可以使用以下 Edge 标签访问特定的闪存消息或验证错误。

```edge
{{-- 通过键读取任何闪存消息 --}}
@flashMessage('key')
  {{ inspect($message) }}
@end

{{-- 读取通用错误 --}}
@error('key')
  {{ inspect($message) }}
@end

{{-- 读取验证错误 --}}
@inputError('key')
  {{ inspect($messages) }}
@end
```

## 事件
请查阅 [事件参考指南](../references/events.md#sessioninitiated)，以查看 `@adonisjs/session` 包分派的事件列表。

## 创建自定义会话存储
会话存储必须实现 [SessionStoreContract](https://github.com/adonisjs/session/blob/main/src/types.ts#L23C18-L23C38) 接口，并定义以下方法。

```ts
import {
  SessionData,
  SessionStoreFactory,
  SessionStoreContract,
} from '@adonisjs/session/types'

/**
 * 你希望接受的配置
 */
export type MongoDBConfig = {}

/**
 * 驱动程序实现
 */
export class MongoDBStore implements SessionStoreContract {
  constructor(public config: MongoDBConfig) {
  }

  /**
   * 返回会话 ID 的会话数据。该方法
   * 必须返回 null 或一个键值对对象
   */
  async read(sessionId: string): Promise<SessionData | null> {
  }

  /**
   * 根据提供的会话 ID 保存会话数据
   */
  async write(sessionId: string, data: SessionData): Promise<void> {
  }

  /**
   * 删除给定会话 ID 的会话数据
   */
  async destroy(sessionId: string): Promise<void> {
  }

  /**
   * 重置会话过期时间
   */
  async touch(sessionId: string): Promise<void> {
  }
}

/**
 * 工厂函数，用于在配置文件中
 * 引用存储。
 */
export function mongoDbStore (config: MongoDbConfig): SessionStoreFactory {
  return (ctx, sessionConfig) => {
    return new MongoDBStore(config)
  }
}
```

在上面的代码示例中，我们导出了以下值：

- `MongoDBConfig`：你希望接受的配置的 TypeScript 类型。

- `MongoDBStore`：作为类的存储实现。它必须遵循 `SessionStoreContract` 接口。

- `mongoDbStore`：最后，一个工厂函数，用于为每个 HTTP 请求创建存储实例。

### 使用存储

创建存储后，你可以在配置文件中使用 `mongoDbStore` 工厂函数引用它。

```ts
// title: config/session.ts
import { defineConfig } from '@adonisjs/session'
import { mongDbStore } from 'my-custom-package'

export default defineConfig({
  stores: {
    mongodb: mongoDbStore({
      // 配置在此处
    })
  }
})
```

### 关于数据序列化的说明

`write` 方法接收的会话数据是一个对象，你可能需要在保存之前将其转换为字符串。你可以使用任何序列化包，或者使用 AdonisJS 辅助模块提供的 [MessageBuilder](../references/helpers.md#message-builder) 辅助函数。有关更多信息，请参阅官方的 [session stores](https://github.com/adonisjs/session/blob/main/src/stores/redis.ts#L59)。