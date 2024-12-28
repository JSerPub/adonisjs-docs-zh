---
summary: 了解 AdonisJS 中的 HTTP 上下文，以及如何在路由处理器、中间件和异常处理器中访问它。
---

# HTTP 上下文

对于每个 HTTP 请求，都会生成一个新的 [HTTP Context 类](https://github.com/adonisjs/http-server/blob/main/src/http_context/main.ts) 实例，并传递给路由处理器、中间件和异常处理器。

HTTP 上下文包含与 HTTP 请求相关的所有信息。例如：

- 您可以使用 [ctx.request](../basics/request.md) 属性访问请求体、请求头和查询参数。
- 您可以使用 [ctx.response](../basics/response.md) 属性响应 HTTP 请求。
- 使用 [ctx.auth](../authentication/introduction.md) 属性访问已登录用户。
- 使用 [ctx.bouncer](../security/authorization.md) 属性对用户操作进行授权。
- 等等。

简而言之，上下文是一个请求特定的存储，保存着当前请求的所有信息。

## 访问 HTTP 上下文

HTTP 上下文通过引用传递给路由处理器、中间件和异常处理器，您可以按以下方式访问它。

### 路由处理器

[路由处理器](../basics/routing.md) 将 HTTP 上下文作为第一个参数接收。

```ts
import router from '@adonisjs/core/services/router'

router.get('/', (ctx) => {
  console.log(ctx.inspect())
})
```

```ts
// title: 解构属性
import router from '@adonisjs/core/services/router'

router.get('/', ({ request, response }) => {
  console.log(request.url())
  console.log(request.headers())
  console.log(request.qs())
  console.log(request.body())
  
  response.send('hello world')
  response.send({ hello: 'world' })
})
```

### 控制器方法

[控制器方法](../basics/controllers.md)（类似于路由处理器）将 HTTP 上下文作为第一个参数接收。

```ts
import { HttpContext } from '@adonisjs/core/http'

export default class HomeController {
  async index({ request, response }: HttpContext) {
  }
}
```

### 中间件类

[中间件类](../basics/middleware.md) 的 `handle` 方法将 HTTP 上下文作为第一个参数接收。

```ts
import { HttpContext } from '@adonisjs/core/http'

export default class AuthMiddleware {
  async handle({ request, response }: HttpContext) {
  }
}
```

### 异常处理器类

[全局异常处理器](../basics/exception_handling.md) 类的 `handle` 和 `report` 方法将 HTTP 上下文作为第二个参数接收。第一个参数是 `error` 属性。

```ts
import {
  HttpContext,
  HttpExceptionHandler
} from '@adonisjs/core/http'

export default class ExceptionHandler extends HttpExceptionHandler {
  async handle(error: unknown, ctx: HttpContext) {
    return super.handle(error, ctx)
  }

  async report(error: unknown, ctx: HttpContext) {
    return super.report(error, ctx)
  }
}
```

## 使用依赖注入注入 HTTP 上下文

如果您的应用程序中使用了依赖注入，可以通过类型提示 `HttpContext` 类，将 HTTP 上下文注入到类或方法中。

:::warning

确保 `#middleware/container_bindings_middleware` 中间件已在 `kernel/start.ts` 文件中注册。此中间件是解析请求特定值（即 `HttpContext` 类）所必需的。

:::

另请参阅：[IoC 容器指南](../concepts/dependency_injection.md)

```ts
// title: app/services/user_service.ts
import { inject } from '@adonisjs/core'
import { HttpContext } from '@adonisjs/core/http'

@inject()
export default class UserService {
  constructor(protected ctx: HttpContext) {}
  
  all() {
    // 方法实现
  }
}
```

为了使自动依赖解析生效，您必须在控制器中注入 `UserService`。记住，控制器方法的第一个参数始终是上下文，其余参数将通过 IoC 容器注入。

```ts
import { inject } from '@adonisjs/core'
import { HttpContext } from '@adonisjs/core/http'
import UserService from '#services/user_service'

export default class UsersController {
  @inject()
  index(ctx: HttpContext, userService: UserService) {
    return userService.all()
  }
}
```

完成！`UserService` 现在将自动接收当前 HTTP 请求的实例。您也可以对嵌套依赖重复此过程。

## 在应用程序中的任何地方访问 HTTP 上下文

依赖注入是一种将 HTTP 上下文作为类构造函数或方法依赖项接受的方式，然后依靠容器为您解析它。

然而，这并不是重构应用程序并在各处使用依赖注入的硬性要求。您还可以使用 Node.js 提供的 [异步本地存储](https://nodejs.org/dist/latest-v21.x/docs/api/async_context.html#class-asynclocalstorage) 在应用程序中的任何地方访问 HTTP 上下文。

我们有一个[专门指南](./async_local_storage.md)介绍异步本地存储的工作原理，以及 AdonisJS 如何使用它提供对 HTTP 上下文的全局访问。

在下面的示例中，`UserService` 类使用 `HttpContext.getOrFail` 方法获取当前请求的 HTTP 上下文实例。

```ts
// title: app/services/user_service.ts
import { HttpContext } from '@adonisjs/core/http'

export default class UserService {
  all() {
    const ctx = HttpContext.getOrFail()
    console.log(ctx.request.url())
  }
}
```

下面的代码块展示了 `UserService` 类在 `UsersController` 中的使用。

```ts
import { HttpContext } from '@adonisjs/core/http'
import UserService from '#services/user_service'

export default class UsersController {
  index(ctx: HttpContext) {
    const userService = new UserService()
    return userService.all()
  }
}
```

## HTTP 上下文属性

以下是通过 HTTP 上下文可以访问的属性列表。随着您安装新的包，它们可能会向上下文添加额外的属性。

<dl>
<dt>

ctx.request

</dt>

<dd>

对 [HTTP Request 类](../basics/request.md) 实例的引用。

</dd>

<dt>

ctx.response

</dt>

<dd>

对 [HTTP Response 类](../basics/response.md) 实例的引用。

</dd>

<dt>

ctx.logger

</dt>

<dd>

为给定 HTTP 请求创建的 [logger](../digging_deeper/logger.md) 实例的引用。

</dd>

<dt>

ctx.route

</dt>

<dd>

当前 HTTP 请求匹配的路由。`route` 属性是 [StoreRouteNode](https://github.com/adonisjs/http-server/blob/main/src/types/route.ts#L69) 类型的对象。

</dd>

<dt>

ctx.params

</dt>

<dd>

路由参数的对象。

</dd>

<dt>

ctx.subdomains

</dt>

<dd>

路由子域的对象。仅当路由是动态子域的一部分时才存在。

</dd>

<dt>

ctx.session

</dt>

<dd>

为当前 HTTP 请求创建的 [Session](../basics/session.md) 实例的引用。

</dd>

<dt>

ctx.auth

</dt>

<dd>

对 [Authenticator 类](https://github.com/adonisjs/auth/blob/main/src/authenticator.ts) 实例的引用。了解更多关于 [身份验证](../authentication/introduction.md) 的信息。

</dd>

<dt>

ctx.view

</dt>

<dd>

对 Edge 渲染器实例的引用。在 [视图和模板指南](../views-and-templates/introduction.md#using-edge) 中了解更多关于 Edge 的信息。

</dd>

<dt>

ctx.ally

</dt>

<dd>

对 [Ally Manager 类](https://github.com/adonisjs/ally/blob/main/src/ally_manager.ts) 实例的引用，用于在您的应用程序中实现社交登录。了解更多关于 [Ally](../authentication/social_authentication.md) 的信息。

</dd>

<dt>

ctx.bouncer

</dt>

<dl>

<dd>

对 [Bouncer 类](https://github.com/adonisjs/bouncer/blob/main/src/bouncer.ts) 实例的引用。了解更多关于 [授权](../security/authorization.md) 的信息。

</dd>

<dt>

ctx.i18n

</dt>

<dd>

对 [I18n 类](https://github.com/adonisjs/i18n/blob/main/src/i18n.ts) 实例的引用。在 [国际化](../digging_deeper/i18n.md) 指南中了解更多关于 `i18n` 的信息。

</dd>

</dl>

## 扩展 HTTP 上下文

你可以使用宏或 getter 向 HTTP 上下文类添加自定义属性。如果你是初次接触宏的概念，请务必先阅读 [扩展 AdonisJS 指南](./extending_the_framework.md)。

```ts
import { HttpContext } from '@adonisjs/core/http'

HttpContext.macro('aMethod', function (this: HttpContext) {
  return value
})

HttpContext.getter('aProperty', function (this: HttpContext) {
  return value
})
```

由于宏和 getter 是在运行时添加的，因此你必须使用模块增强来告知 TypeScript 它们的类型。

```ts
import { HttpContext } from '@adonisjs/core/http'

// insert-start
declare module '@adonisjs/core/http' {
  export interface HttpContext {
    aMethod: () => ValueType
    aProperty: ValueType
  }
}
// insert-end

HttpContext.macro('aMethod', function (this: HttpContext) {
  return value
})

HttpContext.getter('aProperty', function (this: HttpContext) {
  return value
})
```

## 在测试中创建虚拟上下文

在测试期间，你可以使用 `testUtils` 服务创建一个虚拟的 HTTP 上下文。

该上下文实例未附加到任何路由；因此，`ctx.route` 和 `ctx.params` 的值将是 undefined。但是，如果测试中的代码需要这些属性，你可以手动分配它们。

```ts
import testUtils from '@adonisjs/core/services/test_utils'

const ctx = testUtils.createHttpContext()
```

默认情况下，`createHttpContext` 方法为 `req` 和 `res` 对象使用假值。但是，你可以为这些属性定义自定义值，如以下示例所示。

```ts
import { createServer } from 'node:http'
import testUtils from '@adonisjs/core/services/test_utils'

createServer((req, res) => {
  const ctx = testUtils.createHttpContext({
    // highlight-start
    req,
    res
    // highlight-end
  })
})
```

### 使用 HttpContext 工厂

`testUtils` 服务仅在 AdonisJS 应用程序内部可用；因此，如果你在构建一个包并且需要访问一个虚拟的 HTTP 上下文，你可以使用 [HttpContextFactory](https://github.com/adonisjs/http-server/blob/main/factories/http_context.ts#L30) 类。

```ts
import { HttpContextFactory } from '@adonisjs/core/factories/http'
const ctx = new HttpContextFactory().create()
```