---
summary: AdonisJS 应用中可用的 SQL 库和 ORM 选项。
---

# SQL 和 ORM

SQL 数据库因其能够将应用程序的数据存储在持久性存储中而广受欢迎。在 AdonisJS 应用中，你可以使用任何库和 ORM 来执行 SQL 查询。

:::note
AdonisJS 核心团队开发了 [Lucid ORM](./lucid.md)，但并不强制你使用它。你可以在 AdonisJS 应用中使用任何其他你喜欢的 SQL 库和 ORM。
:::

## 流行选项

以下是你可以在 AdonisJS 应用（就像任何其他 Node.js 应用一样）中使用的其他流行 SQL 库和 ORM 的列表。

- [**Lucid**](./lucid.md) 是由 AdonisJS 核心团队创建并维护的，基于 [Knex](https://knexjs.org) 的 SQL 查询构建器和 **Active Record ORM**。
- [**Prisma**](https://prisma.io/orm) Prisma ORM 是 Node.js 生态系统中另一个流行的 ORM。它拥有庞大的社区支持，提供直观的数据模型、自动化迁移、类型安全和自动补全。
- [**Kysely**](https://kysely.dev/docs/getting-started) 是 Node.js 的端到端类型安全的查询构建器。如果你需要一个没有模型的轻量级查询构建器，Kysely 是一个很好的选择。我们曾撰写过一篇文章，解释 [如何在 AdonisJS 应用中集成 Kysely](https://adonisjs.com/blog/kysely-with-adonisjs)。
- [**Drizzle ORM**](https://orm.drizzle.team/) 被我们社区中的许多 AdonisJS 开发者使用。我们没有使用过这个 ORM，但你可能想尝试一下，看看它是否适合你的用例。
- [**Mikro ORM**](https://mikro-orm.io/docs/guide/first-entity) 在 Node.js 生态系统中是一个被低估的 ORM。与 Lucid 相比，MikroORM 有些冗长。然而，它正在积极维护，并且也是基于 Knex 构建的。
- [**TypeORM**](https://typeorm.io) 在 TypeScript 生态系统中是一个流行的 ORM。

## 使用其他 SQL 库和 ORM

当使用其他 SQL 库或 ORM 时，你可能需要手动更改某些包的配置。

### 认证

[AdonisJS 认证模块](../authentication/introduction.md) 内置了对 Lucid 的支持，用于获取已认证用户。当使用其他 SQL 库或 ORM 时，你需要实现 `SessionUserProviderContract` 或 `AccessTokensProviderContract` 接口来获取用户。

以下是如何在使用 `Kysely` 时实现 `SessionUserProviderContract` 接口的示例。

```ts
import { symbols } from '@adonisjs/auth'
import type { SessionGuardUser, SessionUserProviderContract } from '@adonisjs/auth/types/session'
import type { Users } from '../../types/db.js' // 特定于 Kysely

export class SessionKyselyUserProvider implements SessionUserProviderContract<Users> {
  /**
   * 由事件触发器使用，为会话守卫发出的事件添加类型信息。
   */   
  declare [symbols.PROVIDER_REAL_USER]: Users

  /**
   * 会话守卫和你的提供者之间的桥梁。
   */
  async createUserForGuard(user: Users): Promise<SessionGuardUser<Users>> {
    return {
      getId() {
        return user.id
      },
      getOriginal() {
        return user
      },
    }
  }

  /**
   * 使用你的自定义 SQL 库或 ORM，通过用户 ID 查找用户。
   */
  async findById(identifier: number): Promise<SessionGuardUser<Users> | null> {
    const user = await db
      .selectFrom('users')
      .selectAll()
      .where('id', '=', identifier)
      .executeTakeFirst()

    if (!user) {
      return null
    }

    return this.createUserForGuard(user)
  }
}
```

一旦你实现了 `UserProvider` 接口，就可以在配置中使用它。

```ts
const authConfig = defineConfig({
  default: 'web',

  guards: {
    web: sessionGuard({
      useRememberMeTokens: false,
      provider: sessionUserProvider({
        model: () => import('#models/user'),
      }),
      
      provider: configProvider.create(async () => {
        const { SessionKyselyUserProvider } = await import(
          '../app/auth/session_user_provider.js' // 文件的路径
        )

        return new SessionKyselyUserProvider()
      }),
    }),
  },
})
```
