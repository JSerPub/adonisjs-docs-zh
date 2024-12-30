---
summary: 了解如何在 AdonisJS 中创建自定义 Ace 命令
---

# 创建命令

除了使用 Ace 命令外，您还可以创建自定义命令作为应用程序代码库的一部分。命令存储在根级别的 `commands` 目录中。您可以通过运行以下命令来创建一个命令。

另请参阅：[Make command](../references/commands.md#makecommand)

```sh
node ace make:command greet
```

上述命令将在 `commands` 目录中创建一个 `greet.ts` 文件。Ace 命令由一个类表示，并且必须实现 `run` 方法以执行命令指令。

## 命令元数据

命令元数据包括 **命令名称**、**描述**、**帮助文本** 和配置命令行为的 **选项**。

```ts
import { BaseCommand } from '@adonisjs/core/ace'
import { CommandOptions } from '@adonisjs/core/types/ace'

export default class GreetCommand extends BaseCommand {
  static commandName = 'greet'
  static description = '通过名字问候用户'

  static options: CommandOptions = {
    startApp: false,
    allowUnknownFlags: false,
    staysAlive: false,
  }
}
```

<dl>
<dt>
  commandName
</dt>

<dd>

`commandName` 属性用于定义命令名称。命令名称不应包含空格。此外，建议避免在命令名称中使用不熟悉的特殊字符，如 `*`、`&` 或斜杠。

命令名称可以位于命名空间下。例如，要在 `make` 命名空间下定义一个命令，可以在其前加上 `make:` 前缀。

</dd>

<dt>
  description
</dt>

<dd>

命令描述显示在命令列表中以及命令帮助屏幕上。您必须保持描述简短，并使用 **帮助文本** 提供更长的描述。

</dd>

<dt>
  help
</dt>

<dd>

帮助文本用于编写更长的描述或显示用法示例。

```ts
export default class GreetCommand extends BaseCommand {
  static help = [
    'The greet command is used to greet a user by name',
    '',
    'You can also send flowers to a user, if they have an updated address',
    '{{ binaryName }} greet --send-flowers',
  ]
}
```

> `{{ binaryName }}` 变量替换是对用于执行 Ace 命令的二进制文件的引用。

</dd>

<dt>
  aliases
</dt>

<dd>

您可以使用 `aliases` 属性为命令名称定义一个或多个别名。

```ts
export default class GreetCommand extends BaseCommand {
  static commandName = 'greet'
  static aliases = ['welcome', 'sayhi']
}
```

</dd>

<dt>
  options.startApp
</dt>

<dd>

默认情况下，AdonisJS 在运行 Ace 命令时不会启动应用程序。这确保了命令运行迅速，并且不需要经过应用程序启动阶段来执行简单任务。

然而，如果您的命令依赖于应用程序状态，您可以告诉 Ace 在执行命令之前启动应用程序。

```ts
import { BaseCommand } from '@adonisjs/core/ace'
import { CommandOptions } from '@adonisjs/core/types/ace'

export default class GreetCommand extends BaseCommand {
  // highlight-start
  static options: CommandOptions = {
    startApp: true
  }
  // highlight-end
}
```

</dd>

<dt>
  options.allowUnknownFlags
</dt>

<dd>

默认情况下，如果您向命令传递了未知标志，Ace 会打印错误。然而，您可以使用 `options.allowUnknownFlags` 属性在命令级别禁用严格标志解析。

```ts
import { BaseCommand } from '@adonisjs/core/ace'
import { CommandOptions } from '@adonisjs/core/types/ace'

export default class GreetCommand extends BaseCommand {
  // highlight-start
  static options: CommandOptions = {
    allowUnknownFlags: true
  }
  // highlight-end
}
```

</dd>

<dt>
  options.staysAlive
</dt>

<dd>

AdonisJS 在执行命令的 `run` 方法后会隐式终止应用程序。然而，如果您希望在命令中启动一个长时间运行的过程，您必须告诉 Ace 不要终止该过程。

另请参阅：[Terminating app](#terminating-application) 和 [cleaning up before the app terminates](#cleaning-up-before-the-app-terminates) 部分。

```ts
import { BaseCommand } from '@adonisjs/core/ace'
import { CommandOptions } from '@adonisjs/core/types/ace'

export default class GreetCommand extends BaseCommand {
  // highlight-start
  static options: CommandOptions = {
    staysAlive: true
  }
  // highlight-end
}
```
</dd>
</dl>

## 命令生命周期方法

您可以在命令类上定义以下生命周期方法，Ace 将按预定义顺序执行它们。

```ts
import { BaseCommand, args, flags } from '@adonisjs/core/ace'

export default class GreetCommand extends BaseCommand {
  async prepare() {
    console.log('preparing')
  }

  async interact() {
    console.log('interacting')
  }
  
  async run() {
    console.log('running')
  }

  async completed() {
    console.log('completed')
  }
}
```
<table>
  <thead>
    <tr>
      <th width="120px">方法</th>
      <th>描述</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <code>prepare</code>
      </td>
      <td>这是 Ace 在命令上执行的第一个方法。此方法可以设置运行命令所需的状态或数据。</td>
    </tr>
    <tr>
      <td>
        <code>interact</code>
      </td>
      <td><code>prepare</code> 方法之后执行 interact 方法。它可以用于向用户显示提示。</td>
    </tr>
    <tr>
      <td>
        <code>run</code>
      </td>
      <td>命令的主要逻辑放在 <code>run</code> 方法中。此方法在 <code>interact</code> 方法之后调用。</td>
    </tr>
    <tr>
      <td>
        <code>completed</code>
      </td>
      <td>在所有其他生命周期方法运行后调用 completed 方法。此方法可用于执行清理操作或处理/显示其他方法抛出的异常。</td>
    </tr>
  </tbody>
</table>

## 依赖注入

Ace 命令是使用 [IoC 容器](../concepts/dependency_injection.md) 构造和执行的。因此，你可以在命令生命周期方法中通过类型提示来声明依赖，并使用 `@inject` 装饰器来解析它们。

为了演示，让我们在所有生命周期方法中注入 `UserService` 类。

```ts
import { inject } from '@adonisjs/core'
import { BaseCommand } from '@adonisjs/core/ace'
import UserService from '#services/user_service'

export default class GreetCommand extends BaseCommand {
  @inject()
  async prepare(userService: UserService) {
  }

  @inject()
  async interact(userService: UserService) {
  }
  
  @inject()
  async run(userService: UserService) {
  }

  @inject()
  async completed(userService: UserService) {
  }
}
```

## 处理错误和退出代码

命令抛出的异常会使用 CLI 记录器显示，并且命令的 `exitCode` 被设置为 `1`（非零错误代码表示命令失败）。

然而，你也可以通过将代码包裹在 `try/catch` 块中来捕获错误，或者使用 `completed` 生命周期方法来处理错误。在这两种情况下，请记得更新命令的 `exitCode` 和 `error` 属性。

### 使用 try/catch 处理错误

```ts
import { BaseCommand } from '@adonisjs/core/ace'

export default class GreetCommand extends BaseCommand {
  async run() {
    try {
      await runSomeOperation()
    } catch (error) {
      this.logger.error(error.message)
      this.error = error
      this.exitCode = 1
    }
  }
}
```

### 在 completed 方法中处理错误

```ts
import { BaseCommand } from '@adonisjs/core/ace'

export default class GreetCommand extends BaseCommand {
  async run() {
    await runSomeOperation()
  }
  
  async completed() {
    if (this.error) {
      this.logger.error(this.error.message)
      
      /**
       * 通知 Ace 错误已被处理
       */
      return true
    }
  }
}
```

## 终止应用程序

默认情况下，Ace 在执行命令后会终止应用程序。但是，如果你启用了 `staysAlive` 选项，则需要使用 `this.terminate` 方法显式终止应用程序。

假设我们建立了一个 redis 连接来监控服务器内存。我们在 redis 连接上监听 `error` 事件，并在连接失败时终止应用程序。

```ts
import { BaseCommand } from '@adonisjs/core/ace'
import { CommandOptions } from '@adonisjs/core/types/ace'

export default class GreetCommand extends BaseCommand {
  static options: CommandOptions = {
    staysAlive: true
  }
  
  async run() {
    const redis = createRedisConnection()
    
    // highlight-start
    redis.on('error', (error) => {
      this.logger.error(error)
      this.terminate()
    })
    // highlight-end
  }
}
```

## 应用程序终止前的清理

多个事件可以触发应用程序终止，包括 [`SIGTERM` 信号](https://www.howtogeek.com/devops/linux-signals-hacks-definition-and-more/)。因此，你必须在命令中监听 `terminating` 钩子以执行清理操作。

你可以在 `prepare` 生命周期方法中监听 terminating 钩子。

```ts
import { BaseCommand } from '@adonisjs/core/ace'
import { CommandOptions } from '@adonisjs/core/types/ace'

export default class GreetCommand extends BaseCommand {
  static options: CommandOptions = {
    staysAlive: true
  }
  
  prepare() {
    this.app.terminating(() => {
      // 执行清理操作
    })
  }
  
  async run() {
  }
}
```
