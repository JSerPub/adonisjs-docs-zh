---
summary: Ace 是 AdonisJS 使用的命令行框架，用于创建和运行控制台命令。
---

# 介绍

Ace 是 AdonisJS 使用的命令行框架，用于创建和运行控制台命令。Ace 的入口文件存储在你的项目根目录中，你可以通过以下方式执行它：

```sh
node ace
```

由于 `node` 二进制文件无法直接运行 TypeScript 源代码，因此我们必须将 ace 文件保持为纯 JavaScript，并使用 `.js` 扩展名。

在底层，`ace.js` 文件将 TS Node 注册为一个 [ESM 模块加载器钩子](https://nodejs.org/api/module.html#customization-hooks)，以执行 TypeScript 代码并导入 `bin/console.ts` 文件。

## 帮助和命令列表

你可以通过运行不带任何参数的 ace 入口文件或使用 `list` 命令来查看可用命令的列表。

```sh
node ace

# 与上面相同
node ace list
```

![](./ace_help_screen.jpeg)

你可以通过输入命令名称并加上 `--help` 标志来查看单个命令的帮助信息。

```sh
node ace make:controller --help
```

:::note

帮助屏幕的输出格式遵循 [docopt](http://docopt.org/) 标准。

:::

## 启用/禁用颜色

Ace 会检测其运行的 CLI 环境，并在终端不支持颜色时禁用彩色输出。然而，你可以使用 `--ansi` 标志手动启用或禁用颜色。

```sh
# 禁用颜色
node ace list --no-ansi

# 强制启用颜色
node ace list --ansi
```

## 创建命令别名

命令别名提供了一层便利，可以为常用命令定义别名。例如，如果你经常创建单一的资源控制器，你可以在 `adonisrc.ts` 文件中为其创建一个别名。

```ts
{
  commandsAliases: {
    resource: 'make:controller --resource --singular'
  }
}
```

一旦定义了别名，你可以使用别名来运行命令。

```sh
node ace resource admin
```

### 别名扩展的工作原理？

- 每次运行命令时，Ace 都会检查 `commandsAliases` 对象中的别名。
- 如果存在别名，将使用第一个段（空格之前的部分）来查找命令。
- 如果命令存在，别名值的其余段将附加到命令名称之后。

    例如，如果你运行以下命令：

    ```sh
    node ace resource admin --help
    ```
    
    它将被扩展为：
    
    ```sh
    make:controller --resource --singular admin --help
    ```

## 以编程方式运行命令

你可以使用 `ace` 服务以编程方式执行命令。应用程序启动后，ace 服务即可用。

`ace.exec` 方法接受命令名称作为第一个参数，命令行参数数组作为第二个参数。例如：

```ts
import ace from '@adonisjs/core/services/ace'

const command = await ace.exec('make:controller', [
  'user',
  '--resource',
])
    
console.log(command.exitCode)
console.log(command.result)
console.log(command.error)
```

在执行命令之前，你可以使用 `ace.hasCommand` 方法来检查命令是否存在。

```ts
import ace from '@adonisjs/core/services/ace'

/**
 * 引导方法将加载命令（如果尚未加载）
 */
await ace.boot()

if (ace.hasCommand('make:controller')) {
  await ace.exec('make:controller', [
    'user',
    '--resource',
  ])
}
```
