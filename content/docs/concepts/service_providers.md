---
summary: 服务提供者是普通的 JavaScript 类，具有生命周期方法，可以在应用程序的不同阶段执行操作。
---

# 服务提供者

服务提供者是普通的 JavaScript 类，具有生命周期方法，可以在应用程序的不同阶段执行操作。

服务提供者可以在[容器中进行绑定](../concepts/dependency_injection.md#container-bindings)、[扩展现有绑定](../concepts/dependency_injection.md#container-events)，或者在HTTP服务器启动后运行操作。

服务提供者是 AdonisJS 应用程序的入口点，能够在应用程序被认为准备就绪之前修改其状态。**它主要用于外部包，以便钩入应用程序的生命周期**。

:::note
如果你只想在一个类中注入依赖项，可以使用[依赖注入](../concepts/dependency_injection.md)功能。
:::

提供者是在`adonisrc.ts`文件中的`providers`数组内进行注册的。其值是一个函数，用于延迟导入服务提供者。

```ts
{
  providers: [
    () => import('@adonisjs/core/providers/app_provider'),
    () => import('./providers/app_provider.js'),
  ]
}
```

默认情况下，提供者会在所有运行时环境中加载。但是，你可以限制提供者仅在特定环境中运行。

```ts
{
  providers: [
    () => import('@adonisjs/core/providers/app_provider'),
    {
      file: () => import('./providers/app_provider.js'),
      environment: ['web', 'repl']
    }
  ]
}
```

## 编写服务提供者

服务提供者存储在应用程序的 `providers` 目录中。或者，你可以使用`node ace make:provider app`命令。

提供者模块必须具有一个`export default`语句，返回一个提供者类。类的构造函数接收一个[Application](./application.md)类的实例。

另请参阅：[Make provider命令](../references/commands.md#makeprovider)

```ts
import { ApplicationService } from '@adonisjs/core/types'

export default class AppProvider {
  constructor(protected app: ApplicationService) {
  }
}
```

以下是你可以实现的生命周期方法，用于执行不同的操作。

```ts
export default class AppProvider {
  register() {
  }
  
  async boot() {
  }
  
  async start() {
  }
  
  async ready() {
  }
  
  async shutdown() {
  }
}
```

### register

在创建提供者类实例后，会调用 `register` 方法。 `register` 方法可以在IoC容器中注册绑定。

`register`方法是同步的，因此不能在此方法中使用Promise。

```ts
export default class AppProvider {
  register() {
    this.app.container.bind('db', () => {
      return new Database()
    })
  }
}
```

### boot

在所有绑定都已注册到 IoC 容器后，会调用 `boot` 方法。在此方法中，你可以从容器中解析绑定以扩展/修改它们。

```ts
export default class AppProvider {
  async boot() {
   const validator = await this.app.container.make('validator')
    
   // 添加自定义验证规则
   validator.rule('foo', () => {})
  }
}
```

在绑定从容器中解析时扩展它们是一个良好的实践。例如，你可以使用 `resolving` 钩子为验证器添加自定义规则。

```ts
async boot() {
  this.app.container.resolving('validator', (validator) => {
    validator.rule('foo', () => {})
  })
}
```

### start

`start` 方法在 `boot` 方法之后、 `ready` 方法之前调用。它允许你执行 `ready` 钩子操作可能需要的操作。

### ready

根据应用程序的环境，`ready` 方法会在不同的阶段被调用。

<table>
    <tr>
        <td width="100"><code>web</code></td>
        <td>在HTTP服务器启动并准备好接受请求后，会调用<code>ready</code>方法。</td>
    </tr>
    <tr>
        <td width="100"><code>console</code></td>
        <td>在主命令的<code>run</code>方法之前，会调用<code>ready</code>方法。</td>
    </tr>
    <tr>
        <td width="100"><code>test</code></td>
        <td>在运行所有测试之前，会调用<code>ready</code>方法。但是，测试文件在<code>ready</code>方法之前被导入。</td>
    </tr>
    <tr>
        <td width="100"><code>repl</code></td>
        <td>在终端显示REPL提示符之前，会调用<code>ready</code>方法。</td>
    </tr>
</table>

```ts
export default class AppProvider {
  async start() {
    if (this.app.getEnvironment() === 'web') {
    }

    if (this.app.getEnvironment() === 'console') {
    }

    if (this.app.getEnvironment() === 'test') {
    }

    if (this.app.getEnvironment() === 'repl') {
    }
  }
}
```

### shutdown

当AdonisJS正在优雅地退出应用程序时，会调用`shutdown`方法。

退出应用程序的事件取决于应用程序运行的环境以及应用程序进程是如何启动的。请阅读[应用程序生命周期指南](./application_lifecycle.md)以了解更多信息。

```ts
export default class AppProvider {
  async shutdown() {
    // 执行清理操作
  }
}
```