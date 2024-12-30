---
summary: 了解如何使用访问令牌守卫来通过访问令牌验证HTTP请求。
---

# 访问令牌守卫
在API上下文中，当服务器无法在最终用户设备上持久保存cookie时，访问令牌用于验证HTTP请求，例如，第三方访问API或移动应用的身份验证。

访问令牌可以生成为任何格式；例如，符合JWT标准的令牌称为JWT访问令牌，而专有格式的令牌称为不透明访问令牌。

AdonisJS使用结构化和存储方式如下的不透明访问令牌。

- 令牌由一个加密安全的随机值表示，后缀为CRC32校验和。
- 令牌值的哈希存储在数据库中。此哈希用于在身份验证时验证令牌。
- 最终的令牌值经过base64编码，并以`oat_`为前缀。前缀可以自定义。
- 前缀和CRC32校验和后缀有助于[秘密扫描工具](https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning)识别令牌，并防止它们在代码库中泄漏。

## 配置用户模型
在使用访问令牌守卫之前，您必须为用户模型设置令牌提供者。**令牌提供者用于创建、列出和验证访问令牌**。

auth包附带了一个数据库令牌提供者，该提供者将令牌持久化到SQL数据库中。您可以按如下方式配置它。

```ts
import { BaseModel } from '@adonisjs/lucid/orm'
// highlight-start
import { DbAccessTokensProvider } from '@adonisjs/auth/access_tokens'
// highlight-end

export default class User extends BaseModel {
  // ...模型属性的其余部分

  // highlight-start
  static accessTokens = DbAccessTokensProvider.forModel(User)
  // highlight-end
}
```

`DbAccessTokensProvider.forModel`接受用户模型作为第一个参数，接受一个选项对象作为第二个参数。

```ts
export default class User extends BaseModel {
  // ...模型属性的其余部分

  static accessTokens = DbAccessTokensProvider.forModel(User, {
    expiresIn: '30 days',
    prefix: 'oat_',
    table: 'auth_access_tokens',
    type: 'auth_token',
    tokenSecretLength: 40,
  })
}
```

<dl>

<dt>

expiresIn

</dt>

<dd>

