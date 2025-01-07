---
summary: 在 AdonisJS 应用程序中使用 AuthFinder mixin 验证用户凭据。
---

# 验证用户凭据

在 AdonisJS 应用程序中，验证用户凭据与认证层是解耦的。这确保了你可以继续使用认证守卫，而不限制验证用户凭据的选项。

默认情况下，我们提供安全的 API 来查找用户并验证他们的密码。但是，你也可以实现其他方式来验证用户，例如向他们的手机号码发送一次性密码 (OTP) 或使用双因素认证 (2FA)。

在本指南中，我们将介绍通过 UID 查找用户并验证其密码，然后将其标记为已登录的过程。

## 基本示例

你可以直接使用 User 模型来查找用户并验证其密码。在以下示例中，我们通过电子邮件查找用户，并使用 [hash](../security/hashing.md) 服务来验证密码哈希。

```ts
// highlight-start
import User from '#models/user'
import hash from '@adonisjs/core/services/hash'
// highlight-end
import type { HttpContext } from '@adonisjs/core/http'

export default class SessionController {
  async store({ request, response }: HttpContext) {
    const { email, password } = request.only(['email', 'password'])

    // highlight-start
    /**
     * 通过电子邮件查找用户。如果用户不存在，则返回错误。
     */
    const user = await User.findBy('email', email)

    if (!user) {
      return response.abort('Invalid credentials')
    }
    // highlight-end

    // highlight-start
    /**
     * 使用 hash 服务验证密码
     */
    const isPasswordValid = await hash.verify(user.password, password)

    if (!isPasswordValid) {
      return response.abort('Invalid credentials')
    }
    // highlight-end

    /**
     * 现在登录用户或为他们创建令牌
     */
    // ...
  }
}
```

:::caption{for="error"}

**上述方法的问题**

:::

<div class="card">

我们在上述示例中编写的代码容易受到 [时序攻击](https://en.wikipedia.org/wiki/Timing_attack) 的影响。在认证过程中，攻击者可以通过观察应用程序的响应时间来判断提供的凭据中电子邮件或密码是否正确。我们建议你使用 [AuthFinder mixin](#using-the-auth-finder-mixin) 来防止时间攻击，并获得更好的开发体验。

根据上述实现：

- 如果用户的电子邮件不正确，请求将花费较少时间。这是因为当我们找不到用户时，不会验证密码哈希。

- 如果电子邮件存在且密码不正确，请求将花费更多时间。这是因为密码哈希算法本质上较慢。

响应时间的差异足以让攻击者找到一个有效的电子邮件地址，并尝试不同的密码组合。

</div>

## 使用 Auth finder mixin

为了防止时间攻击，我们建议你在 User 模型上使用 [AuthFinder mixin](https://github.com/adonisjs/auth/blob/main/src/mixins/lucid.ts)。

Auth finder mixin 为应用的模型添加了 `findForAuth` 和 `verifyCredentials` 方法。`verifyCredentials` 方法提供了一个防止时间攻击的安全 API，用于查找用户并验证其密码。

你可以按如下方式导入并在模型上应用 mixin。

```ts
import { DateTime } from 'luxon'
import { compose } from '@adonisjs/core/helpers'
import { BaseModel, column } from '@adonisjs/lucid/orm'
// highlight-start
import hash from '@adonisjs/core/services/hash'
import { withAuthFinder } from '@adonisjs/auth/mixins/lucid'
// highlight-end

// highlight-start
const AuthFinder = withAuthFinder(() => hash.use('scrypt'), {
  uids: ['email'],
  passwordColumnName: 'password',
})
// highlight-end

// highlight-start
export default class User extends compose(BaseModel, AuthFinder) {
  // highlight-end
  @column({ isPrimary: true })
  declare id: number

  @column()
  declare fullName: string | null

  @column()
  declare email: string

  @column()
  declare password: string

  @column.dateTime({ autoCreate: true })
  declare createdAt: DateTime

  @column.dateTime({ autoCreate: true, autoUpdate: true })
  declare updatedAt: DateTime
}
```

- `withAuthFinder` 方法接受一个回调作为第一个参数，该回调返回一个散列器。在上述示例中，我们使用 `scrypt` 散列器。但是，你可以替换为其他散列器。

- 接下来，它接受一个配置对象，该对象具有以下属性：
  - `uids`：可以用于唯一标识用户的模型属性数组。如果你为用户分配了用户名或电话号码，也可以使用它们作为 UID。
  - `passwordColumnName`：保存用户密码的模型属性名称。

- 最后，你可以将 `withAuthFinder` 方法的返回值作为 User 模型的 [mixin](../references/helpers.md#compose) 使用。

### 验证凭据

一旦应用了 Auth finder mixin，你可以用以下代码片段替换 `SessionController.store` 方法中的所有代码。

```ts
import { HttpContext } from '@adonisjs/core/http'
import User from '#models/user'
// delete-start
import hash from '@adonisjs/core/services/hash'
// delete-end

export default class SessionController {
  // delete-start
  async store({ request, response }: HttpContext) {
  // delete-end
  // insert-start
  async store({ request }: HttpContext) {
  // insert-end
    const { email, password } = request.only(['email', 'password'])

    // delete-start
    /**
     * 通过电子邮件查找用户。如果用户不存在，则返回错误。
     */ 
    const user = await User.findBy('email', email)
    if (!user) {
      response.abort('Invalid credentials')
    }

    /**
     * 使用 hash 服务验证密码
     */
    await hash.verify(user.password, password)
    // delete-end
    // insert-start
    const user = await User.verifyCredentials(email, password)
    // insert-end

    /**
     * 现在登录用户或为他们创建令牌
     */
  }
}
```

### 处理异常

在凭据无效的情况下，`verifyCredentials` 方法将抛出 [E_INVALID_CREDENTIALS](../references/exceptions.md#e_invalid_credentials) 异常。

该异常是自动处理的，并将根据以下内容协商规则转换为响应：

- 带有 `Accept=application/json` 头的 HTTP 请求将收到一个错误消息数组。每个数组元素将是一个带有 message 属性的对象。

- 带有 `Accept=application/vnd.api+json` 头的 HTTP 请求将收到一个根据 JSON API 规范格式化的错误消息数组。

- 如果你使用会话，用户将被重定向到表单，并通过 [session flash messages](../basics/session.md#flash-messages) 接收错误。

- 所有其他请求将以纯文本形式接收错误。

但是，如果需要，你可以在 [全局异常处理](../basics/exception_handling.md) 中处理异常，如下所示。

```ts
// highlight-start
import { errors } from '@adonisjs/auth'
// highlight-end
import { HttpContext, ExceptionHandler } from '@adonisjs/core/http'

export default class HttpExceptionHandler extends ExceptionHandler {
  protected debug = !app.inProduction
  protected renderStatusPages = app.inProduction

  async handle(error: unknown, ctx: HttpContext) {
    // highlight-start
    if (error instanceof errors.E_INVALID_CREDENTIALS) {
      return ctx
        .response
        .status(error.status)
        .send(error.getResponseMessage(error, ctx))
    }
    // highlight-end

    return super.handle(error, ctx)
  }
}
```

## 散列用户密码

`AuthFinder` mixin 注册了一个 [beforeSave](https://github.com/adonisjs/auth/blob/main/src/mixins/lucid.ts#L40-L50) 钩子，以在 `INSERT` 和 `UPDATE` 调用期间自动散列用户密码。因此，你无需在模型中手动执行密码散列。
