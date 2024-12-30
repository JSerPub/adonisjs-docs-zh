---
summary: 了解 AdonisJS 中的认证系统以及如何在您的应用程序中对用户进行身份验证。
---

# Authentication（认证）

AdonisJS 提供了一个强大且安全的认证系统，您可以使用它来登录并验证应用程序的用户。无论是服务器端渲染的应用程序、SPA 客户端还是移动应用，您都可以为它们设置认证。

认证包是围绕 **guards（守卫）** 和 **providers（提供者）** 构建的。

- Guards 是特定登录类型的端到端实现。例如，`session` 守卫允许您使用 cookie 和会话来验证用户身份。同时，`access_tokens` 守卫将允许您使用令牌来验证客户端。

- Providers 用于从数据库中查找用户和令牌。您可以使用内置的提供者，也可以实现自己的提供者。

:::note

为了确保您应用程序的安全性，我们会对用户密码和令牌进行适当的哈希处理。此外，AdonisJS 的安全原语受到 [timing attacks](https://en.wikipedia.org/wiki/Timing_attack) 和 [session fixation attacks](https://owasp.org/www-community/attacks/Session_fixation) 的保护。

:::

## Auth 包不支持的功能

auth 包专注于对 HTTP 请求进行认证，以下功能不在其范围内：

- 用户注册功能，如 **注册表单**、**电子邮件验证** 和 **帐户激活**。
- 帐户管理功能，如 **密码恢复** 或 **电子邮件更新**。
- 分配角色或验证权限。相反，[use bouncer](../security/authorization.md) 来在您的应用程序中实现授权检查。

<!-- :::note

**寻找一个功能齐全的用户管理系统？**\

查看 persona。Persona 是一个官方包和启动套件，带有一个功能齐全的用户管理系统。

它提供了用户注册、电子邮件管理、会话跟踪、资料管理以及 2FA 的即用型操作。

::: -->

## 选择认证守卫

以下内置认证守卫为您提供了最直接的工作流程，用于在不牺牲应用程序安全性的情况下对用户进行身份验证。此外，您可以 [build your authentication guards](./custom_auth_guard.md) 根据自定义需求构建您的认证守卫。

### Session（会话）

会话守卫使用 [@adonisjs/session](../basics/session.md) 包来跟踪会话存储中已登录用户的状态。

会话和 cookie 在互联网上已经存在很长时间，并且适用于大多数应用程序。我们建议使用会话守卫：

- 如果您正在创建一个服务器端渲染的 Web 应用程序。
- 或者，一个与其客户端位于同一顶级域名的 AdonisJS API。例如，`api.example.com` 和 `example.com`。

### Access tokens（访问令牌）

访问令牌是登录成功后颁发给用户的加密安全随机令牌（也称为不透明访问令牌）。您可以在 AdonisJS 服务器无法写入/读取 cookie 的应用程序中使用访问令牌。例如：

- 一个原生移动应用。
- 一个托管在与 AdonisJS API 服务器不同域名的 Web 应用程序。

当使用访问令牌时，客户端应用程序有责任安全地存储它们。访问令牌代表用户提供对应用程序的无限制访问，泄露它们可能导致安全问题。

### Basic auth（基本认证）

基本认证守卫是 [HTTP authentication framework](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication) 的一种实现，其中客户端必须通过 `Authorization` 头传递以 base64 编码的字符串形式的用户凭据。

有比基本认证更好的方法来实现安全的登录系统。然而，在您的应用程序处于积极开发阶段时，您可以暂时使用它。

## 选择用户提供者

如本指南前面所述，用户提供者在认证过程中负责查找用户。

用户提供者是守卫特定的；例如，会话守卫的用户提供者负责通过用户 ID 查找用户，而访问令牌守卫的用户提供者则负责验证访问令牌。

我们为内置守卫提供了一个 Lucid 用户提供者，它使用 Lucid 模型来查找用户、生成令牌和验证令牌。

<!-- 如果您不使用 Lucid，则必须 [implement a custom user provider]()。 -->

## 安装

认证系统预配置在 `web` 和 `api` 启动套件中。然而，您可以按照以下步骤在应用程序中手动安装和配置它。

```sh
# 使用会话守卫（默认）进行配置
node ace add @adonisjs/auth --guard=session

# 使用访问令牌守卫进行配置
node ace add @adonisjs/auth --guard=access_tokens

# 使用基本认证守卫进行配置
node ace add @adonisjs/auth --guard=basic_auth
```

:::disclosure{title="查看 add 命令执行的步骤"}

1. 使用检测到的包管理器安装 `@adonisjs/auth` 包。

2. 在 `adonisrc.ts` 文件中注册以下服务提供者。

    ```ts
    {
      providers: [
        // ...其他提供者
        () => import('@adonisjs/auth/auth_provider')
      ]
    }
    ```

3. 在 `start/kernel.ts` 文件中创建并注册以下中间件。

    ```ts
    router.use([
      () => import('@adonisjs/auth/initialize_auth_middleware')
    ])
    ```

    ```ts
    router.named({
      auth: () => import('#middleware/auth_middleware'),
      // 仅在使用会话守卫时使用
      guest: () => import('#middleware/guest_middleware')
    })
    ```

4. 在 `app/models` 目录中创建用户模型。
5. 为 `users` 表创建数据库迁移。
6. 为所选守卫创建数据库迁移。
:::

## 初始化认证中间件

在设置过程中，我们会在您的应用程序中注册 `@adonisjs/auth/initialize_auth_middleware`。该中间件负责创建 [Authenticator](https://github.com/adonisjs/auth/blob/main/src/authenticator.ts) 类的实例，并通过 `ctx.auth` 属性与请求的其余部分共享它。

请注意，初始化认证中间件不会对请求进行认证或保护路由。它仅用于初始化认证器并与请求的其余部分共享。您必须使用 [auth](./session_guard.md#protecting-routes) 中间件来保护路由。

此外，如果您的应用程序使用 Edge，则相同的认证器实例会与 Edge 模板共享，您可以使用 `auth` 属性访问它。例如：

```edge
@if(auth.isAuthenticated)
  <p> Hello {{ auth.user.email }} </p>
@end
```

## 创建 users 表

`configure` 命令会在 `database/migrations` 目录中为 `users` 表创建一个数据库迁移。您可以随意打开此文件并根据应用程序需求进行修改。

默认情况下，会创建以下列：

```ts
import { BaseSchema } from '@adonisjs/lucid/schema'

export default class extends BaseSchema {
  protected tableName = 'users'

  async up() {
    this.schema.createTable(this.tableName, (table) => {
      table.increments('id').notNullable()
      table.string('full_name').nullable()
      table.string('email', 254).notNullable().unique()
      table.string('password').notNullable()

      table.timestamp('created_at').notNullable()
      table.timestamp('updated_at').nullable()
    })
  }

  async down() {
    this.schema.dropTable(this.tableName)
  }
}
```

此外，如果您在 `users` 表中定义、重命名或删除列，请更新 `User` 模型。

## 后续步骤

- 了解如何在不牺牲应用程序安全性的情况下 [verify user credentials](./verifying_user_credentials.md)。
- 使用 [session guard](./session_guard.md) 进行有状态认证。
- 使用 [access tokens guard](./access_tokens_guard.md) 进行基于令牌的认证。