令牌过期的时间长度。您可以传递以秒为单位的数值或作为字符串的[时间表达式](https://github.com/poppinss/utils?tab=readme-ov-file#secondsparseformat)。

默认情况下，令牌是长期有效的，不会过期。此外，您可以在生成令牌时指定其过期时间。

</dd>

<dt>

prefix

</dt>

<dd>

公开共享的令牌值的前缀。定义前缀有助于[秘密扫描工具](https://github.blog/2021-04-05-behind-githubs-new-authentication-token-formats/#identifiable-prefixes)识别令牌，并防止其在代码库中泄漏。

在发出令牌后更改前缀将使它们无效。因此，请谨慎选择前缀，并且不要频繁更改。

默认为`oat_`。

</dd>

<dt>

table

</dt>

<dd>

用于存储访问令牌的数据库表名。默认为`auth_access_tokens`。

</dd>

<dt>

type

</dt>

<dd>

用于标识一组令牌的唯一类型。如果您在单个应用程序中发出多种类型的令牌，则必须为它们全部定义唯一的类型。

默认为`auth_token`。

</dd>

<dt>

tokenSecretLength

</dt>

<dd>

随机令牌值的长度（以字符为单位）。默认为`40`。

</dd>

</dl>

---

一旦配置了令牌提供者，您就可以代表用户开始[发出令牌](#issuing-a-token)。发出令牌不需要设置身份验证守卫。守卫用于验证令牌。

## 创建访问令牌数据库表
在初始设置期间，我们为`auth_access_tokens`表创建迁移文件。迁移文件存储在`database/migrations`目录中。

您可以通过执行`migration:run`命令来创建数据库表。

```sh
node ace migration:run
```

但是，如果您出于某种原因手动配置auth包，则可以手动创建迁移文件，并将以下代码片段复制粘贴到其中。

```sh
node ace make:migration auth_access_tokens
```

```ts
import { BaseSchema } from '@adonisjs/lucid/schema'

export default class extends BaseSchema {
  protected tableName = 'auth_access_tokens'

  async up() {
    this.schema.createTable(this.tableName, (table) => {
      table.increments('id')
      table
        .integer('tokenable_id')
        .notNullable()
        .unsigned()
        .references('id')
        .inTable('users')
        .onDelete('CASCADE')

      table.string('type').notNullable()
      table.string('name').nullable()
      table.string('hash').notNullable()
      table.text('abilities').notNullable()
      table.timestamp('created_at')
      table.timestamp('updated_at')
      table.timestamp('last_used_at').nullable()
      table.timestamp('expires_at').nullable()
    })
  }

  async down() {
    this.schema.dropTable(this.tableName)
  }
}
```

## 发出令牌
根据您的应用程序，您可能会在登录时或登录后从应用程序仪表板发出令牌。在这两种情况下，发出令牌都需要一个用户对象（为其生成令牌），并且您可以直接使用`User`模型生成它们。

在以下示例中，我们使用`User.accessTokens.create`方法**通过ID查找用户**并**为其发出访问令牌**。当然，在实际应用程序中，此端点将受身份验证保护，但我们现在保持简单。

`.create`方法接受一个`User`模型实例，并返回一个[AccessToken](https://github.com/adonisjs/auth/blob/main/modules/access_tokens_guard/access_token.ts)类实例。

`token.value`属性包含值（包装为必须与用户共享的[Secret](../references/helpers.md#secret))）。该值仅在生成令牌时可用，用户之后将无法再次查看它。

```ts
import router from '@adonisjs/core/services/router'
import User from '#models/user'

router.post('users/:id/tokens', ({ params }) => {
  const user = await User.findOrFail(params.id)
  const token = await User.accessTokens.create(user)

  return {
    type: 'bearer',
    value: token.value!.release(),
  }
})
```

您还可以直接在响应中返回`token`，它将被序列化为以下JSON对象。

```ts
router.post('users/:id/tokens', ({ params }) => {
  const user = await User.findOrFail(params.id)
  const token = await User.accessTokens.create(user)

  // delete-start
  return {
    type: 'bearer',
    value: token.value!.release(),
  }
  // delete-end
  // insert-start
  return token
  // insert-end
})

/**
 * response: {
 *   type: 'bearer',
 *   value: 'oat_MTA.aWFQUmo2WkQzd3M5cW0zeG5JeHdiaV9rOFQzUWM1aTZSR2xJaDZXYzM5MDE4MzA3NTU',
 *   expiresAt: null,
 * }
 */
```

### 定义权限
根据您正在构建的应用程序，您可能希望限制访问令牌仅执行特定任务。例如，发出一个令牌，允许读取和列出项目，但不允许创建或删除它们。

在以下示例中，我们将权限数组作为第二个参数定义。权限被序列化为JSON字符串并持久化到数据库中。

对于auth包，权限没有实际意义。在执行给定操作之前，检查令牌权限是由您的应用程序负责的。

```ts
await User.accessTokens.create(user, ['server:create', 'server:read'])
```

### 令牌能力与 Bouncer 能力

不应将令牌能力（token abilities）与 [bouncer 授权检查](../security/authorization.md#defining-abilities) 混淆。让我们通过一个实际例子来理解两者的区别。

- 假设你定义了一个 **允许管理员用户创建新项目的 bouncer 能力**。

- 同一个管理员用户为自己创建了一个令牌，但为了防止令牌滥用，他们将令牌能力限制为 **读取项目**。

- 现在，在你的应用程序中，你需要实现访问控制，允许管理员用户创建新项目，同时禁止令牌创建新项目。

你可以为此用例编写如下的 bouncer 能力。

:::note

`user.currentAccessToken` 指的是当前 HTTP 请求中用于身份验证的访问令牌。你可以在 [请求身份验证](#the-current-access-token) 部分了解更多信息。

:::

```ts
import { AccessToken } from '@adonisjs/auth/access_tokens'
import { Bouncer } from '@adonisjs/bouncer'

export const createProject = Bouncer.ability(
  (user: User & { currentAccessToken?: AccessToken }) => {
    /**
     * 如果没有 "currentAccessToken" 属性，则意味着
     * 用户在没有访问令牌的情况下进行了身份验证
     */
    if (!user.currentAccessToken) {
      return user.isAdmin
    }

    /**
     * 否则，检查用户是否为管理员以及他们用于
     * 身份验证的令牌是否允许 "project:create" 能力。
     */
    return user.isAdmin && user.currentAccessToken.allows('project:create')
  }
)
```

### 令牌过期

默认情况下，令牌是长期有效的，且永不过期。但是，你可以在 [配置令牌提供者](#configuring-the-user-model) 时或生成令牌时定义过期时间。

过期时间可以定义为表示秒数的数值或基于字符串的时间表达式。

```ts
await User.accessTokens.create(
  user, // 为用户
  ['*'], // 具有所有能力
  {
    expiresIn: '30 days' // 30 天后过期
  }
)
```

### 命名令牌

默认情况下，令牌没有名称。但是，你可以在生成令牌时为其分配一个名称。例如，如果你的应用程序允许用户自行生成令牌，你可以要求他们指定一个可识别的名称。

```ts
await User.accessTokens.create(
  user,
  ['*'],
  {
    name: request.input('token_name'),
    expiresIn: '30 days'
  }
)
```

## 配置守卫

现在我们可以发放令牌了，接下来让我们配置一个身份验证守卫来验证请求并认证用户。守卫必须在 `config/auth.ts` 文件中的 `guards` 对象下进行配置。

```ts
// title: config/auth.ts
import { defineConfig } from '@adonisjs/auth'
// highlight-start
import { tokensGuard, tokensUserProvider } from '@adonisjs/auth/access_tokens'
// highlight-end

const authConfig = defineConfig({
  default: 'api',
  guards: {
    // highlight-start
    api: tokensGuard({
      provider: tokensUserProvider({
        tokens: 'accessTokens',
        model: () => import('#models/user'),
      })
    }),
    // highlight-end
  },
})

export default authConfig
```

`tokensGuard` 方法创建 [AccessTokensGuard](https://github.com/adonisjs/auth/blob/main/modules/access_tokens_guard/guard.ts) 类的实例。它接受一个用户提供者，用于验证令牌和查找用户。

`tokensUserProvider` 方法接受以下选项，并返回 [AccessTokensLucidUserProvider](https://github.com/adonisjs/auth/blob/main/modules/access_tokens_guard/user_providers/lucid.ts) 类的实例。

- `model`：用于查找用户的 Lucid 模型。
- `tokens`：模型中引用令牌提供者的静态属性名称。

## 请求身份验证

配置好守卫后，你可以开始使用 `auth` 中间件或手动调用 `auth.authenticate` 方法来验证请求。

`auth.authenticate` 方法返回已认证用户的 User 模型实例，或者在无法认证请求时抛出 [E_UNAUTHORIZED_ACCESS](../references/exceptions.md#e_unauthorized_access) 异常。

```ts
import router from '@adonisjs/core/services/router'

router.post('projects', async ({ auth }) => {
  // 使用默认守卫进行身份验证
  const user = await auth.authenticate()

  // 使用命名守卫进行身份验证
  const user = await auth.authenticateUsing(['api'])
})
```

### 使用 auth 中间件

你可以使用 `auth` 中间件来验证请求或抛出异常，而无需手动调用 `authenticate` 方法。

auth 中间件接受一个守卫数组，用于验证请求。一旦其中一个提到的守卫验证了请求，身份验证过程就会停止。

```ts
import router from '@adonisjs/core/services/router'
import { middleware } from '#start/kernel'

router
  .post('projects', async ({ auth }) => {
    console.log(auth.user) // User
    console.log(auth.authenticatedViaGuard) // 'api'
    console.log(auth.user!.currentAccessToken) // AccessToken
  })
  .use(middleware.auth({
    guards: ['api']
  }))
```

### 检查请求是否已认证

你可以使用 `auth.isAuthenticated` 标志来检查请求是否已认证。对于已认证的请求，`auth.user` 的值总是已定义。

```ts
import { HttpContext } from '@adonisjs/core/http'

class PostsController {
  async store({ auth }: HttpContext) {
    if (auth.isAuthenticated) {
      await auth.user!.related('posts').create(postData)
    }
  }
}
```

### 获取已认证用户或失败

如果你不喜欢在 `auth.user` 属性上使用 [非空断言操作符](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#non-null-assertion-operator-postfix-)，你可以使用 `auth.getUserOrFail` 方法。此方法将返回用户对象或抛出 [E_UNAUTHORIZED_ACCESS](../references/exceptions.md#e_unauthorized_access) 异常。

```ts
import { HttpContext } from '@adonisjs/core/http'

class PostsController {
  async store({ auth }: HttpContext) {
    const user = auth.getUserOrFail()
    await user.related('posts').create(postData)
  }
}
```

## 当前访问令牌

访问令牌守卫在成功验证请求后，会在用户对象上定义 `currentAccessToken` 属性。`currentAccessToken` 属性是 [AccessToken](https://github.com/adonisjs/auth/blob/main/modules/access_tokens_guard/access_token.ts) 类的实例。

你可以使用 `currentAccessToken` 对象来获取令牌的能力或检查令牌的过期时间。此外，在身份验证期间，守卫会更新 `last_used_at` 列以反映当前时间戳。

如果你在代码库的其他部分将 User 模型与 `currentAccessToken` 作为类型引用，你可能希望在模型本身上声明此属性。

:::caption{for="error"}

**不要合并 `currentAccessToken`**

:::

```ts
import { AccessToken } from '@adonisjs/auth/access_tokens'

Bouncer.ability((
  user: User & { currentAccessToken?: AccessToken }
) => {
})
```

:::caption{for="success"}

**在模型上声明为属性**

:::

```ts
import { AccessToken } from '@adonisjs/auth/access_tokens'

export default class User extends BaseModel {
  currentAccessToken?: AccessToken
}
```

```ts
Bouncer.ability((user: User) => {
})
```

## 列出所有令牌
您可以使用令牌提供者通过 `accessTokens.all` 方法获取所有令牌的列表。返回值将是 `AccessToken` 类实例的数组。

```ts
router
  .get('/tokens', async ({ auth }) => {
    return User.accessTokens.all(auth.user!)
  })
  .use(
    middleware.auth({
      guards: ['api'],
    })
  )
```

`all` 方法还会返回已过期的令牌。在呈现列表之前，您可能希望对其进行过滤，或者在令牌旁边显示 **“Token expired”**（令牌已过期）消息。例如：

```edge
@each(token in tokens)
  <h2> {{ token.name }} </h2>
  @if(token.isExpired())
    <p> Expired </p>
  @end

  <p> Abilities: {{ token.abilities.join(',') }} </p>
@end
```

## 删除令牌
您可以使用 `accessTokens.delete` 方法删除令牌。该方法接受用户作为第一个参数，令牌 ID 作为第二个参数。

```ts
await User.accessTokens.delete(user, token.identifier)
```

## 事件

请查阅[事件参考指南](../references/events.md#access_tokens_authauthentication_attempted)，以查看访问令牌守卫发出的可用事件列表。
