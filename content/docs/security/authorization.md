---
summary: 学习如何在 AdonisJS 应用程序中使用 `@adonisjs/bouncer` 包编写授权检查。
---

# 授权

您可以使用 `@adonisjs/bouncer` 包在 AdonisJS 应用程序中编写授权检查。Bouncer 提供了一个基于 JavaScript 的 API，用于将授权检查编写为 **能力** 和 **策略**。

能力和策略的目标是将授权操作的逻辑抽象到一个地方，并在代码库的其余部分中重用。

- [能力](#defining-abilities) 被定义为函数，如果您的应用程序有较少且简单的授权检查，它们可能非常适合。

- [策略](#defining-policies) 被定义为类，您必须为应用程序中的每个资源创建一个策略。策略还可以从 [自动依赖注入](#dependency-injection) 中受益。

:::note

Bouncer 不是 RBAC 或 ACL 的实现。相反，它提供了一个具有细粒度控制的低级 API，用于在 AdonisJS 应用程序中授权操作。

:::

## 安装

使用以下命令安装并配置该包：

```sh
node ace add @adonisjs/bouncer
```

:::disclosure{title="查看 add 命令执行的步骤"}

1. 使用检测到的包管理器安装 `@adonisjs/bouncer` 包。

2. 在 `adonisrc.ts` 文件中注册以下服务提供者和命令。

    ```ts
    {
      commands: [
        // ...其他命令
        () => import('@adonisjs/bouncer/commands')
      ],
      providers: [
        // ...其他提供者
        () => import('@adonisjs/bouncer/bouncer_provider')
      ]
    }
    ```

3. 创建 `app/abilities/main.ts` 文件来定义和导出能力。

4. 创建 `app/policies/main.ts` 文件来导出所有策略作为集合。

5. 在 `middleware` 目录中创建 `initialize_bouncer_middleware`。

6. 在 `start/kernel.ts` 文件中注册以下中间件。

    ```ts
    router.use([
      () => import('#middleware/initialize_bouncer_middleware')
    ])
    ```

:::

:::tip

**您是视觉学习者吗？** - 查看我们朋友在 Adocasts 上的 [AdonisJS Bouncer ](https://adocasts.com/series/adonisjs-bouncer) 免费屏幕录像系列。

:::

## 初始化 Bouncer 中间件

在设置期间，我们在应用程序中创建并注册了 `#middleware/initialize_bouncer_middleware` 中间件。初始化中间件负责为当前已认证用户创建一个 [Bouncer](https://github.com/adonisjs/bouncer/blob/main/src/bouncer.ts) 类的实例，并通过 `ctx.bouncer` 属性与请求的其余部分共享它。

此外，我们还使用 `ctx.view.share` 方法将相同的 Bouncer 实例与 Edge 模板共享。如果您在应用程序中不使用 Edge，可以删除中间件中的以下代码行。

:::note

您拥有应用程序的源代码，包括初始设置期间创建的文件。因此，不要犹豫去修改它们，使其与您的应用程序环境一起工作。

:::

```ts
async handle(ctx: HttpContext, next: NextFn) {
  ctx.bouncer = new Bouncer(
    () => ctx.auth.user || null,
    abilities,
    policies
  ).setContainerResolver(ctx.containerResolver)

  // delete-start
  /**
   * 如果不使用 Edge，请删除
   */
  if ('view' in ctx) {
    ctx.view.share(ctx.bouncer.edgeHelpers)
  }
  // delete-end

  return next()
}
```

## 定义能力

能力通常是 JavaScript 函数，写在 `./app/abilities/main.ts` 文件中。您可以从该文件中导出多个能力。

在以下示例中，我们使用 `Bouncer.ability` 方法定义了一个名为 `editPost` 的能力。实现回调必须返回 `true` 以授权用户，并返回 `false` 以拒绝访问。

:::note

能力应始终接受 `User` 作为第一个参数，后跟授权检查所需的其他参数。

:::

```ts
// title: app/abilities/main.ts
import User from '#models/user'
import Post from '#models/post'
import { Bouncer } from '@adonisjs/bouncer'

export const editPost = Bouncer.ability((user: User, post: Post) => {
  return user.id === post.userId
})
```

### 执行授权

定义能力后，您可以使用 `ctx.bouncer.allows` 方法执行授权检查。

Bouncer 会自动将当前已登录用户作为第一个参数传递给能力回调，您必须手动提供其余参数。

```ts
import Post from '#models/post'
// highlight-start
import { editPost } from '#abilities/main'
// highlight-end
import router from '@adonisjs/core/services/router'

router.put('posts/:id', async ({ bouncer, params, response }) => {
  /**
   * 通过 ID 查找帖子，以便我们可以对其执行授权检查。
   */
  const post = await Post.findOrFail(params.id)

  /**
   * 使用能力来查看已登录用户是否被允许执行该操作。
   */
  // highlight-start
  if (await bouncer.allows(editPost, post)) {
    return 'You can edit the post'
  }
  // highlight-end

  return response.forbidden('You cannot edit the post')
})
```

`bouncer.allows` 方法的对立面是 `bouncer.denies` 方法。您可能更喜欢这个方法，而不是编写一个 `if not` 语句。

```ts
if (await bouncer.denies(editPost, post)) {
  response.abort('Your cannot edit the post', 403)
}
```

### 允许访客用户

默认情况下，Bouncer 会拒绝未登录用户的授权检查，而不会调用能力回调。

但是，您可能希望定义某些能力，使其可以与访客用户一起工作。例如，允许访客查看已发布的帖子，但允许帖子的创建者查看草稿。

您可以使用 `allowGuest` 选项定义一个允许访客用户的能力。在这种情况下，选项将作为第一个参数定义，而回调将作为第二个参数。

```ts
export const viewPost = Bouncer.ability(
  // highlight-start
  { allowGuest: true },
  // highlight-end
  (user: User | null, post: Post) => {
    /**
     * 允许每个人访问已发布的帖子
     */
    if (post.isPublished) {
      return true
    }

    /**
     * 访客不能查看未发布的帖子
     */
    if (!user) {
      return false
    }

    /**
     * 帖子的创建者也可以查看未发布的帖子。
     */
    return user.id === post.userId
  }
)
```

### 授权除已登录用户以外的其他用户

如果您想授权除已登录用户以外的其他用户，可以使用 `Bouncer` 构造函数为给定用户创建一个新的 Bouncer 实例。

```ts
import User from '#models/user'
import { Bouncer } from '@adonisjs/bouncer'

const user = await User.findOrFail(1)
// highlight-start
const bouncer = new Bouncer(user)
// highlight-end

if (await bouncer.allows(editPost, post)) {
}
```

## 定义策略

策略提供了一个抽象层，将授权检查组织为类。建议为每个资源创建一个策略。例如，如果您的应用程序有一个 Post 模型，您必须创建一个 `PostPolicy` 类来授权创建或更新帖子等操作。

策略存储在 `./app/policies` 目录中，每个文件表示一个策略。您可以通过运行以下命令创建一个新策略。

另请参阅：[Make policy command](../references/commands.md#makepolicy)

```sh
node ace make:policy post
```

策略类继承自 [BasePolicy](https://github.com/adonisjs/bouncer/blob/main/src/base_policy.ts) 类，您可以为要执行的授权检查实现方法。在以下示例中，我们定义了授权检查以 `create`、`edit` 和 `delete` 一个帖子。

```ts
// title: app/policies/post_policy.ts
import User from '#models/user'
import Post from '#models/post'
import { BasePolicy } from '@adonisjs/bouncer'
import { AuthorizerResponse } from '@adonisjs/bouncer/types'

export default class PostPolicy extends BasePolicy {
  /**
   * 每个已登录用户都可以创建帖子
   */
  create(user: User): AuthorizerResponse {
    return true
  }

  /**
   * 只有帖子的创建者才能编辑帖子
   */
  edit(user: User, post: Post): AuthorizerResponse {
    return user.id === post.userId
  }

  /**
   * 只有帖子的创建者才能删除帖子
   */
  delete(user: User, post: Post): AuthorizerResponse {
    return user.id === post.userId
  }
}
```

### 执行授权

创建策略后，您可以使用 `bouncer.with` 方法指定要用于授权的策略，然后使用 `bouncer.allows` 或 `bouncer.denies` 方法链执行授权检查。

:::note

在 `bouncer.with` 方法之后链式调用的 `allows` 和 `denies` 方法是类型安全的，并且会根据您在策略类中定义的方法显示一个完成列表。

:::

```ts
import Post from '#models/post'
import PostPolicy from '#policies/post_policy'
import type { HttpContext } from '@adonisjs/core/http'

export default class PostsController {
  async create({ bouncer, response }: HttpContext) {
    // highlight-start
    if (await bouncer.with(PostPolicy).denies('create')) {
      return response.forbidden('Cannot create a post')
    }
    // highlight-end

    // 继续执行控制器逻辑
  }

  async edit({ bouncer, params, response }: HttpContext) {
    const post = await Post.findOrFail(params.id)

    // highlight-start
    if (await bouncer.with(PostPolicy).denies('edit', post)) {
      return response.forbidden('Cannot edit the post')
    }
    // highlight-end

    // 继续执行控制器逻辑
  }

  async delete({ bouncer, params, response }: HttpContext) {
    const post = await Post.findOrFail(params.id)

    // highlight-start
    if (await bouncer.with(PostPolicy).denies('delete', post)) {
      return response.forbidden('Cannot delete the post')
    }
    // highlight-end

    // 继续执行控制器逻辑
  }
}
```

### 允许访客用户

[与能力类似](#allowing-guest-users)，策略也可以使用 `@allowGuest` 装饰器为访客用户定义授权检查。例如：

```ts
import User from '#models/user'
import Post from '#models/post'
import { BasePolicy, allowGuest } from '@adonisjs/bouncer'
import type { AuthorizerResponse } from '@adonisjs/bouncer/types'

export default class PostPolicy extends BasePolicy {
  @allowGuest()
  view(user: User | null, post: Post): AuthorizerResponse {
    /**
     * 允许每个人访问已发布的帖子
     */
    if (post.isPublished) {
      return true
    }

    /**
     * 访客不能查看未发布的帖子
     */
    if (!user) {
      return false
    }

    /**
     * 帖子的创建者也可以查看未发布的帖子。
     */
    return user.id === post.userId
  }
}
```

### 策略钩子

您可以在策略类上定义 `before` 和 `after` 模板方法，以在授权检查前后执行操作。一个常见的用例是始终允许或拒绝某个用户的访问。

:::note

无论是否已登录用户，`before` 和 `after` 方法都会被调用。因此，请确保处理 `user` 值为 `null` 的情况。

:::

`before` 的响应解释如下：

- `true` 值将被视为成功的授权，并且不会调用操作方法。
- `false` 值将被视为访问被拒绝，并且不会调用操作方法。
- 如果返回值为 `undefined`，Bouncer 将执行操作方法以执行授权检查。

```ts
// title: app/policies/post_policy.ts
import User from '#models/user'
import Post from '#models/post'
import { BasePolicy } from '@adonisjs/bouncer'
import { AuthorizerResponse } from '@adonisjs/bouncer/types'

export default class PostPolicy extends BasePolicy {
  /**
   * 每个已登录用户都可以创建帖子
   */
  create(user: User): AuthorizerResponse {
    return true
  }

  /**
   * 只有帖子的创建者才能编辑帖子
   */
  edit(user: User, post: Post): AuthorizerResponse {
    return user.id === post.userId
  }

  /**
   * 只有帖子的创建者才能删除帖子
   */
  delete(user: User, post: Post): AuthorizerResponse {
    return user.id === post.userId
  }
}
```

### 执行授权

创建策略后，您可以使用 `bouncer.with` 方法指定要用于授权的策略，然后使用 `bouncer.allows` 或 `bouncer.denies` 方法链执行授权检查。

:::note

在 `bouncer.with` 方法之后链式调用的 `allows` 和 `denies` 方法是类型安全的，并且会根据您在策略类中定义的方法显示一个完成列表。

:::

```ts
import Post from '#models/post'
import PostPolicy from '#policies/post_policy'
import type { HttpContext } from '@adonisjs/core/http'

export default class PostsController {
  async create({ bouncer, response }: HttpContext) {
    // highlight-start
    if (await bouncer.with(PostPolicy).denies('create')) {
      return response.forbidden('Cannot create a post')
    }
    // highlight-end

    // 继续执行控制器逻辑
  }

  async edit({ bouncer, params, response }: HttpContext) {
    const post = await Post.findOrFail(params.id)

    // highlight-start
    if (await bouncer.with(PostPolicy).denies('edit', post)) {
      return response.forbidden('Cannot edit the post')
    }
    // highlight-end

    // 继续执行控制器逻辑
  }

  async delete({ bouncer, params, response }: HttpContext) {
    const post = await Post.findOrFail(params.id)

    // highlight-start
    if (await bouncer.with(PostPolicy).denies('delete', post)) {
      return response.forbidden('Cannot delete the post')
    }
    // highlight-end

    // 继续执行控制器逻辑
  }
}
```

### 允许访客用户

[与能力类似](#allowing-guest-users)，策略也可以使用 `@allowGuest` 装饰器为访客用户定义授权检查。例如：

```ts
import User from '#models/user'
import Post from '#models/post'
import { BasePolicy, allowGuest } from '@adonisjs/bouncer'
import type { AuthorizerResponse } from '@adonisjs/bouncer/types'

export default class PostPolicy extends BasePolicy {
  @allowGuest()
  view(user: User | null, post: Post): AuthorizerResponse {
    /**
     * 允许每个人访问已发布的帖子
     */
    if (post.isPublished) {
      return true
    }

    /**
     * 访客不能查看未发布的帖子
     */
    if (!user) {
      return false
    }

    /**
     * 帖子的创建者也可以查看未发布的帖子。
     */
    return user.id === post.userId
  }
}
```

### 策略钩子

您可以在策略类上定义 `before` 和 `after` 模板方法，以在授权检查前后执行操作。一个常见的用例是始终允许或拒绝某个用户的访问。

:::note

无论是否已登录用户，`before` 和 `after` 方法都会被调用。因此，请确保处理 `user` 值为 `null` 的情况。

:::

`before` 的响应解释如下：

- `true` 值将被视为成功的授权，并且不会调用操作方法。
- `false` 值将被视为访问被拒绝，并且不会调用操作方法。
- 如果返回值为 `undefined`，Bouncer 将执行操作方法以执行授权检查。

```ts
export default class PostPolicy extends BasePolicy {
  async before(user: User | null, action: string, ...params: any[]) {
    /**
     * 始终允许管理员用户，而不进行任何检查
     */
    if (user && user.isAdmin) {
      return true
    }
  }
}
```

`after` 方法接收操作方法的原始响应，并可以通过返回新值来覆盖之前的响应。`after` 的响应解释如下：

- `true` 值将被视为成功的授权，并且旧的响应将被丢弃。
- `false` 值将被视为访问被拒绝，并且旧的响应将被丢弃。
- 如果返回值为 `undefined`，Bouncer 将继续使用旧的响应。

```ts
import { AuthorizerResponse } from '@adonisjs/bouncer/types'

export default class PostPolicy extends BasePolicy {
  async after(
    user: User | null,
    action: string,
    response: AuthorizerResponse,
    ...params: any[]
  ) {
    if (user && user.isAdmin) {
      return true
    }
  }
}
```

### 依赖注入

策略类是使用 [IoC container](../concepts/dependency_injection.md) 创建的；因此，您可以在策略构造函数中使用 `@inject` 装饰器进行类型提示和依赖注入。

```ts
import { inject } from '@adonisjs/core'
import { PermissionsResolver } from '#services/permissions_resolver'

// highlight-start
@inject()
// highlight-end
export class PostPolicy extends BasePolicy {
  constructor(
    // highlight-start
    protected permissionsResolver: PermissionsResolver
    // highlight-end
  ) {
    super()
  }
}
```

如果在 HTTP 请求期间创建了策略类，您还可以在其中注入一个 [HttpContext](../concepts/http_context.md) 实例。

```ts
// highlight-start
import { HttpContext } from '@adonisjs/core/http'
// highlight-end
import { PermissionsResolver } from '#services/permissions_resolver'

@inject()
export class PostPolicy extends BasePolicy {
  // highlight-start
  constructor(protected ctx: HttpContext) {
  // highlight-end
    super()
  }
}
```

## 抛出 AuthorizationException

除了 `allows` 和 `denies` 方法外，您还可以使用 `bouncer.authorize` 方法执行授权检查。当检查失败时，该方法将抛出 [AuthorizationException](https://github.com/adonisjs/bouncer/blob/main/src/errors.ts#L19)。

```ts
router.put('posts/:id', async ({ bouncer, params }) => {
  const post = await Post.findOrFail(post)
  // highlight-start
  await bouncer.authorize(editPost, post)
  // highlight-end

  /**
   * 如果没有抛出异常，则可以认为用户被允许编辑帖子。
   */
})
```

AdonisJS 将使用以下内容协商规则将 `AuthorizationException` 转换为 `403 - Forbidden` HTTP 响应。

- 带有 `Accept=application/json` 头的 HTTP 请求将收到一个错误消息数组。每个数组元素将是一个带有 `message` 属性的对象。

- 带有 `Accept=application/vnd.api+json` 头的 HTTP 请求将收到一个根据 [JSON API](https://jsonapi.org/format/#errors) 规范格式化的错误消息数组。

- 所有其他请求将收到纯文本响应消息。但是，您可以使用 [status pages](../basics/exception_handling.md#status-pages) 为授权错误显示自定义错误页面。

您还可以在 [global exception handler](../basics/exception_handling.md) 中自行处理 `AuthorizationException` 错误。

```ts
import { errors } from '@adonisjs/bouncer'
import { HttpContext, ExceptionHandler } from '@adonisjs/core/http'

export default class HttpExceptionHandler extends ExceptionHandler {
  protected debug = !app.inProduction
  protected renderStatusPages = app.inProduction

  async handle(error: unknown, ctx: HttpContext) {
    // highlight-start
    if (error instanceof errors.E_AUTHORIZATION_FAILURE) {
      return ctx
        .response
        .status(error.status)
        .send(error.getResponseMessage(ctx))
    }
    // highlight-end

    return super.handle(error, ctx)
  }
}
```

## 自定义授权响应

您可以使用 [AuthorizationResponse](https://github.com/adonisjs/bouncer/blob/main/src/response.ts) 类构造错误响应，而不是从能力和策略中返回布尔值。

`AuthorizationResponse` 类为您提供细粒度的控制，以定义自定义 HTTP 状态码和错误消息。

```ts
import User from '#models/user'
import Post from '#models/post'
import { Bouncer, AuthorizationResponse } from '@adonisjs/bouncer'

export const editPost = Bouncer.ability((user: User, post: Post) => {
  if (user.id === post.userId) {
    return true
  }

  // highlight-start
  return AuthorizationResponse.deny('Post not found', 404)
  // highlight-end
})
```

如果您使用的是 [@adonisjs/i18n](../digging_deeper/i18n.md) 包，可以使用 `.t` 方法返回本地化响应。在 HTTP 请求期间，将根据用户的语言使用翻译消息而不是默认消息。

```ts
export const editPost = Bouncer.ability((user: User, post: Post) => {
  if (user.id === post.userId) {
    return true
  }

  // highlight-start
  return AuthorizationResponse
    .deny('Post not found', 404) // 默认消息
    .t('errors.not_found') // 翻译标识符
  // highlight-end
})
```

### 使用自定义响应构建器

为单个授权检查定义自定义错误消息非常灵活。但是，如果您始终希望返回相同的响应，每次都重复相同的代码可能会很麻烦。

因此，您可以按如下方式覆盖 Bouncer 的默认响应构建器。

```ts
import { Bouncer, AuthorizationResponse } from '@adonisjs/bouncer'

Bouncer.responseBuilder = (response: boolean | AuthorizationResponse) => {
  if (response instanceof AuthorizationResponse) {
    return response
  }

  if (response === true) {
    return AuthorizationResponse.allow()
  }

  return AuthorizationResponse
    .deny('Resource not found', 404)
    .t('errors.not_found')
}
```

## 预注册能力和策略

到目前为止，在本指南中，我们每次想要使用能力或策略时，都会显式地导入它们。但是，一旦您预注册了它们，就可以通过字符串名称引用能力或策略。

在 TypeScript 代码库中，预注册能力和策略可能比仅仅清理导入更有用。然而，它们在 Edge 模板中提供了更好的开发者体验（DX）。

查看以下 Edge 模板的代码示例，一个预注册了策略，另一个没有。

:::caption{for="error"}
**未预注册。不，不是很整洁**
:::

```edge
{{-- 首先导入能力 --}}
@let(editPost = (await import('#abilities/main')).editPost)

@can(editPost, post)
  {{-- 可以编辑帖子 --}}
@end
```

:::caption{for="success"}
**已预注册**
:::

```edge
{{-- 以字符串形式引用能力名称 --}}
@can('editPost', post)
  {{-- 可以编辑帖子 --}}
@end
```

如果您打开 `initialize_bouncer_middleware.ts` 文件，会发现我们在创建 Bouncer 实例时已经导入并预注册了能力和策略。

```ts
// highlight-start
import * as abilities from '#abilities/main'
import { policies } from '#policies/main'
// highlight-end

export default InitializeBouncerMiddleware {
  async handle(ctx, next) {
    ctx.bouncer = new Bouncer(
      () => ctx.auth.user,
      // highlight-start
      abilities,
      policies
      // highlight-end
    )
  }
}
```

### 注意事项

- 如果您决定在代码库的其他部分定义能力，请确保在中间件中导入并预注册它们。

- 对于策略，每次运行 `make:policy` 命令时，请确保接受提示，将策略注册到策略集合中。策略集合定义在 `./app/policies/main.ts` 文件中。

  ```ts
  // title: app/policies/main.ts
  export const policies = {
    PostPolicy: () => import('#policies/post_policy'),
    CommentPolicy: () => import('#policies/comment_policy')
  }
  ```

### 引用预注册的能力和策略

在以下示例中，我们移除了导入，并通过名称引用能力和策略。请注意，**基于字符串的 API 也是类型安全的**，但您的代码编辑器的“转到定义”功能可能无法工作。

```ts
// title: 能力使用示例
// delete-start
import { editPost } from '#abilities/main'
// delete-end

router.put('posts/:id', async ({ bouncer, params, response }) => {
  const post = await Post.findOrFail(params.id)

  // delete-start
  if (await bouncer.allows(editPost, post)) {
  // delete-end
  // insert-start
  if (await bouncer.allows('editPost', post)) {
  // insert-end
    return 'You can edit the post'
  }
})
```

```ts
// title: 策略使用示例
// delete-start
import PostPolicy from '#policies/post_policy'
// delete-end

export default class PostsController {
  async create({ bouncer, response }: HttpContext) {
    // delete-start
    if (await bouncer.with(PostPolicy).denies('create')) {
    // delete-end
    // insert-start
    if (await bouncer.with('PostPolicy').denies('create')) {
    // insert-end
      return response.forbidden('Cannot create a post')
    }

    // 继续执行控制器逻辑
  }
}
```

## 在 Edge 模板中进行授权检查

在 Edge 模板中进行授权检查之前，请确保 [预注册能力和策略](#pre-registering-abilities-and-policies)。完成后，您可以使用 `@can` 和 `@cannot` 标签进行授权检查。

这些标签接受 `ability` 名称或 `policy.method` 名称作为第一个参数，后跟能力或策略接受的其他参数。

```edge
// title: 使用能力
@can('editPost', post)
  {{-- 可以编辑帖子 --}}
@end

@cannot('editPost', post)
  {{-- 不能编辑帖子 --}}
@end
```

```edge
// title: 使用策略
@can('PostPolicy.edit', post)
  {{-- 可以编辑帖子 --}}
@end

@cannot('PostPolicy.edit', post)
  {{-- 不能编辑帖子 --}}
@end
```

## 事件

请查阅[事件参考指南](../references/events.md#authorizationfinished)，以查看 `@adonisjs/bouncer` 包发出的事件列表。
