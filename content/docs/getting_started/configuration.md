---
summary: 了解如何在 AdonisJS 中读取和更新配置值。
---

# 配置

你的 AdonisJS 应用程序的配置文件存储在 `config` 目录中。一个新的 AdonisJS 应用程序会附带一些由框架核心和已安装包使用的预置文件。

请随意在 `config` 目录中创建你的应用程序所需的额外文件。

:::note

我们建议使用 [环境变量](./environment_variables.md) 来存储敏感信息和特定环境的配置。

:::

## 导入配置文件

你可以使用标准的 JavaScript `import` 语句在应用程序代码库中导入配置文件。例如：

```ts
import { appKey } from '#config/app'
```

```ts
import databaseConfig from '#config/database'
```

## 使用配置服务

配置服务提供了一个替代 API 用于读取配置值。在以下示例中，我们使用配置服务来读取存储在 `config/app.ts` 文件中的 `appKey` 值。

```ts
import config from '@adonisjs/core/services/config'

config.get('app.appKey')
config.get('app.http.cookie') // 读取嵌套值
```

`config.get` 方法接受一个用点分隔的键，并按以下方式解析它：

- 第一部分是想要读取值的文件名。例如，`app.ts` 文件。
- 字符串片段的其余部分是想要从导出的值中访问的键。例如，这里的 `appKey`。

## 配置服务 vs. 直接导入配置文件

使用配置服务而不是直接导入配置文件没有直接的优势。但是，配置服务是在外部包和 Edge 模板中读取配置的唯一选择。

### 在外部包中读取配置

如果你正在创建一个第三方包，不应直接从用户应用程序中导入配置文件，因为这会使你的包与宿主应用程序的文件夹结构紧密耦合。

相反，你应该使用配置服务在服务提供者中访问配置值。例如：

```ts
import { ApplicationService } from '@adonisjs/core/types'

export default class DriveServiceProvider {
  constructor(protected app: ApplicationService) {}
  
  register() {
    this.app.container.singleton('drive', () => {
      // highlight-start
      const driveConfig = this.app.config.get('drive')
      return new DriveManager(driveConfig)
      // highlight-end
    })
  }
}
```

### 在 Edge 模板中读取配置

你可以使用 `config` 全局方法在 Edge 模板中访问配置值。

```edge
<a href="{{ config('app.appUrl') }}"> Home </a>
```

你可以使用 `config.has` 方法来检查给定键是否存在配置值。如果值为 `undefined`，则该方法返回 `false`。

```edge
@if(config.has('app.appUrl'))
  <a href="{{ config('app.appUrl') }}"> Home </a>
@else
  <a href="/"> Home </a>
@end
```

## 更改配置位置

你可以通过修改 `adonisrc.ts` 文件来更新配置目录的位置。更改后，将从新位置导入配置文件。

```ts
directories: {
  config: './configurations'
}
```

请确保更新 `package.json` 文件中的导入别名。

```json
{
  "imports": {
    "#config/*": "./configurations/*.js"
  }
}
```

## 配置文件的限制

在应用程序的启动阶段导入存储在 `config` 目录中的配置文件。因此，配置文件不能依赖应用程序代码。

例如，如果你尝试在 `config/app.ts` 文件中导入和使用路由服务，应用程序将无法启动。这是因为路由服务直到应用程序处于 `booted` 状态时才会配置。

从根本上说，这一限制对你的代码库有积极影响，因为应用程序代码应该依赖配置，而不是相反。

## 在运行时更新配置

你可以使用配置服务在运行时更改配置值。`config.set` 更新内存中的值，不会对磁盘上的文件进行任何更改。

:::note

配置值在整个应用程序中被更改，而不仅仅是针对单个 HTTP 请求。这是因为 Node.js 不是多线程运行时，Node.js 中的内存在多个 HTTP 请求之间共享。

:::

```ts
import env from '#start/env'
import config from '@adonisjs/core/services/config'

const HOST = env.get('HOST')
const PORT = env.get('PORT')

config.set('app.appUrl', `http://${HOST}:${PORT}`)
```
