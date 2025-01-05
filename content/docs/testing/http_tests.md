---
summary: 使用 Japa API 客户端插件在 AdonisJS 中进行 HTTP 测试。
---

# HTTP 测试

HTTP 测试指的是通过向应用程序端点发送实际的 HTTP 请求，并对响应体、头信息、Cookie、会话等进行断言，从而测试应用程序端点。

HTTP 测试是使用 Japa 的 [API 客户端插件](https://japa.dev/docs/plugins/api-client) 进行的。API 客户端插件是一个无状态的请求库，类似于 `Axios` 或 `fetch`，但更适合用于测试。

如果你想在真实的浏览器中测试你的 Web 应用程序，并以编程方式与它们交互，我们建议使用 [浏览器客户端](./browser_tests.md)，它使用 Playwright 进行测试。

## 设置

第一步是从 npm 包注册表中安装以下包。

:::codegroup

```sh
// title: npm
npm i -D @japa/api-client
```

:::

### 注册插件

在继续之前，请在 `tests/bootstrap.ts` 文件中注册插件。

```ts
// title: tests/bootstrap.ts
import { apiClient } from '@japa/api-client'

export const plugins: Config['plugins'] = [
  assert(),
  // highlight-start
  apiClient(),
  // highlight-end
  pluginAdonisJS(app),
]
```

`apiClient` 方法可以选择性地接受服务器的 `baseURL`。如果未提供，它将使用 `HOST` 和 `PORT` 环境变量。

```ts
import env from '#start/env'

export const plugins: Config['plugins'] = [
  apiClient({
    baseURL: `http://${env.get('HOST')}:${env.get('PORT')}`
  })
]
```

## 基本示例

注册 `apiClient` 插件后，你可以从 [TestContext](https://japa.dev/docs/test-context) 访问 `client` 对象以发起 HTTP 请求。

HTTP 测试必须写在为 `functional` 测试套件配置的文件夹中。你可以使用以下命令创建一个新的测试文件。

```sh
node ace make:test users/list --suite=functional
```

```ts
import { test } from '@japa/runner'

test.group('Users list', () => {
  test('get a list of users', async ({ client }) => {
    const response = await client.get('/users')

    response.assertStatus(200)
    response.assertBody({
      data: [
        {
          id: 1,
          email: 'foo@bar.com',
        }
      ]
    })
  })
})
```

要查看所有可用的请求和断言方法，请务必 [浏览 Japa 文档](https://japa.dev/docs/plugins/api-client)。

## OpenAPI 测试

断言和 API 客户端插件允许你使用 OpenAPI 规范文件来编写断言。你可以使用规范文件来测试 HTTP 响应的结构，而不是手动将响应与固定负载进行测试。

这是保持你的 OpenAPI 规范和服务器响应同步的好方法。因为如果你从规范文件中删除某个端点或更改响应数据结构，你的测试将会失败。

### 注册模式

AdonisJS 不提供从代码生成 OpenAPI 模式文件的工具。你可以手动编写，或使用图形化工具来创建它。

一旦你有了规范文件，请将其保存在 `resources` 目录中（如果该目录不存在，请创建它），并在 `tests/bootstrap.ts` 文件中使用 `assert` 插件进行注册。

```ts
// title: tests/bootstrap.ts
import app from '@adonisjs/core/services/app'

export const plugins: Config['plugins'] = [
  // highlight-start
  assert({
    openApi: {
      schemas: [app.makePath('resources/open_api_schema.yaml')]
    }
  }),
  // highlight-end
  apiClient(),
  pluginAdonisJS(app)
]
```

### 编写断言

注册模式后，你可以使用 `response.assertAgainstApiSpec` 方法来根据 API 规范进行断言。

```ts
test('paginate posts', async ({ client }) => {
  const response = await client.get('/posts')
  response.assertAgainstApiSpec()
})
```

- `response.assertAgainstApiSpec` 方法将使用 **请求方法**、**端点** 和 **响应状态码** 来查找预期的响应模式。
- 如果找不到响应模式，将抛出异常。否则，将根据模式验证响应体。

仅测试响应的结构，而不测试实际值。因此，你可能需要编写额外的断言。例如：

```ts
// 断言响应符合模式
response.assertAgainstApiSpec()

