---
summary: 了解如何使用 AdonisJS 日志器将日志写入控制台、文件和外部服务。该日志器基于 Pino 构建，速度快且支持多个目标。
---

# Logger

AdonisJS 内置了一个日志器，支持将日志写入 **文件**、**标准输出** 和 **外部日志服务**。在底层，我们使用 [pino](https://getpino.io/#/)。Pino 是 Node.js 生态系统中最快的日志记录库之一，生成 [NDJSON 格式](https://github.com/ndjson/ndjson-spec) 的日志。

## 使用方法

首先，你可以导入 Logger 服务，以便在应用程序中的任何位置写入日志。日志将被写入 `stdout`，并显示在终端上。

```ts
import logger from '@adonisjs/core/services/logger'

logger.info('this is an info message')
logger.error({ err: error }, 'Something went wrong')
```

建议在 HTTP 请求期间使用 `ctx.logger` 属性。HTTP 上下文持有一个请求感知日志器的实例，该实例会将当前请求 ID 添加到每个日志语句中。

```ts
import router from '@adonisjs/core/services/router'
import User from '#models/user'

router.get('/users/:id', async ({ logger, params }) => {
  logger.info('Fetching user by id %s', params.id)
  const user = await User.find(params.id)
})
```

## 配置

日志器的配置存储在 `config/logger.ts` 文件中。默认情况下，只配置了一个日志器。但是，如果你想在应用程序中使用多个日志器，可以定义多个日志器的配置。


```ts
// title: config/logger.ts
import env from '#start/env'
import { defineConfig } from '@adonisjs/core/logger'

export default defineConfig({
  default: 'app',
  
  loggers: {
    app: {
      enabled: true,
      name: env.get('APP_NAME'),
      level: env.get('LOG_LEVEL', 'info')
    },
  }
})
```

<dl>
<dt>

default

<dt>

<dd>

`default` 属性是对同一文件中 `loggers` 对象下配置的某个日志器的引用。 

除非在使用日志器 API 时选择了特定的日志器，否则将使用默认日志器来写入日志。

</dd>

<dt>

loggers

<dt>

<dd>

`loggers` 对象是一个键值对，用于配置多个日志器。键是日志器的名称，值是 [pino](https://getpino.io/#/docs/api?id=options) 所接受的配置对象。

</dd>
</dl>



## 传输目标
在 pino 中，传输扮演着将日志写入目标的重要角色。你可以在配置文件中配置 [多个目标](https://getpino.io/#/docs/api?id=transport-object)，pino 会将日志传递给所有这些目标。每个目标还可以指定一个级别，用于接收日志。

:::note

如果在目标配置中未定义 `level`，则配置的目标将从父日志器继承它。

此行为与 pino 不同。在 Pino 中，目标不会从父日志器继承级别。

:::

```ts
{
  loggers: {
    app: {
      enabled: true,
      name: env.get('APP_NAME'),
      level: env.get('LOG_LEVEL', 'info'),
      
      // highlight-start
      transport: {
        targets: [
          {
            target: 'pino/file',
            level: 'info',
            options: {
              destination: 1
            }
          },
          {
            target: 'pino-pretty',
            level: 'info',
            options: {}
          },
        ]
      }
      // highlight-end
    }
  }
}
```

<dl>
<dt>

File target（文件目标）

<dt>

<dd>

`pino/file` 目标将日志写入文件描述符。`destination = 1` 表示将日志写入 `stdout`（这是标准的 [unix 文件描述符约定](https://en.wikipedia.org/wiki/File_descriptor))）。

</dd>

<dt>

Pretty target（美化目标）

<dt>

<dd>

`pino-pretty` 目标使用 [pino-pretty npm 模块](http://npmjs.com/package/pino-pretty) 将日志美化后写入文件描述符。

</dd>
</dl>

## 有条件地定义目标

通常，会根据代码运行的环境来注册目标。例如，在开发环境中使用 `pino-pretty` 目标，在生产环境中使用 `pino/file` 目标。

如下所示，使用条件语句构造 `targets` 数组会使配置文件显得杂乱无章。

```ts
import app from '@adonisjs/core/services/app'

loggers: {
  app: {
    transport: {
      targets: [
        ...(!app.inProduction
          ? [{ target: 'pino-pretty', level: 'info' }]
          : []
        ),
        ...(app.inProduction
          ? [{ target: 'pino/file', level: 'info' }]
          : []
        ),
      ]
    }
  } 
}
```

因此，你可以使用 `targets` 辅助工具，通过流畅的 API 定义条件数组项。在下面的示例中，我们使用 `targets.pushIf` 方法表达了相同的条件。

```ts
import { targets, defineConfig } from '@adonisjs/core/logger'

loggers: {
  app: {
    transport: {
      targets: targets()
       .pushIf(
         !app.inProduction,
         { target: 'pino-pretty', level: 'info' }
       )
       .pushIf(
         app.inProduction,
         { target: 'pino/file', level: 'info' }
       )
       .toArray()
    }
  } 
}
```

为了进一步简化代码，你可以使用 `targets.pretty` 和 `targets.file` 方法为 `pino/file` 和 `pino-pretty` 目标定义配置对象。

```ts
import { targets, defineConfig } from '@adonisjs/core/logger'

loggers: {
  app: {
    transport: {
      targets: targets()
       .pushIf(app.inDev, targets.pretty())
       .pushIf(app.inProduction, targets.file())
       .toArray()
    }
  }
}
```

## 使用多个日志器

AdonisJS 提供了一流的支持来配置多个日志器。日志器的唯一名称和配置在 `config/logger.ts` 文件中定义。

```ts
export default defineConfig({
  default: 'app',
  
  loggers: {
    // highlight-start
    app: {
      enabled: true,
      name: env.get('APP_NAME'),
      level: env.get('LOG_LEVEL', 'info')
    },
    payments: {
      enabled: true,
      name: 'payments',
      level: env.get('LOG_LEVEL', 'info')
    },
    // highlight-start
  }
})
```

配置完成后，你可以使用 `logger.use` 方法访问命名日志器。

```ts
import logger from '@adonisjs/core/services/logger'

logger.use('payments')
logger.use('app')

// 获取默认日志器的实例
logger.use()
```

## 依赖注入

当使用依赖注入时，你可以将 `Logger` 类作为依赖进行类型提示，IoC 容器将解析配置文件中定义的默认日志器实例。

如果该类是在 HTTP 请求期间构造的，那么容器将注入请求感知的 Logger 实例。

```ts
import { inject } from '@adonisjs/core'
import { Logger } from '@adonisjs/core/logger'

// highlight-start
@inject()
// highlight-end
class UserService {
  // highlight-start
  constructor(protected logger: Logger) {}
  // highlight-end

  async find(userId: string | number) {
    this.logger.info('Fetching user by id %s', userId)
    const user = await User.find(userId)
  }
}
```

## 日志记录方法

Logger API 几乎与 Pino 相同，除了 AdonisJS 的日志器不是事件发射器实例（而 Pino 是）。除此之外，日志记录方法的 API 与 Pino 相同。

```ts
import logger from '@adonisjs/core/services/logger'

logger.trace(config, 'using config')
logger.debug('user details: %o', { username: 'virk' })
logger.info('hello %s', 'world')
logger.warn('Unable to connect to database')
logger.error({ err: Error }, 'Something went wrong')
logger.fatal({ err: Error }, 'Something went wrong')
```

可以传递一个额外的合并对象作为第一个参数。然后，该对象的属性将被添加到输出的 JSON 中。

```ts
logger.info({ user: user }, 'Fetched user by id %s', user.id)
```

为了显示错误，你可以 [use the `err` key](https://getpino.io/#/docs/api?id=serializers-object) 来指定错误值。

```ts
logger.error({ err: error }, 'Unable to lookup user')
```

## 条件日志记录

日志器生成配置文件中配置的级别及以上级别的日志。例如，如果级别设置为 `warn`，则 `info`、`debug` 和 `trace` 级别的日志将被忽略。

如果计算日志消息的数据很耗时，你应该在计算数据之前检查给定的日志级别是否已启用。

```ts
import logger from '@adonisjs/core/services/logger'

if (logger.isLevelEnabled('debug')) {
  const data = await getLogData()
  logger.debug(data, 'Debug message')
}
```

你可以使用 `ifLevelEnabled` 方法表达相同的条件。该方法接受一个回调函数作为第二个参数，当指定的日志级别启用时，该回调函数将被执行。

```ts
logger.ifLevelEnabled('debug', async () => {
  const data = await getLogData()
  logger.debug(data, 'Debug message')
})
```

## 子日志器

子日志器是一个独立的实例，它从父日志器继承配置和绑定。

可以使用 `logger.child` 方法创建子日志器实例。该方法接受绑定作为第一个参数，可选的配置对象作为第二个参数。

```ts
import logger from '@adonisjs/core/services/logger'

const requestLogger = logger.child({ requestId: ctx.request.id() })
```

子日志器也可以在不同的日志级别下进行日志记录。

```ts
logger.child({}, { level: 'warn' })
```

## Pino 静态属性

`@adonisjs/core/logger` 模块导出了 [Pino static](https://getpino.io/#/docs/api?id=statics) 方法和属性。

```ts
import { 
  multistream,
  destination,
  transport,
  stdSerializers,
  stdTimeFunctions,
  symbols,
  pinoVersion
} from '@adonisjs/core/logger'
```

## 将日志写入文件

Pino 提供了 `pino/file` 目标，你可以使用它将日志写入文件。在目标选项中，你可以指定日志文件的目标路径。

```ts
app: {
  enabled: true,
  name: env.get('APP_NAME'),
  level: env.get('LOG_LEVEL', 'info')

  transport: {
    targets: targets()
      .push({
         transport: 'pino/file',
         level: 'info',
         options: {
           destination: '/var/log/apps/adonisjs.log'
         }
      })
      .toArray()
  }
}
```

### 文件轮转

Pino 没有内置支持文件轮转，因此，你要么使用系统级工具如 [logrotate](https://getpino.io/#/docs/help?id=rotate)，要么使用第三方包如 [pino-roll](https://github.com/feugy/pino-roll)。

```sh
npm i pino-roll
```

```ts
app: {
  enabled: true,
  name: env.get('APP_NAME'),
  level: env.get('LOG_LEVEL', 'info')

  transport: {
    targets: targets()
      // highlight-start
      .push({
        target: 'pino-roll',
        level: 'info',
        options: {
          file: '/var/log/apps/adonisjs.log',
          frequency: 'daily',
          mkdir: true
        }
      })
      // highlight-end
     .toArray()
  }
}
```

## 隐藏敏感值

日志可能成为泄露敏感数据的源头。因此，建议检查你的日志，并从输出中移除/隐藏敏感值。

在 Pino 中，你可以使用 `redact` 选项来隐藏/移除日志中的敏感键值对。在底层，使用了 [fast-redact](https://github.com/davidmarkclements/fast-redact) 包，你可以查阅其文档以查看可用的表达式。

```ts
// title: config/logger.ts
app: {
  enabled: true,
  name: env.get('APP_NAME'),
  level: env.get('LOG_LEVEL', 'info')

  // highlight-start
  redact: {
    paths: ['password', '*.password']
  }
  // highlight-end
}
```

```ts
import logger from '@adonisjs/core/services/logger'

const username = request.input('username')
const password = request.input('password')

logger.info({ username, password }, 'user signup')
// output: {"username":"virk","password":"[Redacted]","msg":"user signup"}
```

默认情况下，值会被 `[Redacted]` 占位符替换。你可以定义自定义占位符或移除键值对。

```ts
redact: {
  paths: ['password', '*.password'],
  censor: '[PRIVATE]'
}

// Remove property
redact: {
  paths: ['password', '*.password'],
  remove: true
}
```

### 使用 Secret 数据类型

另一种替代隐藏的方法是使用 Secret 类来包装敏感值。例如：

另请参阅：[Secret class usage docs](../references/helpers.md#secret)

```ts
import { Secret } from '@adonisjs/core/helpers'

const username = request.input('username')
// delete-start
const password = request.input('password')
// delete-end
// insert-start
const password = new Secret(request.input('password'))
// insert-end

logger.info({ username, password }, 'user signup')
// output: {"username":"virk","password":"[redacted]","msg":"user signup"}
```
