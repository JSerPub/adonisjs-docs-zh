---
summary: Ace Terminal UI 使用 @poppinss/cliui 包，提供显示日志、表格和动画的工具。专为测试设计，包含“raw”模式以简化日志收集和断言。
---

# Terminal UI

Ace terminal UI 由 [@poppinss/cliui](https://github.com/poppinss/cliui) 包提供支持。该包导出辅助工具以显示日志、渲染表格、渲染动画任务 UI 等。

终端 UI 基元是考虑到测试而构建的。在编写测试时，你可以开启 `raw` 模式以禁用颜色和格式，并在内存中收集所有日志以便对其编写断言。

另请参阅：[Testing Ace commands](../testing/console_tests.md)

## 显示日志消息

你可以使用 CLI 记录器显示日志消息。以下是可用日志方法的列表。

```ts
import { BaseCommand } from '@adonisjs/core/ace'

export default class GreetCommand extends BaseCommand {
  async run() {
    this.logger.debug('Something just happened')
    this.logger.info('This is an info message')
    this.logger.success('Account created')
    this.logger.warning('Running out of disk space')

    // 写入到 stderr
    this.logger.error(new Error('Unable to write. Disk full'))
    this.logger.fatal(new Error('Unable to write. Disk full'))
  }
}
```

### 添加前缀和后缀

使用选项对象，你可以为日志消息定义 `prefix`（前缀）和 `suffix`（后缀）。前缀和后缀以较低的透明度显示。

```ts
this.logger.info('installing packages', {
  suffix: 'npm i --production'
})

this.logger.info('installing packages', {
  prefix: process.pid
})
```

### 加载动画

带有加载动画的日志消息会在消息后附加动画的三个点（...）。你可以使用 `animation.update` 方法更新日志消息，并使用 `animation.stop` 方法停止动画。

```ts
const animation = this.logger.await('installing packages', {
  suffix: 'npm i'
})

animation.start()

// 更新消息
animation.update('unpacking packages', {
  suffix: undefined
})

// 停止动画
animation.stop()
```

### 记录器操作

记录器操作可以以一致的样式和颜色显示操作状态。

你可以使用 `logger.action` 方法创建操作。该方法将操作标题作为第一个参数。

```ts
const createFile = this.logger.action('creating config/auth.ts')

try {
  await performTasks()
  createFile.displayDuration().succeeded()  
} catch (error) {
  createFile.failed(error)
}
```

你可以将操作标记为 `succeeded`（成功）、`failed`（失败）或 `skipped`（跳过）。

```ts
action.succeeded()
action.skipped('Skip reason')
action.failed(new Error())
```

## 使用 ANSI 颜色格式化文本

Ace 使用 [kleur](https://www.npmjs.com/package/kleur) 来格式化带有 ANSI 颜色的文本。使用 `this.colors` 属性，你可以访问 kleur 的链式 API。

```ts
this.colors.red('[ERROR]')
this.colors.bgGreen().white(' CREATED ')
```

## 渲染表格

可以使用 `this.ui.table` 方法创建表格。该方法返回 `Table` 类的一个实例，你可以使用它来定义表格头部和行。

```ts
import { BaseCommand } from '@adonisjs/core/ace'

export default class GreetCommand extends BaseCommand {
  async run() {
    const table = this.ui.table()
    
    table
      .head([
        'Migration',
        'Duration',
        'Status',
      ])
      .row([
        '1590591892626_tenants.ts',
        '2ms',
        'DONE'
      ])
      .row([
        '1590595949171_entities.ts',
        '2ms',
        'DONE'
      ])
      .render()
  }
}
```

在渲染表格时，你可以对任何值应用颜色转换。例如：

```ts
table.row([
  '1590595949171_entities.ts',
  '2',
  this.colors.green('DONE')
])
```

### 右对齐列

你可以通过将列定义为对象并使用 hAlign 属性来右对齐列。请确保也右对齐标题列。

```ts
table
  .head([
    'Migration',
    'Batch'
    {
      content: 'Status',
      hAlign: 'right'
    },
  ])

table.row([
  '1590595949171_entities.ts',
  '2',
  {
    content: this.colors.green('DONE'),
    hAlign: 'right'
  }
])
```

### 全宽渲染

默认情况下，表格以宽度 `auto` 渲染，仅占用每列内容所需的空间。

然而，你可以使用 `fullWidth` 方法以全宽（占用所有终端空间）渲染表格。在全宽模式下：

- 我们将根据内容大小渲染所有列。
- 除了第一列，它将占用所有可用空间。

```ts
table.fullWidth().render()
```

你可以通过调用 `table.fluidColumnIndex` 方法来更改流体列（占用所有空间的列）的列索引。

```ts
table
  .fullWidth()
  .fluidColumnIndex(1)
```

## 打印贴纸

贴纸允许你在带边框的框内渲染内容。当你想引起用户对重要信息的注意时，它们非常有用。

你可以使用 `this.ui.sticker` 方法创建贴纸实例。

```ts
import { BaseCommand } from '@adonisjs/core/ace'

export default class GreetCommand extends BaseCommand {
  async run() {
    const sticker = this.ui.sticker()

    sticker
      .add('Started HTTP server')
      .add('')
      .add(`Local address:   ${this.colors.cyan('http://localhost:3333')}`)
      .add(`Network address: ${this.colors.cyan('http://192.168.1.2:3333')}`)
      .render()
  }
}
```

如果你想在框内显示一组说明，可以使用 `this.ui.instructions` 方法。说明输出中的每一行都将以箭头符号 `>` 作为前缀。

## 渲染任务

任务小部件为共享多个耗时任务的进度提供了出色的 UI。该小部件具有以下两种渲染模式：

- 在 `minimal` 模式下，当前正在运行的任务的 UI 会展开以一次显示一行日志。
- 在 `verbose` 模式下，每个日志语句都在其自己的行中渲染。调试任务时必须使用详细渲染器。

你可以使用 `this.ui.tasks` 方法创建任务小部件的实例。

```ts
import { BaseCommand } from '@adonisjs/core/ace'

export default class GreetCommand extends BaseCommand {
  async run() {
    const tasks = this.ui.tasks()
    
    // ... 其余代码如下
  }
}
```

使用 `tasks.add` 方法添加单个任务。该方法将任务标题作为第一个参数，将实现回调作为第二个参数。

你必须从回调中返回任务的状态。所有返回值都被视为成功消息，除非你将它们包装在 `task.error` 方法中或在回调中抛出异常。

```ts
import { BaseCommand } from '@adonisjs/core/ace'

export default class GreetCommand extends BaseCommand {
  async run() {
    const tasks = this.ui.tasks()

    // highlight-start
    await tasks
      .add('clone repo', async (task) => {
        return 'Completed'
      })
      .add('update package file', async (task) => {
        return task.error('Unable to update package file')
      })
      .add('install dependencies', async (task) => {
        return 'Installed'
      })
      .run()
    // highlight-end
  }
}
```

### 报告任务进度

建议调用 `task.update` 方法，而不是将任务进度消息写入控制台。

`update` 方法使用 `minimal` 渲染器显示最新的日志消息，并使用 `verbose` 渲染器记录所有消息。

```ts
const sleep = () => new Promise<void>((resolve) => setTimeout(resolve, 50))
const tasks = this.ui.tasks()
await tasks
  .add('clone repo', async (task) => {
    for (let i = 0; i <= 100; i = i + 2) {
      await sleep()
      task.update(`Downloaded ${i}%`)
    }

    return 'Completed'
  })
  .run()
```

### Switching to the verbose renderer

`summary:` 在创建任务小部件时，你可以切换到详细渲染器。你可以考虑允许命令的用户传递一个标志来启用 `verbose` 模式。

```ts
import { BaseCommand, flags } from '@adonisjs/core/ace'

export default class GreetCommand extends BaseCommand {
  @flags.boolean()
  declare verbose: boolean

  async run() {
    const tasks = this.ui.tasks({
      verbose: this.verbose
    })
  }
}
```