// 断言预期值
response.assertBodyContains({
  data: [{ title: 'Adonis 101' }, { title: 'Lucid 101' }]
})
```

## 读取/写入 Cookie

你可以使用 `withCookie` 方法在 HTTP 请求期间发送 Cookie。该方法接受 Cookie 名称作为第一个参数，值作为第二个参数。

```ts
await client
  .get('/users')
  .withCookie('user_preferences', { limit: 10 })
```

`withCookie` 方法定义了一个 [签名 Cookie](../basics/cookies.md#signed-cookies)。此外，你可以使用 `withEncryptedCookie` 或 `withPlainCookie` 方法向服务器发送其他类型的 Cookie。

```ts
await client
  .get('/users')
  .witEncryptedCookie('user_preferences', { limit: 10 })
```

```ts
await client
  .get('/users')
  .withPlainCookie('user_preferences', { limit: 10 })
```

### 从响应中读取 Cookie

你可以使用 `response.cookies` 方法访问 AdonisJS 服务器设置的 Cookie。该方法返回一个包含 Cookie 的键值对对象。

```ts
const response = await client.get('/users')
console.log(response.cookies())
```

你可以使用 `response.cookie` 方法通过名称访问单个 Cookie 的值。或者使用 `assertCookie` 方法来断言 Cookie 的值。

```ts
const response = await client.get('/users')

console.log(response.cookie('user_preferences'))

response.assertCookie('user_preferences')
```

## 填充会话存储

如果你在应用程序中使用 [`@adonisjs/session`](../basics/session.md) 包来读取/写入会话数据，那么在编写测试时，你可能还想使用 `sessionApiClient` 插件来填充会话存储。

### 设置

第一步是在 `tests/bootstrap.ts` 文件中注册插件。

```ts
// title: tests/bootstrap.ts
// insert-start
import { sessionApiClient } from '@adonisjs/session/plugins/api_client'
// insert-end

export const plugins: Config['plugins'] = [
  assert(),
  pluginAdonisJS(app),
  // insert-start
  sessionApiClient(app)
  // insert-end
]
```

然后，更新 `.env.test` 文件（如果该文件不存在，请创建它），并将 `SESSION_DRIVER` 设置为 `memory`。

```dotenv
// title: .env.test
SESSION_DRIVER=memory
```

### 使用会话数据发起请求

你可以在 Japa API 客户端上使用 `withSession` 方法，以一些预定义的会话数据发起 HTTP 请求。

`withSession` 方法将创建一个新的会话 ID，并用数据填充内存存储，你的 AdonisJS 应用程序代码可以像平常一样读取会话数据。

请求完成后，会话 ID 及其数据将被销毁。

```ts
测试（'使用购物车项目结账', async ({ client }) => {
  await client
    .post('/checkout')
    // highlight-start
    .withSession({
      cartItems: [
        {
          id: 1,
          name: '南印度过滤压榨咖啡'
        },
        {
          id: 2,
          name: '冷萃包',
        }
      ]
    })
    // highlight-end
})
```

像 `withSession` 方法一样，你可以在发起 HTTP 请求时使用 `withFlashMessages` 方法来设置闪存消息。

```ts
const response = await client
  .get('posts/1')
  .withFlashMessages({
    success: '帖子创建成功'
  })

response.assertTextIncludes(`帖子创建成功`)
```

### 从响应中读取会话数据

你可以使用 `response.session()` 方法访问由你的 AdonisJS 服务器设置的会话数据。该方法返回一个包含键值对的对象形式的会话数据。

```ts
const response = await client
  .post('/posts')
  .json({
    title: 'some title',
    body: 'some description',
  })

console.log(response.session()) // 所有会话数据
console.log(response.session('post_id')) // post_id 的值
```

你可以使用 `response.flashMessage` 或 `response.flashMessages` 方法从响应中读取闪存消息。

```ts
const response = await client.post('/posts')

response.flashMessages()
response.flashMessage('errors')
response.flashMessage('success')
```

最后，你可以使用以下方法之一为会话数据编写断言。

```ts
const response = await client.post('/posts')

