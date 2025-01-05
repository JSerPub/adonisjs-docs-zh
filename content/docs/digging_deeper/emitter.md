---
summary: 基于 emittery 创建的内置事件触发器。异步分发事件并修复了 Node.js 默认事件触发器的许多常见问题。
---

# 事件触发器

AdonisJS 具有一个基于 [emittery](https://github.com/sindresorhus/emittery) 创建的内置事件触发器。Emittery 异步分发事件，并[修复了 Node.js 默认事件触发器的许多常见问题](https://github.com/sindresorhus/emittery#how-is-this-different-than-the-built-in-eventemitter-in-nodejs)。

AdonisJS 通过附加功能进一步增强了 emittery。

- 通过定义事件列表及其关联的数据类型，提供静态类型安全性。
- 支持基于类的事件和监听器。将监听器移动到专用文件中，可以使代码库保持整洁且易于测试。
- 能够在测试期间模拟事件。

## 基本用法

事件监听器在 `start/events.ts` 文件中定义。你可以使用 `make:preload` ace 命令创建此文件。

```sh
node ace make:preload events
```

你必须使用 `emitter.on` 来监听事件。该方法将事件名称作为第一个参数，将监听器作为第二个参数。

```ts
// title: start/events.ts
import emitter from '@adonisjs/core/services/emitter'

emitter.on('user:registered', function (user) {
  console.log(user)
})
```

定义事件监听器后，你可以使用 `emitter.emit` 方法触发 `user:registered` 事件。该方法将事件名称作为第一个参数，将事件数据作为第二个参数。

```ts
import emitter from '@adonisjs/core/services/emitter'

export default class UsersController {
  async store() {
    const user = await User.create(data)
    emitter.emit('user:registered', user)
  }
}
```

你可以使用 `emitter.once` 来监听一次事件。

```ts
emitter.once('user:registered', function (user) {
  console.log(user)
})
```

## 使事件类型安全

AdonisJS 要求为你在应用程序中希望触发的每个事件定义静态类型。类型在 `types/events.ts` 文件中注册。

在以下示例中，我们将 `User` 模型注册为 `user:registered` 事件的数据类型。

:::note

如果你觉得为每个事件定义类型很繁琐，可以切换到[基于类的事件](#class-based-events)。

:::

```ts
import User from '#models/User'

declare module '@adonisjs/core/types' {
  interface EventsList {
    'user:registered': User
  }
}
```

## 基于类的监听器

与 HTTP 控制器类似，监听器类提供了一个抽象层，用于将内联事件监听器移动到专用文件中。监听器类存储在 `app/listeners` 目录中，你可以使用 `make:listener` 命令创建新的监听器。

另请参阅：[Make listener 命令](../references/commands.md#makelistener)

```sh
node ace make:listener sendVerificationEmail
```

监听器文件使用 `class` 声明和 `handle` 方法进行构建。在此类中，你可以定义附加方法来监听多个事件（如果需要）。

```ts
import User from '#models/user'

export default class SendVerificationEmail {
  handle(user: User) {
    // 发送电子邮件
  }
}
```

作为最后一步，你必须在 `start/events.ts` 文件中将监听器类绑定到事件。你可以使用 `#listeners` 别名导入监听器。别名是使用 [Node.js 的子路径导入功能](../getting_started/folder_structure.md#the-sub-path-imports) 定义的。

```ts
// title: start/events.ts
import emitter from '@adonisjs/core/services/emitter'
import SendVerificationEmail from '#listeners/send_verification_email'

emitter.on('user:registered', [SendVerificationEmail, 'handle'])
```

### 延迟加载监听器

建议延迟加载监听器以加快应用程序启动阶段的速度。延迟加载只需将导入语句放在函数后面并使用动态导入即可。

```ts
import emitter from '@adonisjs/core/services/emitter'
// delete-start
import SendVerificationEmail from '#listeners/send_verification_email'
// delete-end
// insert-start
const SendVerificationEmail = () => import('#listeners/send_verification_email')
// insert-end

emitter.on('user:registered', [SendVerificationEmail, 'handle'])
```

### 依赖注入

:::warning

你不能在监听器类中注入 `HttpContext`。因为事件是异步处理的，监听器可能在 HTTP 请求完成后才运行。

:::

监听器类是使用 [IoC 容器](../concepts/dependency_injection.md) 实例化的；因此，你可以在类构造函数或处理事件的方法中对依赖项进行类型提示。

在以下示例中，我们将 `TokensService` 作为构造函数参数进行类型提示。在调用此监听器时，IoC 容器将注入 `TokensService` 类的实例。

```ts
// title: 构造函数注入
import { inject } from '@adonisjs/core'
import TokensService from '#services/tokens_service'

@inject()
export default class SendVerificationEmail {
  constructor(protected tokensService: TokensService) {}

  handle(user: User) {
    const token = this.tokensService.generate(user.email)
  }
}
```

在以下示例中，我们在 handle 方法中注入 `TokensService`。但是，请记住，第一个参数始终是事件负载。

```ts
// title: 方法注入
import { inject } from '@adonisjs/core'
import TokensService from '#services/tokens_service'
import UserRegistered from '#events/user_registered'

export default class SendVerificationEmail {
  @inject()
  handle(event: UserRegistered, tokensService: TokensService) {
    const token = tokensService.generate(event.user.email)
  }
}
```

## 基于类的事件

事件是事件标识符（通常是基于字符串的事件名称）和关联数据的组合。例如：

- `user:registered` 是事件标识符（即事件名称）。
- User 模型的实例是事件数据。

基于类的事件将事件标识符和事件数据封装在同一个类中。类构造函数作为标识符，而类的实例保存事件数据。

你可以使用 `make:event` 命令创建事件类。

另请参阅：[Make event 命令](../references/commands.md#makeevent)

```sh
node ace make:event UserRegistered
```

事件类没有行为，而是事件的数据容器。你可以通过类构造函数接受事件数据，并将其作为实例属性提供。

```ts
// title: app/events/user_registered.ts
import { BaseEvent } from '@adonisjs/core/events'
import User from '#models/user'

export default class UserRegistered extends BaseEvent {
  constructor(public user: User) {} 
}
```

### 监听基于类的事件

你可以使用 `emitter.on` 方法为基于类的事件附加监听器。第一个参数是事件类引用，后面是监听器。

```ts
import emitter from '@adonisjs/core/services/emitter'
import UserRegistered from '#events/user_registered'

emitter.on(UserRegistered, function (event) {
  console.log(event.user)
})
```

以下示例中，我们同时使用了基于类的事件和监听器。

```ts
import emitter from '@adonisjs/core/services/emitter'

import UserRegistered from '#events/user_registered'
const SendVerificationEmail = () => import('#listeners/send_verification_email')

emitter.on(UserRegistered, [SendVerificationEmail])
```

### 触发基于类的事件

你可以使用 `static dispatch` 方法触发基于类的事件。`dispatch` 方法接受事件类构造函数所接受的参数。

```ts
import User from '#models/user'
import UserRegistered from '#events/user_registered'

export default class UsersController {
  async store() {
    const user = await User.create(data)
    
    /**
     * 派发/触发事件
     */
    UserRegistered.dispatch(user)
  }
}
```

## 简化事件监听体验

如果你决定使用基于类的事件和监听器，可以使用 `emitter.listen` 方法来简化绑定监听器的过程。

`emitter.listen` 方法接受事件类作为第一个参数，接受基于类的监听器数组作为第二个参数。此外，注册的监听器必须具有 `handle` 方法来处理事件。

```ts
import emitter from '@adonisjs/core/services/emitter'
import UserRegistered from '#events/user_registered'

emitter.listen(UserRegistered, [
  () => import('#listeners/send_verification_email'),
  () => import('#listeners/register_with_payment_provider'),
  () => import('#listeners/provision_account')
])
```

## 处理错误

默认情况下，监听器抛出的异常会导致 [unhandledRejection](https://nodejs.org/api/process.html#event-unhandledrejection)。因此，建议使用 `emitter.onError` 方法自行捕获和处理错误。

`emitter.onError` 方法接受一个回调函数，该函数接收事件名称、错误和事件数据。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.onError((event, error, eventData) => {
})
```

## 监听所有事件

你可以使用 `emitter.onAny` 方法监听所有事件。该方法接受监听器回调函数作为唯一参数。

```ts
import emitter from '@adonisjs/core/services/emitter'

emitter.onAny((name, event) => {
  console.log(name)
  console.log(event)
})
```

## 取消订阅事件

`emitter.on` 方法返回一个取消订阅函数，你可以调用该函数来移除事件监听器订阅。

```ts
import emitter from '@adonisjs/core/services/emitter'

const unsubscribe = emitter.on('user:registered', () => {})

// 移除监听器
unsubscribe()
```

你也可以使用 `emitter.off` 方法来移除事件监听器订阅。该方法接受事件名称/类作为第一个参数，接受监听器的引用作为第二个参数。

```ts
import emitter from '@adonisjs/core/services/emitter'

function sendEmail () {}

// 监听事件
emitter.on('user:registered', sendEmail)

// 移除监听器
emitter.off('user:registered', sendEmail)
```

### emitter.offAny

`emitter.offAny` 移除通配符监听器，该监听器监听所有事件。

```ts
emitter.offAny(callback)
```

### emitter.clearListeners

`emitter.clearListeners` 方法移除给定事件的所有监听器。

```ts
// 基于字符串的事件
emitter.clearListeners('user:registered')

// 基于类的事件
emitter.clearListeners(UserRegistered)
```

### emitter.clearAllListeners

`emitter.clearAllListeners` 方法移除所有事件的所有监听器。

```ts
emitter.clearAllListeners()
```

## 可用事件列表

请查阅 [events reference guide](../references/events.md) 以查看可用事件列表。

## 测试期间模拟事件

事件监听器通常用于在给定操作后执行副作用。例如：向新注册的用户发送电子邮件，或在订单状态更新后发送通知。

在编写测试时，你可能希望避免向测试期间创建的用户发送电子邮件。

你可以通过模拟事件触发器来禁用事件监听器。在以下示例中，我们在发出 HTTP 请求以注册用户之前调用 `emitter.fake`。请求发出后，我们使用 `events.assertEmitted` 方法确保 `SignupController` 触发了特定事件。

```ts
import emitter from '@adonisjs/core/services/emitter'
import UserRegistered from '#events/user_registered'

test.group('User signup', () => {
  test('create a user account', async ({ client, cleanup }) => {
    // highlight-start
    const events = emitter.fake()
    cleanup(() => {
      emitter.restore()
    })
    // highlight-end
  
    await client
      .post('signup')
      .form({
        email: 'foo@bar.com',
        password: 'secret',
      })
  })
  
  // highlight-start
  // 断言事件已触发
  events.assertEmitted(UserRegistered)
  // highlight-end
})
```

- `event.fake` 方法返回 [EventBuffer](https://github.com/adonisjs/events/blob/main/src/events_buffer.ts) 类的实例，你可以使用它进行断言和查找触发的事件。
- `emitter.restore` 方法恢复模拟。恢复模拟后，事件将正常触发。

### 模拟特定事件

如果你在调用 `emitter.fake` 方法时不传递任何参数，该方法将模拟所有事件。如果你希望模拟特定事件，请将事件名称或类作为第一个参数传递。

```ts
// 仅模拟 user:registered 事件
emitter.fake('user:registered')

// 模拟多个事件
emitter.fake([UserRegistered, OrderUpdated])
```

多次调用 `emitter.fake` 方法将移除旧的模拟。

### 事件断言

你可以使用 `assertEmitted`、`assertNotEmitted`、`assertNoneEmitted` 和 `assertEmittedCount` 方法为模拟的事件编写断言。

`assertEmitted` 和 `assertNotEmitted` 方法接受事件名称或类构造函数作为第一个参数，并接受一个可选的查找函数，该函数必须返回布尔值以标记事件是否已触发。

```ts
const events = emitter.fake()

events.assertEmitted('user:registered')
events.assertNotEmitted(OrderUpdated)
```

```ts
// title: 使用回调函数
events.assertEmitted(OrderUpdated, ({ data }) => {
  /**
   * 仅当 orderId 匹配时，才将事件视为已触发
   */
  return data.order.id === orderId
})
```

```ts
// title: 断言事件计数
// 断言特定事件的计数
events.assertEmittedCount(OrderUpdated, 1)

// 断言未触发任何事件
events.assertNoneEmitted()
```
