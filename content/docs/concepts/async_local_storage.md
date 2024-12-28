---
summary: 了解 AsyncLocalStorage 以及如何在 AdonisJS 中使用它。
---

# 异步本地存储

根据 [Node.js 官方文档](https://nodejs.org/docs/latest-v21.x/api/async_context.html#class-asynclocalstorage)：“AsyncLocalStorage 用于在回调和 Promise 链中创建异步状态。**它允许在 Web 请求或其他任何异步过程的整个生命周期内存储数据。它类似于其他语言中的线程本地存储**。”

为了简化解释，AsyncLocalStorage 允许在执行异步函数时存储一个状态，并使该状态在该函数内的所有代码路径中可用。

## 基本示例

让我们来看看它的实际应用。首先，我们将创建一个新的 Node.js 项目（不依赖任何库），并使用 `AsyncLocalStorage` 在模块之间共享状态，而无需通过引用传递。

:::note

您可以在 [als-basic-example](https://github.com/thetutlage/als-basic-example) GitHub 仓库中找到此示例的最终代码。

:::

### 步骤 1. 创建新项目

```sh
npm init --yes
```

打开 `package.json` 文件，并将模块系统设置为 ESM。

```json
{
  "type": "module"
}
```

### 步骤 2. 创建 `AsyncLocalStorage` 实例

创建一个名为 `storage.js` 的文件，该文件创建并导出一个 `AsyncLocalStorage` 实例。

```ts
// title: storage.js
import { AsyncLocalStorage } from 'async_hooks'
export const storage = new AsyncLocalStorage()
```

### 步骤 3. 在 `storage.run` 中执行代码

创建一个名为 `main.js` 的入口点文件。在此文件中，导入在 `./storage.js` 文件中创建的 `AsyncLocalStorage` 实例。

`storage.run` 方法接受我们想要共享的状态作为第一个参数，以及一个回调函数作为第二个参数。此回调函数内的所有代码路径（包括导入的模块）都将访问相同的状态。

```ts
// title: main.js
import { storage } from './storage.js'
import UserService from './user_service.js'
import { setTimeout } from 'node:timers/promises'

async function run(user) {
  const state = { user }

  return storage.run(state, async () => {
    await setTimeout(100)
    const userService = new UserService()
    await userService.get()
  })
}
```

为了演示，我们将三次执行 `run` 方法，而不等待其完成。将以下代码粘贴到 `main.js` 文件的末尾。

```ts
// title: main.js
run({ id: 1 })
run({ id: 2 })
run({ id: 3 })
```

### 步骤 4. 从 `user_service` 模块访问状态

最后，让我们在 `user_service` 模块中导入存储实例并访问当前状态。

```ts
// title: user_service.js
import { storage } from './storage.js'

export class UserService {
  async get() {
    const state = storage.getStore()
    console.log(`用户 ID 是 ${state.user.id}`)
  }
}
```

### 步骤 5. 执行 `main.js` 文件

让我们运行 `main.js` 文件，看看 `UserService` 是否能访问状态。

```sh
node main.js
```

## 为什么需要异步本地存储？

与其他语言（如 PHP）不同，Node.js 不是一种多线程语言。在 PHP 中，每个 HTTP 请求都会创建一个新线程，每个线程都有自己的内存。这允许你在全局内存中存储状态，并在代码库中的任何地方访问它。

在 Node.js 中，你不能在 HTTP 请求之间隔离全局状态，因为 Node.js 在单个线程上运行并且具有共享内存。因此，所有 Node.js 应用程序都通过参数传递数据来共享数据。

通过引用传递数据在技术上没有缺点。但是，它确实会使代码变得冗长，尤其是在配置 APM 工具时，需要手动向他们提供请求数据。

## 使用方法

AdonisJS 在 HTTP 请求期间使用 `AsyncLocalStorage`，并将 [HTTP 上下文](./http_context.md) 作为状态共享。因此，你可以在应用程序中全局访问 HTTP 上下文。

首先，你必须在 `config/app.ts` 文件中启用 `useAsyncLocalStorage` 标志。

```ts
// title: config/app.ts
export const http = defineConfig({
  useAsyncLocalStorage: true,
})
```

启用后，你可以使用 `HttpContext.get` 或 `HttpContext.getOrFail` 方法来获取当前请求的 HTTP 上下文实例。

在以下示例中，我们在 Lucid 模型中获取上下文。

```ts
import { HttpContext } from '@adonisjs/core/http'
import { BaseModel } from '@adonisjs/lucid'

export default class Post extends BaseModel {
  get isLiked() {
    const ctx = HttpContext.getOrFail()
    const authUserId = ctx.auth.user.id
    
    return !!this.likes.find((like) => {
      return like.userId === authUserId
    })
  }
}
```

## 注意事项
如果你认为使用 ALS 能使代码更简洁，并且你更倾向于全局访问而不是通过引用传递 HTTP 上下文，那么你可以使用 ALS。

然而，请注意以下情况，这些情况很容易导致内存泄漏或程序不稳定。

### 顶级访问

不要在任何模块的顶级访问 ALS，因为 Node.js 中的模块是缓存的。

:::caption{for="error"}
**错误用法**\
在顶级将 `HttpContext.getOrFail()` 方法的结果赋值给变量，将持有对首次导入该模块的请求的引用。
:::

```ts
import { HttpContext } from '@adonisjs/core/http'
const ctx = HttpContext.getOrFail()

export default class UsersController {
  async index() {
    ctx.request
  }
}
```

:::caption[]{for="success"}
**正确用法**\
相反，你应该将 `getOrFail` 方法调用移到 `index` 方法内。
:::

```ts
import { HttpContext } from '@adonisjs/core/http'

export default class UsersController {
  async index() {
    const ctx = HttpContext.getOrFail()
  }
}
```

### 静态属性内

任何类的静态属性在模块被导入时立即求值；因此，你不应在静态属性内访问 HTTP 上下文。

:::caption{for="error"}
**错误用法**
:::

```ts
import { HttpContext } from '@adonisjs/core/http'
import { BaseModel } from '@adonisjs/lucid'

export default class User extends BaseModel {
  static connection = HttpContext.getOrFail().tenant.name
}
```

:::caption[]{for="success"}
**正确用法**\
相反，你应该将 `HttpContext.get` 调用移到方法内，或将属性转换为 getter。
:::

```ts
import { HttpContext } from '@adonisjs/core/http'
import { BaseModel } from '@adonisjs/lucid'

export default class User extends BaseModel {
  static query() {
    const ctx = HttpContext.getOrFail()
    return super.query({ connection: tenant.connection })
  }
}
```

### 事件处理程序

事件处理程序在 HTTP 请求完成后执行。因此，你应该避免在其中尝试访问 HTTP 上下文。

```ts
import emitter from '@adonisjs/core/services/emitter'

export default class UsersController {
  async index() {
    const user = await User.create({})
    emitter.emit('new:user', user)
  }
}
```

:::caption[]{for="error"}
**避免在事件监听器中使用**
:::

```ts
import { HttpContext } from '@adonisjs/core/http'
import emitter from '@adonisjs/core/services/emitter'

emitter.on('new:user', () => {
  const ctx = HttpContext.getOrFail()
})
```