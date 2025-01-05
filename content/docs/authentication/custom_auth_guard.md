---
summary: 学习如何为 AdonisJS 创建自定义身份验证守卫。
---

# 创建自定义身份验证守卫

auth 包使你能够为内置守卫无法满足的用例创建自定义身份验证守卫。在本指南中，我们将创建一个使用 JWT 令牌进行身份验证的守卫。

身份验证守卫围绕以下概念展开。

- **用户提供者**：守卫应与用户无关。它们不应将查询和查找数据库中用户的函数硬编码。相反，守卫应依赖于用户提供者，并将其实现作为构造函数依赖项接受。

- **守卫实现**：守卫实现必须遵循 `GuardContract` 接口。该接口描述了将守卫与 Auth 层的其他部分集成所需的 API。

## 创建 `UserProvider` 接口

守卫负责定义 `UserProvider` 接口及其应包含的方法/属性。例如，[Session 守卫](https://github.com/adonisjs/auth/blob/develop/modules/session_guard/types.ts#L153-L166) 所接受的 UserProvider 比 [访问令牌守卫](https://github.com/adonisjs/auth/blob/develop/modules/access_tokens_guard/types.ts#L192-L222) 所接受的 UserProvider 要简单得多。

因此，无需创建满足每个守卫实现的 UserProvider。每个守卫都可以规定它们所接受的 UserProvider 的要求。

在本例中，我们需要一个提供者使用 `user ID` 在数据库中查找用户。我们不关心使用哪个数据库或如何执行查询。这是实现 UserProvider 的开发人员的责任。

:::note

本指南中的所有代码最初都可以存储在 `app/auth/guards` 目录中的单个文件中。

:::

```ts
// title: app/auth/guards/jwt.ts
import { symbols } from '@adonisjs/auth'

/**
 * 用户提供者和守卫之间的桥梁
 */
export type JwtGuardUser<RealUser> = {
  /**
   * 返回用户的唯一 ID
   */
  getId(): string | number | BigInt

  /**
   * 返回原始用户对象
   */
  getOriginal(): RealUser
}

/**
 * JWT 守卫所接受的 UserProvider 接口。
 */
export interface JwtUserProviderContract<RealUser> {
  /**
   * 守卫实现可以使用此属性来推断实际用户（即 RealUser）的数据类型
   */
  [symbols.PROVIDER_REAL_USER]: RealUser

  /**
   * 创建一个用户对象，作为守卫和真实用户值之间的适配器。
   */
  createUserForGuard(user: RealUser): Promise<JwtGuardUser<RealUser>>

  /**
   * 通过用户 ID 查找用户。
   */
  findById(identifier: string | number | BigInt): Promise<JwtGuardUser<RealUser> | null>
}
```

在上面的示例中，`JwtUserProviderContract` 接口接受一个名为 `RealUser` 的泛型用户属性。由于此接口不知道实际用户（我们从数据库中获取的用户）的样子，因此它将其作为泛型接受。例如：

- 使用 Lucid 模型的实现将返回 Model 的实例。因此，`RealUser` 的值将是该实例。

- 使用 Prisma 的实现将返回具有特定属性的用户对象；因此，`RealUser` 的值将是该对象。

总之，`JwtUserProviderContract` 将用户的数据类型留给 UserProvider 实现来决定。

### 理解 `JwtGuardUser` 类型
`JwtGuardUser` 类型作为用户提供者和守卫之间的桥梁。守卫使用 `getId` 方法获取用户的唯一 ID，使用 `getOriginal` 方法在请求身份验证后获取用户的对象。

## 实现守卫
让我们创建 `JwtGuard` 类，并定义 [`GuardContract`](https://github.com/adonisjs/auth/blob/main/src/types.ts#L30) 接口所需的方法/属性。最初，此文件中会有很多错误，但这没关系；随着我们的进展，所有错误都会消失。

:::note

请花一些时间阅读以下示例中每个属性/方法旁边的注释。

:::

```ts
import { symbols } from '@adonisjs/auth'
import { AuthClientResponse, GuardContract } from '@adonisjs/auth/types'

export class JwtGuard<UserProvider extends JwtUserProviderContract<unknown>>
  implements GuardContract<UserProvider[typeof symbols.PROVIDER_REAL_USER]>
{
  /**
   * 守卫发出的事件及其类型的列表。
   */
  declare [symbols.GUARD_KNOWN_EVENTS]: {}

  /**
   * 守卫驱动程序的唯一名称
   */
  driverName: 'jwt' = 'jwt'

  /**
   * 标志，用于指示当前 HTTP 请求期间是否尝试了身份验证
   */
  authenticationAttempted: boolean = false

  /**
   * 布尔值，用于指示当前请求是否已进行身份验证
   */
  isAuthenticated: boolean = false

  /**
   * 当前已身份验证用户的引用
   */
  user?: UserProvider[typeof symbols.PROVIDER_REAL_USER]

  /**
   * 为给定用户生成 JWT 令牌。
   */
  async generate(user: UserProvider[typeof symbols.PROVIDER_REAL_USER]) {
  }

  /**
   * 对当前 HTTP 请求进行身份验证，并返回用户实例（如果存在有效的 JWT 令牌），否则抛出异常
   */
  async authenticate(): Promise<UserProvider[typeof symbols.PROVIDER_REAL_USER]> {
  }

  /**
   * 与 authenticate 相同，但不抛出异常
   */
  async check(): Promise<boolean> {
  }

  /**
   * 返回已身份验证的用户或抛出错误
   */
  getUserOrFail(): UserProvider[typeof symbols.PROVIDER_REAL_USER] {
  }

  /**
   * 当使用 "loginAs" 方法登录用户时，Japa 在测试期间会调用此方法。
   */
  async authenticateAsClient(
    user: UserProvider[typeof symbols.PROVIDER_REAL_USER]
  ): Promise<AuthClientResponse> {
  }
}
```

## 接受用户提供者
守卫必须接受一个用户提供者，以便在身份验证期间查找用户。你可以将其作为构造函数参数接受，并存储一个私有引用。

```ts
export class JwtGuard<UserProvider extends JwtUserProviderContract<unknown>>
  implements GuardContract<UserProvider[typeof symbols.PROVIDER_REAL_USER]>
{
  // insert-start
  #userProvider: UserProvider

  constructor(
    userProvider: UserProvider
  ) {
    this.#userProvider = userProvider
  }
  // insert-end
}
```

## 生成令牌
让我们实现 `generate` 方法，并为给定用户创建令牌。我们将从 npm 安装并使用 `jsonwebtoken` 包来生成令牌。

```sh
npm i jsonwebtoken @types/jsonwebtoken
```

此外，我们必须使用**密钥**来签署令牌，因此让我们更新 `constructor` 方法，并通过选项对象将密钥作为选项接受。

```ts
// insert-start
import jwt from 'jsonwebtoken'

export type JwtGuardOptions = {
  secret: string
}
// insert-end

export class JwtGuard<UserProvider extends JwtUserProviderContract<unknown>>
  implements GuardContract<UserProvider[typeof symbols.PROVIDER_REAL_USER]>
{
  #userProvider: UserProvider
  // insert-start
  #options: JwtGuardOptions
  // insert-end

  constructor(
    userProvider: UserProvider
    // insert-start
    options: JwtGuardOptions
    // insert-end
  ) {
    this.#userProvider = userProvider
    // insert-start
    this.#options = options
    // insert-end
  }

  /**
   * 为给定用户生成一个 JWT 令牌。
   */
  async generate(
    user: UserProvider[typeof symbols.PROVIDER_REAL_USER]
  ) {
    // insert-start
    const providerUser = await this.#userProvider.createUserForGuard(user)
    const token = jwt.sign({ userId: providerUser.getId() }, this.#options.secret)

    return {
      type: 'bearer',
      token: token
    }
    // insert-end
  }
}
```

- 首先，我们使用 `userProvider.createUserForGuard` 方法创建提供者用户（即真实用户和守卫之间的桥梁）的实例。

- 然后，我们使用 `jwt.sign` 方法创建一个包含 `userId` 的签名令牌，并将其返回。

## 认证请求

认证请求包括：

- 从请求头或 cookie 中读取 JWT 令牌。
- 验证其真实性。
- 获取令牌所对应用户。

我们的守卫需要访问 [HttpContext](../concepts/http_context.md) 以读取请求头和 cookie，因此让我们更新 `constructor` 类并接受它作为参数。

```ts
// insert-start
import type { HttpContext } from '@adonisjs/core/http'
// insert-end

export class JwtGuard<UserProvider extends JwtUserProviderContract<unknown>>
  implements GuardContract<UserProvider[typeof symbols.PROVIDER_REAL_USER]>
{
  // insert-start
  #ctx: HttpContext
  // insert-end
  #userProvider: UserProvider
  #options: JwtGuardOptions

  constructor(
    // insert-start
    ctx: HttpContext,
    // insert-end
    userProvider: UserProvider,
    options: JwtGuardOptions
  ) {
    // insert-start
    this.#ctx = ctx
    // insert-end
    this.#userProvider = userProvider
    this.#options = options
  }
}
```

在本例中，我们将从 `authorization` 头中读取令牌。不过，你可以调整实现以支持 cookie。

```ts
import {
  symbols,
  // insert-start
  errors
  // insert-end
} from '@adonisjs/auth'

export class JwtGuard<UserProvider extends JwtUserProviderContract<unknown>>
  implements GuardContract<UserProvider[typeof symbols.PROVIDER_REAL_USER]>
{
  /**
   * 认证当前 HTTP 请求，如果存在有效的 JWT 令牌，
   * 则返回用户实例，否则抛出异常
   */
  async authenticate(): Promise<UserProvider[typeof symbols.PROVIDER_REAL_USER]> {
    /**
     * 如果已对给定请求进行过认证，则避免重新认证
     */
    if (this.authenticationAttempted) {
      return this.getUserOrFail()
    }
    this.authenticationAttempted = true

    /**
     * 确保存在授权头
     */
    const authHeader = this.#ctx.request.header('authorization')
    if (!authHeader) {
      throw new errors.E_UNAUTHORIZED_ACCESS('未经授权的访问', {
        guardDriverName: this.driverName,
      })
    }

    /**
     * 拆分头值并从中读取令牌
     */
    const [, token] = authHeader.split('Bearer ')
    if (!token) {
      throw new errors.E_UNAUTHORIZED_ACCESS('未经授权的访问', {
        guardDriverName: this.driverName,
      })
    }

    /**
     * 验证令牌
     */
    const payload = jwt.verify(token, this.#options.secret)
    if (typeof payload !== 'object' || !('userId' in payload)) {
      throw new errors.E_UNAUTHORIZED_ACCESS('未经授权的访问', {
        guardDriverName: this.driverName,
      })
    }

    /**
     * 通过用户 ID 获取用户并保存对其的引用
     */
    const providerUser = await this.#userProvider.findById(payload.userId)
    if (!providerUser) {
      throw new errors.E_UNAUTHORIZED_ACCESS('未经授权的访问', {
        guardDriverName: this.driverName,
      })
    }

    this.user = providerUser.getOriginal()
    return this.getUserOrFail()
  }
}
```

## 实现 `check` 方法
`check` 方法是 `authenticate` 方法的静默版本，你可以按如下方式实现它。

```ts
export class JwtGuard<UserProvider extends JwtUserProviderContract<unknown>>
  implements GuardContract<UserProvider[typeof symbols.PROVIDER_REAL_USER]>
{
  /**
   * 与 authenticate 相同，但不抛出异常
   */
  async check(): Promise<boolean> {
    // insert-start
    try {
      await this.authenticate()
      return true
    } catch {
      return false
    }
    // insert-end
  }
}
```

## 实现 `getUserOrFail` 方法
最后，让我们实现 `getUserOrFail` 方法。它应返回用户实例（如果用户不存在则抛出错误）。

```ts
export class JwtGuard<UserProvider extends JwtUserProviderContract<unknown>>
  implements GuardContract<UserProvider[typeof symbols.PROVIDER_REAL_USER]>
{
  /**
   * 返回已认证的用户或抛出错误
   */
  getUserOrFail(): UserProvider[typeof symbols.PROVIDER_REAL_USER] {
    // insert-start
    if (!this.user) {
      throw new errors.E_UNAUTHORIZED_ACCESS('未经授权的访问', {
        guardDriverName: this.driverName,
      })
    }

    return this.user
    // insert-end
  }
}
```

## 实现 `authenticateAsClient` 方法
`authenticateAsClient` 方法在测试期间使用，当你希望通过 [`loginAs` 方法](../testing/http_tests.md#authenticating-users) 在测试中登录用户时。对于 JWT 实现，该方法应返回包含 JWT 令牌的 `authorization` 头。

```ts
export class JwtGuard<UserProvider extends JwtUserProviderContract<unknown>>
  implements GuardContract<UserProvider[typeof symbols.PROVIDER_REAL_USER]>
{
  /**
   * 当使用 "loginAs" 方法登录用户时，
   * Japa 在测试期间会调用此方法。
   */
  async authenticateAsClient(
    user: UserProvider[typeof symbols.PROVIDER_REAL_USER]
  ): Promise<AuthClientResponse> {
    // insert-start
    const token = await this.generate(user)
    return {
      headers: {
        authorization: `Bearer ${token.token}`,
      },
    }
    // insert-end
  }
}
```

## 使用守卫
让我们转到 `config/auth.ts` 并在 `guards` 列表中注册守卫。

```ts
import { defineConfig } from '@adonisjs/auth'
// insert-start
import { sessionUserProvider } from '@adonisjs/auth/session'
import env from '#start/env'
import { JwtGuard } from '../app/auth/jwt/guard.js'
// insert-end

// insert-start
const jwtConfig = {
  secret: env.get('APP_KEY'),
}
const userProvider = sessionUserProvider({
  model: () => import('#models/user'),
})
// insert-end

const authConfig = defineConfig({
  default: 'jwt',
  guards: {
    // insert-start
    jwt: (ctx) => {
      return new JwtGuard(ctx, userProvider, jwtConfig)
    },
    // insert-end
  },
})

export default authConfig
```

如你所见，我们在 `JwtGuard` 实现中使用了 `sessionUserProvider`。这是因为 `JwtUserProviderContract` 接口与 Session guard 创建的用户提供者（User Provider）兼容。

因此，我们无需创建自己的用户提供者实现，而是重用了 Session guard 中的用户提供者。

## 最终示例

实现完成后，你可以像使用其他内置 guard 一样使用 `jwt` guard。以下是如何生成和验证 JWT 令牌的示例。

```ts
import User from '#models/user'
import router from '@adonisjs/core/services/router'
import { middleware } from './kernel.js'

router.post('login', async ({ request, auth }) => {
  const { email, password } = request.all()
  const user = await User.verifyCredentials(email, password)

  return await auth.use('jwt').generate(user)
})

router
  .get('/', async ({ auth }) => {
    return auth.getUserOrFail()
  })
  .use(middleware.auth())
```
