---
summary: '`adonisrc.ts` 文件用于配置应用程序的工作区设置。'
---

# AdonisRC 文件

`adonisrc.ts` 文件用于配置应用程序的工作区设置。在此文件中，您可以[注册提供者](#providers)、定义[命令别名](#commandsaliases)、指定要复制到生产构建的[文件](#metafiles)等等。

:::warning

除您的 AdonisJS 应用程序之外，`adonisrc.ts` 文件还会被其他工具导入。因此，您不得在此文件中编写任何应用程序特定的代码或环境特定的条件语句。

:::

该文件包含运行您的应用程序所需的最小配置。然而，您可以通过运行 `node ace inspect:rcfile` 命令来查看文件的完整内容。

```sh
node ace inspect:rcfile
```

您可以使用 `app` 服务访问解析后的 RCFile 内容。

```ts
import app from '@adonisjs/core/services/app'

console.log(app.rcFile)
```

## typescript

`typescript` 属性告诉框架和 Ace 命令，您的应用程序使用的是 TypeScript。目前，该值始终设置为 `true`。然而，我们未来将允许应用程序使用 JavaScript 编写。

## directories

一组目录及其路径，用于脚手架命令。如果您决定重命名特定目录，请在 `directories` 对象中更新其新路径，以告知脚手架命令。

```ts
{
  directories: {
    config: 'config',
    commands: 'commands',
    contracts: 'contracts',
    public: 'public',
    providers: 'providers',
    languageFiles: 'resources/lang',
    migrations: 'database/migrations',
    seeders: 'database/seeders',
    factories: 'database/factories',
    views: 'resources/views',
    start: 'start',
    tmp: 'tmp',
    tests: 'tests',
    httpControllers: 'app/controllers',
    models: 'app/models',
    services: 'app/services',
    exceptions: 'app/exceptions',
    mails: 'app/mails',
    middleware: 'app/middleware',
    policies: 'app/policies',
    validators: 'app/validators',
    events: 'app/events',
    listeners: 'app/listeners',
    stubs: 'stubs',
  }
}
```

## preloads

在启动应用程序时导入的文件数组。这些文件在服务提供者启动后立即导入。

您可以定义导入文件的环境。有效选项包括：

- `web` 为 HTTP 服务器启动的进程。
- `console` Ace 命令，除了 `repl` 命令。
- `repl` 使用 `node ace repl` 命令启动的进程。
- `test` 为运行测试启动的进程。

:::note

您可以使用 `node ace make:preload` 命令创建并注册一个预加载文件。

:::

```ts
{
  preloads: [
    () => import('./start/view.js')
  ]
}
```

```ts
{
  preloads: [
    {
      file: () => import('./start/view.js'),
      environment: [
        'web',
        'console',
        'test'
      ]
    },
  ]
}
```

## metaFiles

`metaFiles` 数组是您希望在创建生产构建时复制到 `build` 文件夹的文件集合。

这些是非 TypeScript/JavaScript 文件，它们必须存在于生产构建中，您的应用程序才能正常工作。例如，Edge 模板、i18n 语言文件等。

- `pattern`: 用于查找匹配文件的[glob 模式](https://github.com/sindresorhus/globby#globbing-patterns)。
- `reloadServer`: 当匹配的文件发生变化时，重新加载开发服务器。

```ts
{
  metaFiles: [
    {
      pattern: 'public/**',
      reloadServer: false
    },
    {
      pattern: 'resources/views/**/*.edge',
      reloadServer: false
    }
  ]
}
```

## commands

用于从已安装包中懒加载导入 Ace 命令的函数数组。您的应用程序命令将自动导入，因此您无需显式注册它们。

另请参阅：[创建 Ace 命令](../ace/creating_commands.md)

```ts
{
  commands: [
    () => import('@adonisjs/core/commands'),
    () => import('@adonisjs/lucid/commands')
  ]
}
```

## commandsAliases

命令别名的键值对。这通常用于为那些难以键入或记忆的命令创建易于记忆的别名。

另请参阅：[创建命令别名](../ace/introduction.md#creating-command-aliases)

```ts
{
  commandsAliases: {
    migrate: 'migration:run'
  }
}
```

您还可以为同一个命令定义多个别名。

```ts
{
  commandsAliases: {
    migrate: 'migration:run',
    up: 'migration:run'
  }
}
```

## tests

`tests` 对象注册测试套件和测试运行器的一些全局设置。

另请参阅：[测试简介](../testing/introduction.md)

```ts
{
  tests: {
    timeout: 2000,
    forceExit: false,
    suites: [
      {
        name: 'functional',
        files: [
          'tests/functional/**/*.spec.ts'
        ],
        timeout: 30000
      }
    ]
  }
}
```

- `timeout`: 定义所有测试的默认超时时间。
- `forceExit`: 测试完成后立即强制退出应用程序进程。通常，执行优雅退出是一个好习惯。
- `suite.name`: 测试套件的唯一名称。
- `suite.files`: 用于导入测试文件的 glob 模式数组。
- `suite.timeout`: 套件内所有测试的默认超时时间。

## providers

在应用程序启动阶段加载的服务提供者数组。

默认情况下，提供者会在所有环境中加载。然而，您也可以定义一个明确的环境数组来导入提供者。

- `web` 环境指的是为 HTTP 服务器启动的进程。
- `console` 环境指的是 Ace 命令，除了 `repl` 命令。
- `repl` 环境指的是使用 `node ace repl` 命令启动的进程。
- `test` 环境指的是为运行测试启动的进程。

:::note
提供者按照 `providers` 数组中注册的顺序加载。
:::

另请参阅：[服务提供者](./service_providers.md)

```ts
{
  providers: [
    () => import('@adonisjs/core/providers/app_provider'),
    () => import('@adonisjs/core/providers/http_provider'),
    () => import('@adonisjs/core/providers/hash_provider'),
    () => import('./providers/app_provider.js'),
  ]
}
```

```ts
{
  providers: [
    {
      file: () => import('./providers/app_provider.js'),
      environment: [
        'web',
        'console',
        'test'
      ]
    },
    {
      file: () => import('@adonisjs/core/providers/http_provider'),
      environment: [
        'web'
      ]
    },
    () => import('@adonisjs/core/providers/hash_provider'),
    () => import('@adonisjs/core/providers/app_provider')
  ]
}
```

## 资源打包器

`serve` 和 `build` 命令会尝试检测你的应用程序使用的资源以编译前端资源。

通过搜索 `vite.config.js` 文件来检测 [vite](https://vitejs.dev)，通过搜索 `webpack.config.js` 文件来检测 [Webpack Encore](https://github.com/symfony/webpack-encore)。

但是，如果您使用不同的资源打包器，可以在 `adonisrc.ts` 文件中进行配置，如下所示。

```ts
{
  assetsBundler: {
    name: 'vite',
    devServer: {
      command: 'vite',
      args: []
    },
    build: {
      command: 'vite',
      args: ["build"]
    },
  }
}
```

- `name` - 您使用的资源打包器的名称。这是为了显示目的而必需的。
- `devServer.*` - 启动开发服务器的命令及其参数。
- `build.*` - 创建生产构建的命令及其参数。
