---
summary: 使用 `@adonisjs/redis` 包在 AdonisJS 应用程序中集成 Redis。
---

# Redis

你可以使用 `@adonisjs/redis` 包在 AdonisJS 应用程序中集成 Redis。该包是基于 [ioredis](https://github.com/redis/ioredis) 的轻量级封装，提供了更好的发布/订阅（Pub/Sub）开发体验和自动管理多个 Redis 连接的功能。

## 安装

使用以下命令安装并配置该包：

```sh
node ace add @adonisjs/redis
```

:::disclosure{title="查看 add 命令执行的步骤"}

1. 使用检测到的包管理器安装 `@adonisjs/redis` 包。

2. 在 `adonisrc.ts` 文件中注册以下服务提供者。

    ```ts
    {
      providers: [
        // ...其他服务提供者
        () => import('@adonisjs/redis/redis_provider')
      ]
    }
    ```

3. 创建 `config/redis.ts` 文件。该文件包含你的 Redis 服务器的连接配置。

4. 定义以下环境变量及其验证规则。

    ```dotenv
    REDIS_HOST=127.0.0.1
    REDIS_PORT=6379
    REDIS_PASSWORD=
    ```

:::

## 配置

Redis 包的配置存储在 `config/redis.ts` 文件中。

另请参阅：[配置文件存根](https://github.com/adonisjs/redis/blob/main/stubs/config/redis.stub)

```ts
import env from '#start/env'
import { defineConfig } from '@adonisjs/redis'

const redisConfig = defineConfig({
  connection: 'main',
  connections: {
    main: {
      host: env.get('REDIS_HOST'),
      port: env.get('REDIS_PORT'),
      password: env.get('REDIS_PASSWORD', ''),
      db: 0,
      keyPrefix: '',
    },
  },
})

export default redisConfig
```

<dl>
<dt>

connection

<dt>

<dd>

`connection` 属性定义了默认使用的连接。当你运行 Redis 命令而没有选择明确的连接时，它们将在默认连接上执行。

</dd>

<dt>

connections

<dt>

<dd>

`connections` 属性是多个命名连接的集合。你可以在这个对象中定义一个或多个连接，并使用 `redis.connection()` 方法在它们之间切换。

每个命名连接配置与 [ioredis 接受的配置](https://redis.github.io/ioredis/index.html#RedisOptions) 相同。

</dd>
</dl>

---

### 配置集群

如果你在连接配置中定义了一个主机数组，`@adonisjs/redis` 包将创建一个[集群连接](https://github.com/redis/ioredis#cluster)。例如：

```ts
const redisConfig = defineConfig({
  connections: {
    main: {
      // highlight-start
      clusters: [
        { host: '127.0.0.1', port: 6380 },
        { host: '127.0.0.1', port: 6381 },
      ],
      clusterOptions: {
        scaleReads: 'slave',
        slotsRefreshTimeout: 10 * 1000,
      },
      // highlight-end
    },
  },
})
```

### 配置哨兵

你可以在连接配置中定义哨兵节点数组，以配置 Redis 连接使用哨兵。例如：

另请参阅：[IORedis 文档中的哨兵配置](https://github.com/redis/ioredis?tab=readme-ov-file#sentinel)

```ts
const redisConfig = defineConfig({
  connections: {
    main: {
      // highlight-start
      sentinels: [
        { host: 'localhost', port: 26379 },
        { host: 'localhost', port: 26380 },
      ],
      name: 'mymaster',
      // highlight-end
    },
  },
})
```

## 使用

你可以使用包导出的 `redis` 服务运行 Redis 命令。`redis` 服务是一个单例对象，根据你在 `config/redis.ts` 文件中定义的配置进行配置。

:::note

查阅 [ioredis](https://redis.github.io/ioredis/classes/Redis.html) 文档以查看可用方法的列表。由于我们是基于 IORedis 的封装，因此命令 API 是相同的。

:::

```ts
import redis from '@adonisjs/redis/services/main'

await redis.set('username', 'virk')
const username = await redis.get('username')
```

### 切换连接

使用 `redis` 服务执行的命令是在配置文件中定义的**默认连接**上调用的。但是，你可以通过首先获取特定连接的实例来在该连接上执行命令。

`.connection()` 方法为进程的生命周期创建并缓存一个连接实例。

```ts
import redis from '@adonisjs/redis/services/main'

// highlight-start
// 获取连接实例
const redisMain = redis.connection('main')
// highlight-end

await redisMain.set('username', 'virk')
const username = await redisMain.get('username')
```

### 关闭连接

连接是长连接的，并且每次调用 `.connection()` 方法时都会获得相同的实例。你可以使用 `quit` 方法关闭连接。使用 `disconnect` 方法可以强制结束连接。

```ts
import redis from '@adonisjs/redis/services/main'

await redis.quit('main') // 关闭主连接
await redis.disconnect('main') // 强制关闭主连接
```

```ts
import redis from '@adonisjs/redis/services/main'

const redisMain = redis.connection('main')
redisMain.quit() // 使用连接实例关闭
redisMain.disconnect() // 使用连接实例强制关闭
```

## 错误处理

Redis 连接在应用程序生命周期中的任何时候都可能失败。因此，捕获错误并制定重试策略是至关重要的。

默认情况下，AdonisJS 将使用[应用程序日志记录器](../digging_deeper/logger.md)记录 Redis 连接错误，并在永久关闭连接之前重试连接十次。重试策略是在 `config/redis.ts` 文件中为每个连接定义的。

另请参阅：[IORedis 文档中的自动重连](https://github.com/redis/ioredis#auto-reconnect)

```ts
// title: config/redis.ts
{
  main: {
    host: env.get('REDIS_HOST'),
    port: env.get('REDIS_PORT'),
    password: env.get('REDIS_PASSWORD', ''),
    // highlight-start
    retryStrategy(times) {
      return times > 10 ? null : times * 50
    },
    // highlight-end
  },
}
```

您可以使用 `.doNotLogErrors` 方法禁用默认的错误报告器。这样做将从 redis 连接中移除 `error` 事件监听器。

```ts
import redis from '@adonisjs/redis/services/main'

/**
 * 禁用默认错误报告器
 */
redis.doNotLogErrors()

redis.on('connection', (connection) => {
  /**
   * 确保始终定义一个错误监听器。
   * 否则，应用程序将崩溃
   */
  connection.on('error', (error) => {
    console.log(error)
  })
})
```

## 发布/订阅

Redis 需要多个连接来发布和订阅频道。订阅者连接只能执行订阅新频道/模式和取消订阅的操作。

使用 `@adonisjs/redis` 包时，您无需手动创建订阅者连接；我们会为您处理。当您第一次调用 `subscribe` 方法时，我们将自动创建一个新的订阅者连接。

```ts
import redis from '@adonisjs/redis/services/main'

redis.subscribe('user:add', function (message) {
  console.log(message)
})
```

### IORedis 和 AdonisJS 之间的 API 差异

使用 `ioredis` 时，您必须使用两个不同的 API 来订阅频道并监听新消息。然而，使用 AdonisJS 包装器时，`subscribe` 方法会同时处理这两者。

:::caption{for="info"}
**使用 IORedis**
:::

```ts
redis.on('message', (channel, messages) => {
  console.log(message)
})

redis.subscribe('user:add', (error, count) => {
  if (error) {
    console.log(error)
  }
})
```

:::caption{for="info"}
**使用 AdonisJS**
:::

```ts
redis.subscribe('user:add', (message) => {
  console.log(message)
},
{
  onError(error) {
    console.log(error)
  },
  onSubscription(count) {
    console.log(count)
  },
})
```

### 发布消息

您可以使用 `publish` 方法发布消息。该方法接受频道名称作为第一个参数，要发布的数据作为第二个参数。

```ts
redis.publish(
  'user:add',
  JSON.stringify({
    id: 1,
    username: 'virk',
  })
)
```

### 订阅模式

您可以使用 `psubscribe` 方法订阅模式。与 `subscribe` 方法类似，它将创建一个订阅者连接（如果不存在）。

```ts
redis.psubscribe('user:*', (channel, message) => {
  console.log(channel)
  console.log(message)
})

redis.publish(
  'user:add',
  JSON.stringify({
    id: 1,
    username: 'virk',
  })
)
```

### 取消订阅

您可以使用 `unsubscribe` 和 `punsubscribe` 方法从频道或模式取消订阅。

```ts
await redis.unsubscribe('user:add')
await redis.punsubscribe('user:*add*')
```

## 使用 Lua 脚本

您可以将 Lua 脚本注册为 redis 服务的命令，它们将应用于所有连接。

另请参阅：[IORedis 文档中的 Lua 脚本](https://github.com/redis/ioredis#lua-scripting)

```ts
import redis from '@adonisjs/redis/services/main'

redis.defineCommand('release', {
  numberOfKeys: 2,
  lua: `
    redis.call('zrem', KEYS[2], ARGV[1])
    redis.call('zadd', KEYS[1], ARGV[2], ARGV[1])
    return true
  `,
})
```

定义命令后，您可以使用 `runCommand` 方法执行它。首先定义所有键，然后是参数。

```ts
redis.runCommand(
  'release', // 命令名称
  'jobs:completed', // 键 1
  'jobs:running', // 键 2
  '11023', // argv 1
  100 // argv 2
)
```

相同的命令可以在明确的连接上执行。

```ts
redis.connection('jobs').runCommand(
  'release', // 命令名称
  'jobs:completed', // 键 1
  'jobs:running', // 键 2
  '11023', // argv 1
  100 // argv 2
)
```

最后，您还可以为特定的连接实例定义命令。例如：

```ts
redis.on('connection', (connection) => {
  if (connection.connectionName === 'jobs') {
    connection.defineCommand('release', {
      numberOfKeys: 2,
      lua: `
        redis.call('zrem', KEYS[2], ARGV[1])
        redis.call('zadd', KEYS[1], ARGV[2], ARGV[1])
        return true
      `,
    })
  }
})
```

## 转换参数和回复

您可以使用 `redis.Command` 属性定义参数转换器和回复转换器。API 与 [IORedis API](https://github.com/redis/ioredis#transforming-arguments--replies) 相同。

```ts
// title: 参数转换器
import redis from '@adonisjs/redis/services/main'

redis.Command.setArgumentTransformer('hmset', (args) => {
  if (args.length === 2) {
    if (args[1] instanceof Map) {
      // utils 是 ioredis 的内部模块
      return [args[0], ...utils.convertMapToArray(args[1])]
    }
    if (typeof args[1] === 'object' && args[1] !== null) {
      return [args[0], ...utils.convertObjectToArray(args[1])]
    }
  }
  return args
})
```

```ts
// title: 回复转换器
import redis from '@adonisjs/redis/services/main'

redis.Command.setReplyTransformer('hgetall', (result) => {
  if (Array.isArray(result)) {
    const obj = {}
    for (let i = 0; i < result.length; i += 2) {
      obj[result[i]] = result[i + 1]
    }
    return obj
  }
  return result
})
```
## 事件

以下是 Redis 连接实例发出的事件列表。

### connect / subscriber\:connect
当连接建立时，会发出此事件。当建立订阅者连接时，会发出 `subscriber:connect` 事件。

```ts
import redis from '@adonisjs/redis/services/main'

redis.on('connection', (connection) => {
  connection.on('connect', () => {})
  connection.on('subscriber:connect', () => {})
})
```

### wait
当连接处于 `wait` 模式时发出，因为配置中设置了 `lazyConnect` 选项。执行第一个命令后，连接将从 `wait` 状态移出。

```ts
import redis from '@adonisjs/redis/services/main'

redis.on('connection', (connection) => {
  connection.on('wait', () => {})
})
```

### ready / subscriber\:ready
除非你在配置中启用了 `enableReadyCheck` 标志，否则此事件将在 `connect` 事件之后立即发出。在那种情况下，我们会等待 Redis 服务器报告其已准备好接受命令。

```ts
import redis from '@adonisjs/redis/services/main'

redis.on('connection', (connection) => {
  connection.on('ready', () => {})
  connection.on('subscriber:ready', () => {})
})
```

### error / subscriber\:error
当无法连接到 Redis 服务器时，会发出此事件。请参阅 [错误处理](#error-handling) 以了解 AdonisJS 如何处理连接错误。

```ts
import redis from '@adonisjs/redis/services/main'

redis.on('connection', (connection) => {
  connection.on('error', () => {})
  connection.on('subscriber:error', () => {})
})
```

### close / subscriber\:close
当连接关闭时，会发出此事件。根据重试策略，IORedis 可能会在发出 `close` 事件后尝试重新建立连接。

```ts
import redis from '@adonisjs/redis/services/main'

redis.on('connection', (connection) => {
  connection.on('close', () => {})
  connection.on('subscriber:close', () => {})
})
```

### reconnecting / subscriber\:reconnecting
在 `close` 事件后尝试重新连接到 Redis 服务器时，会发出此事件。

```ts
import redis from '@adonisjs/redis/services/main'

redis.on('connection', (connection) => {
  connection.on('reconnecting', ({ waitTime }) => {
    console.log(waitTime)
  })
  connection.on('subscriber:reconnecting', ({ waitTime }) => {
    console.log(waitTime)
  })
})
```

### end / subscriber\:end
当连接已关闭且不会再进行进一步的重连时，会发出此事件。它应该是连接生命周期的结束。

```ts
import redis from '@adonisjs/redis/services/main'

redis.on('connection', (connection) => {
  connection.on('end', () => {})
  connection.on('subscriber:end', () => {})
})
```

### node\:added
当连接到新的集群节点时（仅适用于集群实例），会发出此事件。

```ts
import redis from '@adonisjs/redis/services/main'

redis.on('connection', (connection) => {
  connection.on('node:added', () => {})
})
```

### node\:removed
当集群节点被移除时（仅适用于集群实例），会发出此事件。

```ts
import redis from '@adonisjs/redis/services/main'

redis.on('connection', (connection) => {
  connection.on('node:removed', () => {})
})
```

### node\:error
当无法连接到集群节点时（仅适用于集群实例），会发出此事件。

```ts
import redis from '@adonisjs/redis/services/main'

redis.on('connection', (connection) => {
  connection.on('node:error', ({ error, address }) => {
    console.log(error, address)
  })
})
```

### subscription\:ready / psubscription\:ready

当给定频道或模式的订阅已建立时，会发出此事件。

```ts
import redis from '@adonisjs/redis/services/main'

redis.on('connection', (connection) => {
  connection.on('subscription:ready', ({ count }) => {
    console.log(count)
  })
  connection.on('psubscription:ready', ({ count }) => {
    console.log(count)
  })
})
```

### subscription\:error / psubscription\:error
当无法订阅频道或模式时，会发出此事件。

```ts
import redis from '@adonisjs/redis/services/main'

redis.on('connection', (connection) => {
  connection.on('subscription:error', () => {})
  connection.on('psubscription:error', () => {})
})
```
