---
summary: Lucid ORM 简介，这是一个基于 Knex 构建的 SQL 查询构建器和 Active Record ORM。
---

# Lucid ORM

Lucid 是一个 SQL 查询构建器，同时也是一个基于 [Knex](https://knexjs.org) 构建的 Active Record ORM，由 AdonisJS 核心团队创建和维护。Lucid 致力于充分发挥 SQL 的潜力，并为许多高级 SQL 操作提供简洁的 API。

:::note
Lucid 的文档可在 [https://lucid.adonisjs.com](https://lucid.adonisjs.com) 查看。
:::

## 为什么选择 Lucid

以下是 Lucid 的一些精选功能。

- 基于 Knex 的流畅查询构建器。
- 支持读写分离和多个连接管理。
- 遵循 active record 模式的基于类的模型（处理关系、序列化和钩子）。
- 使用增量更改集修改数据库模式的迁移系统。
- 模型工厂，用于生成测试用的假数据。
- 数据库填充器，用于向数据库插入初始/虚拟数据。

除此之外，以下是在 AdonisJS 应用中使用 Lucid 的其他原因。

- 我们为 Lucid 提供了与 Auth 包和验证器的顶级集成。因此，你无需自行编写这些集成。

- Lucid 预配置在 `api` 和 `web` 初学者工具包中，为你的应用程序提供良好开端。

- Lucid 的主要目标之一是充分发挥 SQL 的潜力，并支持许多高级 SQL 操作，如 **窗口函数**、**递归 CTE**、**JSON 操作**、**基于行的锁**等。

- Lucid 和 Knex 已经存在多年。因此，与许多其他新的 ORM 相比，它们更加成熟且经过实战考验。

话虽如此，AdonisJS 并不强制你使用 Lucid。你可以卸载该包并安装你选择的 ORM。

## 安装

使用以下命令安装并配置 Lucid。

```sh
node ace add @adonisjs/lucid
```

:::disclosure{title="查看配置命令执行的步骤"}

1. 在 `adonisrc.ts` 文件中注册以下服务提供者。

   ```ts
   {
     providers: [
       // ...其他提供者
       () => import('@adonisjs/lucid/database_provider'),
     ]
   }
   ```

2. 在 `adonisrc.ts` 文件中注册以下命令。

   ```ts
   {
     commands: [
       // ...其他命令
       () => import('@adonisjs/lucid/commands'),
     ]
   }
   ```

3. 创建 `config/database.ts` 文件。

4. 为所选方言定义环境变量及其验证。

5. 安装所需的同等依赖项。

:::

## 创建你的第一个模型

配置完成后，你可以使用以下命令创建第一个模型。

```sh
node ace make:model User
```

此命令将在 `app/models` 目录中创建一个新文件，内容如下。

```ts
import { DateTime } from 'luxon'
import { BaseModel, column } from '@adonisjs/lucid/orm'

export default class User extends BaseModel {
  @column({ isPrimary: true })
  declare id: number

  @column.dateTime({ autoCreate: true })
  declare createdAt: DateTime

  @column.dateTime({ autoCreate: true, autoUpdate: true })
  declare updatedAt: DateTime
}
```

有关模型的更多信息，请访问 [官方文档](https://lucid.adonisjs.com/docs/models)。

## 迁移

迁移是一种使用增量更改集修改数据库模式和数据的方法。你可以使用以下命令创建新的迁移。

```sh
node ace make:migration users
```

此命令将在 `database/migrations` 目录中创建一个新文件，内容如下。

```ts
import { BaseSchema } from '@adonisjs/lucid/schema'

export default class extends BaseSchema {
  protected tableName = 'users'

  async up() {
    this.schema.createTable(this.tableName, (table) => {
      table.increments('id')
      table.timestamp('created_at')
      table.timestamp('updated_at')
    })
  }

  async down() {
    this.schema.dropTable(this.tableName)
  }
}
```

你可以使用以下命令运行所有待处理的迁移。

```sh
node ace migration:run
```

有关迁移的更多信息，请访问 [官方文档](https://lucid.adonisjs.com/docs/migrations)。

## 查询构建器

Lucid 附带了一个基于 Knex 构建的流畅查询构建器。你可以使用查询构建器对数据库执行 CRUD 操作。

```ts
import db from '@adonisjs/lucid/services/db'

/**
 * 创建查询构建器实例
 */
const query = db.query()

/**
 * 创建查询构建器实例并选择表
 */
const queryWithTableSelection = db.from('users')
```

查询构建器也可以限定于模型实例。

```ts
import User from '#models/user'

const user = await User.query().where('username', 'rlanz').first()
```

有关查询构建器的更多信息，请访问 [官方文档](https://lucid.adonisjs.com/docs/select-query-builder)。

## CRUD 操作

Lucid 模型具有内置方法，可以对数据库执行 CRUD 操作。

```ts
import User from '#models/user'

/**
 * 创建新用户
 */
const user = await User.create({
  username: 'rlanz',
  email: 'romain@adonisjs.com',
})

/**
 * 按主键查找用户
 */
const user = await User.find(1)

/**
 * 更新用户
 */

const user = await User.find(1)
user.username = 'romain'
await user.save()

/**
 * 删除用户
 */
const user = await User.find(1)
await user.delete()
```

有关 CRUD 操作的更多信息，请访问 [官方文档](https://lucid.adonisjs.com/docs/crud-operations)。

## 了解更多

- [Lucid 文档](https://lucid.adonisjs.com)
- [安装与使用](https://lucid.adonisjs.com/docs/installation)
- [CRUD 操作](https://lucid.adonisjs.com/docs/crud-operations)
- [模型钩子](https://lucid.adonisjs.com/docs/model-hooks)
- [关系](https://lucid.adonisjs.com/docs/relationships)
- [Adocasts Lucid 系列](https://adocasts.com/topics/lucid)