/**
 * 断言会话存储中存在特定键（可选值）
 */
response.assertSession('cart_items')
response.assertSession('cart_items', [{
  id: 1,
}, {
  id: 2,
}])

/**
 * 断言会话存储中不存在特定键
 */
response.assertSessionMissing('cart_items')

/**
 * 断言闪存消息存储中存在闪存消息（可选值）
 */
response.assertFlashMessage('success')
response.assertFlashMessage('success', '帖子创建成功')

/**
 * 断言闪存消息存储中不存在特定键
 */
response.assertFlashMissing('errors')

/**
 * 断言闪存消息存储中存在验证错误
 */
response.assertHasValidationError('title')
response.assertValidationError('title', '请输入帖子标题')
response.assertValidationErrors('title', [
  '请输入帖子标题',
  '帖子标题必须为 10 个字符长。'
])
response.assertDoesNotHaveValidationError('title')
```

## 用户认证

如果你在应用程序中使用 `@adonisjs/auth` 包来认证用户，你可以使用 `authApiClient` Japa 插件在向应用程序发起 HTTP 请求时认证用户。

第一步是在 `tests/bootstrap.ts` 文件中注册插件。

```ts
// title: tests/bootstrap.ts
// insert-start
import { authApiClient } from '@adonisjs/auth/plugins/api_client'
// insert-end

export const plugins: Config['plugins'] = [
  assert(),
  pluginAdonisJS(app),
  // insert-start
  authApiClient(app)
  // insert-end
]
```

如果你使用的是基于会话的认证，请确保也设置了会话插件。请参阅[填充会话存储 - 设置](#setup-1)。

就这样。现在，你可以使用 `loginAs` 方法登录用户。该方法接受用户对象作为唯一参数，并将用户标记为当前 HTTP 请求的已登录用户。

```ts
import User from '#models/user'

test('获取支付列表', async ({ client }) => {
  const user = await User.create(payload)

  await client
    .get('/me/payments')
    // highlight-start
    .loginAs(user)
    // highlight-end
})
```

`loginAs` 方法使用 `config/auth.ts` 文件中配置的默认守卫进行认证。但是，你可以使用 `withGuard` 方法指定自定义守卫。例如：

```ts
await client
    .get('/me/payments')
    // highlight-start
    .withGuard('api_tokens')
    .loginAs(user)
    // highlight-end
```

## 带有 CSRF 令牌的请求

如果你的应用程序中的表单使用[CSRF 保护](../security/securing_ssr_applications.md)，你可以使用 `withCsrfToken` 方法生成 CSRF 令牌，并在请求期间将其作为头信息传递。

在使用 `withCsrfToken` 方法之前，请在 `tests/bootstrap.ts` 文件中注册以下 Japa 插件，并确保将[ `SESSION_DRIVER` 环境变量](#setup-1)切换为 `memory`。

```ts
// title: tests/bootstrap.ts
// insert-start
import { shieldApiClient } from '@adonisjs/shield/plugins/api_client'
import { sessionApiClient } from '@adonisjs/session/plugins/api_client'
// insert-end

export const plugins: Config['plugins'] = [
  assert(),
  pluginAdonisJS(app),
  // insert-start
  sessionApiClient(app),
  shieldApiClient()
  // insert-end
]
```

```ts
test('创建帖子', async ({ client }) => {
  await client
    .post('/posts')
    .form(dataGoesHere)
    .withCsrfToken()
})
```

## 路由助手

你可以使用 TestContext 中的 `route` 助手为路由创建 URL。使用路由助手可以确保每当你更新路由时，无需返回并修复测试中的所有 URL。

`route` 助手接受与全局模板方法[route](../basics/routing.md#url-builder)相同的一组参数。

```ts
test('获取用户列表', async ({ client, route }) => {
  const response = await client.get(
    // highlight-start
    route('users.list')
    // highlight-end
  )

  response.assertStatus(200)
  response.assertBody({
    data: [
      {
        id: 1,
        email: 'foo@bar.com',
      }
    ]
  })
})
```
