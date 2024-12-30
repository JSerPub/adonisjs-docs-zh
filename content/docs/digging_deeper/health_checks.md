---
summary: 了解如何在生产环境中监控您的应用程序并确保其顺利运行
---

# 健康检查

健康检查用于确保您的应用程序在生产环境中运行时处于健康状态。这可能包括监控服务器上的**可用磁盘空间**、**应用程序消耗的内存**或**测试数据库连接**。

AdonisJS 提供了一些内置的 [health checks](#available-health-checks)，以及创建和注册 [custom health checks](#creating-a-custom-health-check) 的能力。

## 配置健康检查

您可以通过执行以下命令在应用程序中配置健康检查。该命令将创建一个 `start/health.ts` 文件，并为**内存使用**和**已用磁盘空间**配置健康检查。您可以随意修改此文件，并添加或删除其他健康检查。

:::note

在使用以下命令之前，请确保已安装 `@adonisjs/core@6.12.1`。

:::

```sh
node ace configure health_checks
```

```ts
// title: start/health.ts
import { HealthChecks, DiskSpaceCheck, MemoryHeapCheck } from '@adonisjs/core/health'

export const healthChecks = new HealthChecks().register([
  new DiskSpaceCheck(),
  new MemoryHeapCheck(),
])
```

## 暴露端点

执行健康检查的一种常见方法是暴露一个 HTTP 端点，外部监控服务可以通过该端点来收集健康检查结果。

因此，让我们在 `start/routes.ts` 文件中定义一个路由，并将 `HealthChecksController` 绑定到该路由。`health_checks_controller.ts` 文件在初始设置期间创建，并位于 `app/controllers` 目录中。

```ts
// title: start/routes.ts
import router from '@adonisjs/core/services/router'
const HealthChecksController = () => import('#controllers/health_checks_controller')

router.get('/health', [HealthChecksController])
```

### 示例报告

`healthChecks.run` 方法将执行所有检查，并返回一个详细的 [report as a JSON object](https://github.com/adonisjs/health/blob/develop/src/types.ts#L36)。报告具有以下属性：

```json
{
  "isHealthy": true,
  "status": "warning",
  "finishedAt": "2024-06-20T07:09:35.275Z",
  "debugInfo": {
    "pid": 16250,
    "ppid": 16051,
    "platform": "darwin",
    "uptime": 16.271809083,
    "version": "v21.7.3"
  },
  "checks": [
    {
      "name": "Disk space check",
      "isCached": false,
      "message": "Disk usage is 76%, which is above the threshold of 75%",
      "status": "warning",
      "finishedAt": "2024-06-20T07:09:35.275Z",
      "meta": {
        "sizeInPercentage": {
          "used": 76,
          "failureThreshold": 80,
          "warningThreshold": 75
        }
      }
    },
    {
      "name": "Memory heap check",
      "isCached": false,
      "message": "Heap usage is under defined thresholds",
      "status": "ok",
      "finishedAt": "2024-06-20T07:09:35.265Z",
      "meta": {
        "memoryInBytes": {
          "used": 41821592,
          "failureThreshold": 314572800,
          "warningThreshold": 262144000
        }
      }
    }
  ]
}
```

<dl>

<dt>

isHealthy

</dt>

<dd>

一个布尔值，用于指示所有检查是否通过。如果有一个或多个检查失败，该值将设置为 `false`。

</dd>

<dt>

status

</dt>

<dd>

执行所有检查后的报告状态。它将是以下状态之一。

- `ok`：所有检查均成功通过。
- `warning`：一个或多个检查报告了警告。
- `error`：一个或多个检查失败。

</dd>

<dt>

finishedAt

</dt>

<dd>

测试完成的日期和时间。

</dd>

<dt>

checks

</dt>

<dd>

包含所有执行检查的详细报告的对象数组。

</dd>

<dt>

debugInfo

</dt>

<dd>

调试信息可用于识别进程及其运行时长。它包括以下属性。

- `pid`：进程 ID。
- `ppid`：管理您的 AdonisJS 应用程序进程的父进程 ID。
- `platform`：应用程序运行的平台。
- `uptime`：应用程序运行的时长（以秒为单位）。
- `version`：Node.js 版本。

</dd>

</dl>

### 保护端点

您可以使用身份验证中间件或创建自定义中间件来检查请求头中的特定 API 密钥，从而保护 `/health` 端点免受公共访问。例如：

```ts
import router from '@adonisjs/core/services/router'
const HealthChecksController = () => import('#controllers/health_checks_controller')

router
  .get('/health', [HealthChecksController])
  // insert-start
  .use(({ request, response }, next) => {
    if (request.header('x-monitoring-secret') === 'some_secret_value') {
      return next()
    }
    response.unauthorized({ message: 'Unauthorized access' })
  })
  // insert-end
```

## 可用的健康检查

以下是在 `start/health.ts` 文件中可以配置的可用健康检查列表。

### DiskSpaceCheck

`DiskSpaceCheck` 计算服务器上的已用磁盘空间，并在超过某个阈值时报告警告/错误。

```ts
import { HealthChecks, DiskSpaceCheck } from '@adonisjs/core/health'

export const healthChecks = new HealthChecks().register([
  new DiskSpaceCheck()
])
```

默认情况下，警告阈值设置为 75%，失败阈值设置为 80%。但是，您也可以定义自定义阈值。

```ts
export const healthChecks = new HealthChecks().register([
  new DiskSpaceCheck()
    // highlight-start
    .warnWhenExceeds(80) // 当使用超过 80% 时发出警告
    .failWhenExceeds(90), // 当使用超过 90% 时失败
  // highlight-end
])
```

### MemoryHeapCheck

`MemoryHeapCheck` 监控 [process.memoryUsage()](https://nodejs.org/api/process.html#processmemoryusage) 方法报告的堆大小，并在超过某个阈值时报告警告/错误。

```ts
import { HealthChecks, MemoryHeapCheck } from '@adonisjs/core/health'

export const healthChecks = new HealthChecks().register([
  new MemoryHeapCheck()
])
```

默认情况下，警告阈值设置为 **250MB**，故障阈值设置为 **300MB**。不过，您也可以定义自定义阈值。

```ts
export const healthChecks = new HealthChecks().register([
  new MemoryHeapCheck()
    // highlight-start
    .warnWhenExceeds('300 mb')
    .failWhenExceeds('700 mb'),
  // highlight-end
])
```

### MemoryRSSCheck

`MemoryRSSCheck` 监控 [process.memoryUsage()](https://nodejs.org/api/process.html#processmemoryusage) 方法报告的常驻集大小 (Resident Set Size)，并在超过某个阈值时报告警告/错误。

```ts
import { HealthChecks, MemoryRSSCheck } from '@adonisjs/core/health'

export const healthChecks = new HealthChecks().register([
  new MemoryRSSCheck()
])
```

默认情况下，警告阈值设置为 **320MB**，故障阈值设置为 **350MB**。不过，您也可以定义自定义阈值。

```ts
export const healthChecks = new HealthChecks().register([
  new MemoryRSSCheck()
    // highlight-start
    .warnWhenExceeds('600 mb')
    .failWhenExceeds('800 mb'),
  // highlight-end
])
```

### DbCheck

`DbCheck` 由 `@adonisjs/lucid` 包提供，用于监控与 SQL 数据库的连接。您可以按如下方式导入和使用它。

```ts
// insert-start
import db from '@adonisjs/lucid/services/db'
import { DbCheck } from '@adonisjs/lucid/database'
// insert-end
import { HealthChecks, DiskSpaceCheck, MemoryHeapCheck } from '@adonisjs/core/health'

export const healthChecks = new HealthChecks().register([
  new DiskSpaceCheck(),
  new MemoryHeapCheck(),
  // insert-start
  new DbCheck(db.connection()),
  // insert-end
])
```

以下是数据库健康检查的示例报告。

```json
// title: Report sample
{
  "name": "Database health check (postgres)",
  "isCached": false,
  "message": "Successfully connected to the database server",
  "status": "ok",
  "finishedAt": "2024-06-20T07:18:23.830Z",
  "meta": {
    "connection": {
      "name": "postgres",
      "dialect": "postgres"
    }
  }
}
```

`DbCheck` 类接受一个数据库连接以进行监控。如果您想监控多个连接，请为每个连接多次注册此检查。例如：

```ts
// title: Monitoring multiple connections
export const healthChecks = new HealthChecks().register([
  new DiskSpaceCheck(),
  new MemoryHeapCheck(),
  // insert-start
  new DbCheck(db.connection()),
  new DbCheck(db.connection('mysql')),
  new DbCheck(db.connection('pg')),
  // insert-end
])
```

### DbConnectionCountCheck

`DbConnectionCountCheck` 监控数据库服务器上的活动连接，并在超过某个阈值时报告警告/错误。此检查仅支持 **PostgreSQL** 和 **MySQL** 数据库。

```ts
import db from '@adonisjs/lucid/services/db'
// insert-start
import { DbCheck, DbConnectionCountCheck } from '@adonisjs/lucid/database'
// insert-end
import { HealthChecks, DiskSpaceCheck, MemoryHeapCheck } from '@adonisjs/core/health'

export const healthChecks = new HealthChecks().register([
  new DiskSpaceCheck(),
  new MemoryHeapCheck(),
  new DbCheck(db.connection()),
  // insert-start
  new DbConnectionCountCheck(db.connection())
  // insert-end
])
```

以下是数据库连接数健康检查的示例报告。

```json
// title: Report sample
{
  "name": "Connection count health check (postgres)",
  "isCached": false,
  "message": "There are 6 active connections, which is under the defined thresholds",
  "status": "ok",
  "finishedAt": "2024-06-20T07:30:15.840Z",
  "meta": {
    "connection": {
      "name": "postgres",
      "dialect": "postgres"
    },
    "connectionsCount": {
      "active": 6,
      "warningThreshold": 10,
      "failureThreshold": 15
    }
  }
}
```

默认情况下，警告阈值设置为 **10 个连接**，故障阈值设置为 **15 个连接**。不过，您也可以定义自定义阈值。

```ts
new DbConnectionCountCheck(db.connection())
  .warnWhenExceeds(4)
  .failWhenExceeds(10)
```

### RedisCheck

`RedisCheck` 由 `@adonisjs/redis` 包提供，用于监控与 Redis 数据库（包括集群）的连接。您可以按如下方式导入和使用它。

```ts
// insert-start
import redis from '@adonisjs/redis/services/main'
import { RedisCheck } from '@adonisjs/redis'
// insert-end
import { HealthChecks, DiskSpaceCheck, MemoryHeapCheck } from '@adonisjs/core/health'

export const healthChecks = new HealthChecks().register([
  new DiskSpaceCheck(),
  new MemoryHeapCheck(),
  // insert-start
  new RedisCheck(redis.connection()),
  // insert-end
])
```

以下是数据库健康检查的示例报告。

```json
// title: Report sample
{
  "name": "Redis health check (main)",
  "isCached": false,
  "message": "Successfully connected to the redis server",
  "status": "ok",
  "finishedAt": "2024-06-22T05:37:11.718Z",
  "meta": {
    "connection": {
      "name": "main",
      "status": "ready"
    }
  }
}
```

`RedisCheck` 类接受一个 Redis 连接以进行监控。如果您想监控多个连接，请为每个连接多次注册此检查。例如：

```ts
// title: Monitoring multiple connections
export const healthChecks = new HealthChecks().register([
  new DiskSpaceCheck(),
  new MemoryHeapCheck(),
  // insert-start
  new RedisCheck(redis.connection()),
  new RedisCheck(redis.connection('cache')),
  new RedisCheck(redis.connection('locks')),
  // insert-end
])
```

### RedisMemoryUsageCheck

`RedisMemoryUsageCheck` 监控 Redis 服务器的内存使用情况，并在超过某个阈值时报告警告/错误。

```ts
import redis from '@adonisjs/redis/services/main'
// insert-start
import { RedisCheck, RedisMemoryUsageCheck } from '@adonisjs/redis'
// insert-end
import { HealthChecks, DiskSpaceCheck, MemoryHeapCheck } from '@adonisjs/core/health'

export const healthChecks = new HealthChecks().register([
  new DiskSpaceCheck(),
  new MemoryHeapCheck(),
  new RedisCheck(redis.connection()),
  // insert-start
  new RedisMemoryUsageCheck(redis.connection())
  // insert-end
])
```

以下是来自 redis 内存使用情况健康检查的示例报告。

```json
// title: Report sample
{
  "name": "Redis memory consumption health check (main)",
  "isCached": false,
  "message": "Redis 内存使用量为 1.06MB，低于定义的阈值",
  "status": "ok",
  "finishedAt": "2024-06-22T05:36:32.524Z",
  "meta": {
    "connection": {
      "name": "main",
      "status": "ready"
    },
    "memoryInBytes": {
      "used": 1109616,
      "warningThreshold": 104857600,
      "failureThreshold": 125829120
    }
  }
}
```

默认情况下，警告阈值设置为 **100MB**，失败阈值设置为 **120MB**。但是，您也可以定义自定义阈值。

```ts
new RedisMemoryUsageCheck(db.connection())
  .warnWhenExceeds('200MB')
  .failWhenExceeds('240MB')
```

## 缓存结果

每次调用 `healthChecks.run` 方法（即访问 `/health` 端点）时，都会执行健康检查。您可能希望频繁地 ping `/health` 端点，但希望避免每次访问都执行某些检查。

例如，每分钟监控磁盘空间并不是很有用，但每分钟跟踪内存可能很有帮助。

因此，我们允许您在注册单个健康检查时缓存其结果。例如：

```ts
import {
  HealthChecks,
  MemoryRSSCheck,
  DiskSpaceCheck,
  MemoryHeapCheck,
} from '@adonisjs/core/health'

export const healthChecks = new HealthChecks().register([
  // highlight-start
  new DiskSpaceCheck().cacheFor('1 hour'),
  // highlight-end
  new MemoryHeapCheck(), // 不缓存
  new MemoryRSSCheck(), // 不缓存
])
```

## 创建自定义健康检查

您可以创建一个自定义健康检查，作为遵循 [HealthCheckContract](https://github.com/adonisjs/health/blob/develop/src/types.ts#L98) 接口的 JavaScript 类。您可以在项目或包的任何位置定义健康检查，并在 `start/health.ts` 文件中导入它以进行注册。

```ts
import { Result, BaseCheck } from '@adonisjs/core/health'
import type { HealthCheckResult } from '@adonisjs/core/types/health'

export class ExampleCheck extends BaseCheck {
  async run(): Promise<HealthCheckResult> {
    /**
     * 以下检查仅供参考
     */
    if (checkPassed) {
      return Result.ok('要显示的成功消息')
    }
    if (checkFailed) {
      return Result.failed('错误消息', errorIfAny)
    }
    if (hasWarning) {
      return Result.warning('警告消息')
    }
  }
}
```

如上述示例所示，您可以使用 [Result](https://github.com/adonisjs/health/blob/develop/src/result.ts) 类来创建健康检查结果。此外，您还可以按如下方式合并结果的元数据。

```ts
Result.ok('数据库连接正常').mergeMetaData({
  connection: {
    dialect: 'pg',
    activeCount: connections,
  },
})
```

### 注册自定义健康检查

您可以在 `start/health.ts` 文件中导入自定义健康检查类，并通过创建新的类实例来注册它。

```ts
// highlight-start
import { ExampleCheck } from '../app/health_checks/example.js'
// highlight-end

export const healthChecks = new HealthChecks().register([
  new DiskSpaceCheck().cacheFor('1 hour'),
  new MemoryHeapCheck(),
  new MemoryRSSCheck(),
  // highlight-start
  new ExampleCheck()
  // highlight-end
])
```
