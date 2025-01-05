---
summary: 了解如何在 AdonisJS 测试中模拟或伪造依赖项。
---

# Mocks 和 Fakes

在测试你的应用程序时，你可能希望模拟或伪造特定的依赖项，以防止实际实现运行。例如，你希望在运行测试时不要向客户发送电子邮件，也不要调用第三方支付网关等服务。

AdonisJS 提供了一些不同的 API 和建议，你可以使用它们来伪造、模拟或存根依赖项。

## 使用 fakes API

Fakes 是专为测试而创建的具有工作实现的对象。例如，AdonisJS 的 mailer 模块有一个伪实现，你可以在测试期间使用它来收集内存中的外发电子邮件并为它们编写断言。

我们为以下容器服务提供伪实现。fakes API 的文档与模块文档一起提供。

- [Emitter fake](../digging_deeper/emitter.md#faking-events-during-tests)
- [Hash fake](../security/hashing.md#faking-hash-service-during-tests)
- [Fake mailer](../digging_deeper/mail.md#fake-mailer)
- [Drive fake](../digging_deeper/drive.md#faking-disks)

## 依赖注入和 fakes

如果你在应用程序中使用依赖注入或使用[容器创建类实例](../concepts/dependency_injection.md)，则可以使用 `container.swap` 方法为类提供伪实现。

在以下示例中，我们将 `UserService` 注入到 `UsersController` 中。

```ts
import UserService from '#services/user_service'
import { inject } from '@adonisjs/core'

export default class UsersController {
  @inject()
  index(service: UserService) {}
}
```

在测试期间，我们可以如下提供 `UserService` 类的替代/伪实现。

```ts
import UserService from '#services/user_service'
import app from '@adonisjs/core/services/app'

test('get all users', async () => {
  // highlight-start
  class FakeService extends UserService {
    all() {
      return [{ id: 1, username: 'virk' }]
    }
  }
  
  /**
   * 用 `FakeService` 的实例替换 `UserService`
   */  
  app.container.swap(UserService, () => {
    return new FakeService()
  })
  
  /**
   * 测试逻辑在这里
   */
  // highlight-end
})
```

测试完成后，你必须使用 `container.restore` 方法恢复伪实现。

```ts
app.container.restore(UserService)

// 恢复 UserService 和 PostService
app.container.restoreAll([UserService, PostService])

// 恢复所有
app.container.restoreAll()
```

## 使用 Sinon.js 进行 Mocks 和 Stubs

[Sinon](https://sinonjs.org/#get-started) 是 Node.js 生态系统中一个成熟、经过时间考验的模拟库。如果你经常使用 mocks 和 stubs，我们建议使用 Sinon，因为它与 TypeScript 配合良好。

## 模拟网络请求

如果你的应用程序向第三方服务发出 HTTP 请求，你可以在测试期间使用 [nock](https://github.com/nock/nock) 来模拟网络请求。

由于 nock 拦截来自 [Node.js HTTP 模块](https://nodejs.org/dist/latest-v20.x/docs/api/http.html#class-httpclientrequest) 的所有外发请求，因此它几乎可以与每个第三方库（如 `got`、`axios` 等）一起工作。

## 冻结时间

在编写测试时，你可以使用 [timekeeper](https://www.npmjs.com/package/timekeeper) 包来冻结或穿越时间。timekeeper 包通过模拟 `Date` 类来工作。

在以下示例中，我们将 `timekeeper` 的 API 封装在 [Japa Test 资源](https://japa.dev/docs/test-resources) 中。

```ts
import { getActiveTest } from '@japa/runner'
import timekeeper from 'timekeeper'

export function timeTravel(secondsToTravel: number) {
  const test = getActiveTest()
  if (!test) {
    throw new Error('Cannot use "timeTravel" outside of a Japa test')
  }

  timekeeper.reset()

  const date = new Date()
  date.setSeconds(date.getSeconds() + secondsToTravel)
  timekeeper.travel(date)

  test.cleanup(() => {
    timekeeper.reset()
  })
}
```

你可以在测试中使用 `timeTravel` 方法，如下所示。

```ts
import { timeTravel } from '#test_helpers'

test('expire token after 2 hours', async ({ assert }) => {
  /**
   * 穿越到未来 3 小时
   */
  timeTravel(60 * 60 * 3)
})
```
