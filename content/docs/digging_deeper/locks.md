---
summary: 使用 `@adonisjs/lock` 包在你的 AdonisJS 应用程序中管理原子锁。
---

# 原子锁

原子锁，亦称 `mutex`，用于同步对共享资源的访问。换句话说，它防止多个进程或并发代码同时执行一段代码。

AdonisJS 团队创建了一个与框架无关的包，名为 [Verrou](https://github.com/Julien-R44/verrou)。`@adonisjs/lock` 包基于此包，**因此请务必也阅读 [Verrou 文档](https://verrou.dev/docs/introduction)，该文档更为详细。**

## 安装

使用以下命令安装并配置该包：

```sh
node ace add @adonisjs/lock
```

:::disclosure{title="查看 add 命令执行的步骤"}

1. 使用检测到的包管理器安装 `@adonisjs/lock` 包。

2. 在 `adonisrc.ts` 文件中注册以下服务提供者。
    ```ts
    {
      providers: [
        // ...其他提供者
        () => import('@adonisjs/lock/lock_provider')
      ]
    }
    ```

3. 创建 `config/lock.ts` 文件。

4. 在 `start/env.ts` 文件中定义以下环境变量及其验证。
   ```ts
   LOCK_STORE=redis
   ```

5. 如果使用 `database` 存储，可选择为 `locks` 表创建数据库迁移。

:::

## 配置

锁的配置存储在 `config/lock.ts` 文件中。

```ts
import env from '#start/env'
import { defineConfig, stores } from '@adonisjs/lock'

const lockConfig = defineConfig({
  default: env.get('LOCK_STORE'),
  stores: {
    redis: stores.redis({}),

    database: stores.database({
      tableName: 'locks',
    }),

    memory: stores.memory()
  },
})

export default lockConfig

declare module '@adonisjs/lock/types' {
  export interface LockStoresList extends InferLockStores<typeof lockConfig> {}
}
```

<dl>

<dt>

default

</dt>

<dd>

用于管理锁的`默认`存储。存储在同一配置文件中的 `stores` 对象下定义。

</dd>

<dt>

stores

</dt>

<dd>

你计划在应用程序中使用的存储集合。我们建议始终配置 `memory` 存储，以便在测试期间使用。

</dd>

</dl>

---

### 环境变量

默认锁存储通过 `LOCK_STORE` 环境变量定义，因此你可以在不同环境中切换不同的存储。例如，在测试期间使用 `memory` 存储，在开发和生产环境中使用 `redis` 存储。

此外，必须验证环境变量以允许预配置存储中的一个。验证在 `start/env.ts` 文件中使用 `Env.schema.enum` 规则定义。

```ts
{
  LOCK_STORE: Env.schema.enum(['redis', 'database', 'memory'] as const),
}
```

### Redis 存储

`redis` 存储对 `@adonisjs/redis` 包有对等依赖；因此，在使用 Redis 存储之前，你必须配置此包。

以下是 Redis 存储接受的选项列表：

```ts
{
  redis: stores.redis({
    connectionName: 'main',
  }),
}
```

<dl>
<dt>
connectionName
</dt>
<dd>

`connectionName` 属性指的是在 `config/redis.ts` 文件中定义的连接。

</dd>
</dl>

### 数据库存储

`database` 存储对 `@adonisjs/lucid` 包有对等依赖，因此，在使用数据库存储之前，你必须配置此包。

以下是数据库存储接受的选项列表：

```ts
{
  database: stores.database({
    connectionName: 'postgres',
    tableName: 'my_locks',
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

tableName

</dt>

<dd>

用于存储速率限制的数据库表。

</dd>

</dl>

### 内存存储

`memory` 存储是一个简单的内存存储，可用于测试目的，但不仅限于此。有时，对于某些用例，你可能希望有一个仅对当前进程有效的锁，而不是在多个进程之间共享。

内存存储基于 [`async-mutex`](https://www.npmjs.com/package/async-mutex) 包构建。

```ts
{
  memory: stores.memory(),
}
```

## 锁定资源

配置好锁存储后，你可以在应用程序中的任何地方开始使用锁来保护资源。

以下是一个如何使用锁来保护资源的简单示例。

:::codegroup

```ts
// title: 手动锁定
import { errors } from '@adonisjs/lock'
import locks from '@adonisjs/lock/services/main'
import { HttpContext } from '@adonisjs/core/http'

export default class OrderController {
  async process({ response, request }: HttpContext) {
    const orderId = request.input('order_id')

    /**
     * 尝试立即获取锁（不重试）
     */
    const lock = locks.createLock(`order.processing.${orderId}`)
    const acquired = await lock.acquireImmediately()
    if (!acquired) {
      return 'Order is already being processed'
    }

    /**
     * 已获取锁。我们可以处理订单
     */
    try {
      await processOrder()
      return 'Order processed successfully'
    } finally {
      /**
       * 始终使用 `finally` 块释放锁，以确保
       * 即使在处理过程中抛出异常，锁也会被释放。
       */
      await lock.release()
    }
  }
}
```

```ts
// title: 自动锁定
import { errors } from '@adonisjs/lock'
import locks from '@adonisjs/lock/services/main'
import { HttpContext } from '@adonisjs/core/http'

export default class OrderController {
  async process({ response, request }: HttpContext) {
    const orderId = request.input('order_id')

    /**
     * 仅在锁可用时运行函数
     * 函数执行后，锁也将自动释放
     */
    const [executed, result] = await locks
      .createLock(`order.processing.${orderId}`)
      .runImmediately(async (lock) => {
        /**
         * 已获取锁。我们可以处理订单
         */
        await processOrder()
        return 'Order processed successfully'
      })

    /**
     * 无法获取锁，函数未执行
     */
    if (!executed) return 'Order is already being processed'

    return result
  }
}
```

### 

这是一个如何在你的应用程序中使用锁的快速示例。

还有许多其他方法来管理锁，例如使用 `extend` 来延长锁的持续时间，使用 `getRemainingTime` 来获取锁到期前的剩余时间，配置锁的选项等等。

**为此，请务必阅读 [Verrou 文档](https://verrou.dev/docs/introduction) 以获取更多详细信息**。提醒一下，`@adonisjs/lock` 包是基于 `Verrou` 包的，因此你在 Verrou 文档中读到的所有内容也适用于 `@adonisjs/lock` 包。

## 使用另一个存储

如果你在 `config/lock.ts` 文件中定义了多个存储，可以使用 `use` 方法为特定锁使用不同的存储。

```ts
import locks from '@adonisjs/lock/services/main'

const lock = locks.use('redis').createLock('order.processing.1')
```

否则，如果只使用 `default` 存储，可以省略 `use` 方法。

```ts
import locks from '@adonisjs/lock/services/main'

const lock = locks.createLock('order.processing.1')
```

## 跨多个进程管理锁

有时，你可能希望一个进程创建并获取锁，而另一个进程释放锁。例如，你可能希望在 Web 请求中获取锁，并在后台作业中释放锁。这可以通过使用 `restoreLock` 方法实现。

```ts
// title: 你的主服务器
import locks from '@adonisjs/lock/services/main'

export class OrderController {
  async process({ response, request }: HttpContext) {
    const orderId = request.input('order_id')

    const lock = locks.createLock(`order.processing.${orderId}`)
    await lock.acquire()

    /**
     * 分派一个后台作业来处理订单。
     * 
     * 我们还将序列化的锁传递给作业，以便作业
     * 在订单处理完毕后可以释放锁。
     */
    queue.dispatch('app/jobs/process_order', {
      lock: lock.serialize()
    })
  }
}
```

```ts
// title: 你的后台作业
import locks from '@adonisjs/lock/services/main'

export class ProcessOrder {
  async handle({ lock }) {
    /**
     * 我们正在从序列化版本中恢复锁
     */
    const handle = locks.restoreLock(lock)

    /**
     * 处理订单
     */
    await processOrder()

    /**
     * 释放锁
     */
    await handle.release()
  }
}
```

## 测试

在测试期间，你可以使用 `memory` 存储来避免发出获取锁的实际网络请求。你可以通过在 `.env.testing` 文件中将 `LOCK_STORE` 环境变量设置为 `memory` 来实现这一点。

```env
// title: .env.test
LOCK_STORE=memory
```

## 创建自定义锁存储

首先，请务必查阅 [Verrou 文档](https://verrou.dev/docs/custom-lock-store)，其中深入介绍了如何创建自定义锁存储。在 AdonisJS 中，过程几乎相同。

让我们创建一个简单的 Noop 存储，它不执行任何操作。首先，我们必须创建一个实现 `LockStore` 接口的类。

```ts
import type { LockStore } from '@adonisjs/lock/types'

class NoopStore implements LockStore {
  /**
   * 在存储中保存锁。
   * 如果给定的键已锁定，该方法应返回 false
   *
   * @param key 要锁定的键
   * @param owner 锁的所有者
   * @param ttl 锁的存活时间（以毫秒为单位）。Null 表示无过期
   *
   * @returns 如果获取了锁，则返回 True，否则返回 False
   */
  async save(key: string, owner: string, ttl: number | null): Promise<boolean> {
    return false
  }

  /**
   * 如果锁由给定的所有者拥有，则从存储中删除锁
   * 否则应抛出 E_LOCK_NOT_OWNED 错误
   *
   * @param key 要删除的键
   * @param owner 所有者
   */
  async delete(key: string, owner: string): Promise<void> {
    return false
  }

  /**
   * 强制从存储中删除锁，而不检查所有者
   */
  async forceDelete(key: string): Promise<Void> {
    return false
  }

  /**
   * 检查锁是否存在。如果存在，则返回 true，否则返回 false
   */
  async exists(key: string): Promise<boolean> {
    return false
  }

  /**
   * 延长锁的过期时间。如果锁不由给定的所有者拥有，则抛出错误
   * 持续时间以毫秒为单位
   */
  async extend(key: string, owner: string, duration: number): Promise<void> {
    return false
  }
}
```

### 定义存储工厂

创建存储后，必须定义一个简单的工厂函数，`@adonisjs/lock` 将使用该函数来创建你存储的实例。

```ts
function noopStore(options: MyNoopStoreConfig) {
  return { driver: { factory: () => new NoopStore(options) } }
}
```

### 使用自定义存储

完成后，你可以按如下方式使用 `noopStore` 函数：

```ts
import { defineConfig } from '@adonisjs/lock'

const lockConfig = defineConfig({
  default: 'noop',
  stores: {
    noop: noopStore({}),
  },
})
```
