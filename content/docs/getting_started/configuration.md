---
summary: 学习如何在AdonisJS中读取和更新配置值。
---

# 配置

The configuration files of your AdonisJS application are stored inside the `config` directory. A brand new AdonisJS application comes with a handful of pre-existing files used by the framework core and installed packages.

Feel free to create additional files your application requires inside the `config` directory.


您的AdonisJS应用程序的配置文件存储在 `config` 目录中。一个新的 AdonisJS 应用程序会附带一些由框架核心和已安装包使用的预置文件。

请随意在 `config` 目录中创建您的应用程序所需的其他配置文件。

:::note

We recommend using [environment variables](./environment_variables.md) for storing secrets and environment-specific configuration.

我们建议使用 [环境变量](./environment_variables.md) 来存储机密信息和特定于环境的配置。


:::

## 导入配置文件

You may import the configuration files within your application codebase using the standard JavaScript `import` statement. For example:

您可以使用标准的 JavaScript `import` 语句从应用代码库中导入配置文件。例如：

```ts
import { appKey } from '#config/app'
```

```ts
import databaseConfig from '#config/database'
```

## 使用配置服务

The config service offers an alternate API for reading the configuration values. In the following example, we use the config service to read the `appKey` value stored within the `config/app.ts` file.


配置服务提供了另一种读取配置值的 API。在以下示例中，我们使用配置服务来读取存储在 `config/app.ts` 文件中的 `appKey` 值。

```ts
import config from '@adonisjs/core/services/config'

config.get('app.appKey')
config.get('app.http.cookie') // read nested values
```

The `config.get` method accepts a dot-separated key and parses it as follows.

- The first part is the filename from which you want to read the values. I.e., `app.ts` file.
- The rest of the string fragment is the key you want to access from the exported values. I.e., `appKey` in this case.

`config.get` 方法接受一个用点分隔的下标，并按如下方式解析它：

- 第一部分是您想从中读取值的文件名。即，`app.ts`文件。
- 字符串片段的其余部分是您想从导出的值中访问的键。即，在这个例子中为`appKey`。

## 配置服务 vs. 直接导入配置文件

Using the config service over directly importing the config files has no direct benefits. However, the config service is the only choice to read the configuration in external packages and edge templates.

与使用直接导入配置文件相比，使用配置服务没有直接的益处。然而，配置服务是在外部包和 Edge 模板中读取配置的唯一选择。

### 在外部包中读取配置

If you are creating a third-party package, you should not directly import the config files from the user application because it will make your package tightly coupled with the folder structure of the host application.

Instead, you should use the config service to access the config values inside a service provider. For example:

如果您正在创建一个第三方包，您不应该直接从用户应用程序中导入配置文件，因为这会使您的包与宿主应用程序的文件夹结构紧密耦合。

相反，您应该在服务提供者中使用配置服务来访问配置值。例如：

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

您可以使用 `config` 全局方法在Edge模板中访问配置值。

```edge
<a href="{{ config('app.appUrl') }}"> Home </a>
```

You can use the `config.has` method to check if a configuration value exists for a given key. The method returns `false` if the value is `undefined`.

您可以使用 `config.has` 方法来检查给定键是否存在配置值。如果值为 `undefined`，则该方法返回 `false`。

```edge
@if(config.has('app.appUrl'))
  <a href="{{ config('app.appUrl') }}"> Home </a>
@else
  <a href="/"> Home </a>
@end
```


## 更改配置位置

You can update the location for the config directory by modifying the `adonisrc.ts` file. After the change, the config files will be imported from the new location.

您可以通过修改 `adonisrc.ts` 文件来更新配置目录的位置。更改后，配置文件将从新位置导入。

```ts
directories: {
  config: './configurations'
}
```

Make sure to update the import alias within the `package.json` file.

请确保更新 `package.json` 文件中的导入别名。

```json
{
  "imports": {
    "#config/*": "./configurations/*.js"
  }
}
```

## 配置文件限制

The config files stored within the `config` directory are imported during the boot phase of the application. As a result, the config files cannot rely on the application code.

For example, if you try to import and use the router service inside the `config/app.ts` file, the application will fail to start. This is because the router service is not configured until the app is in a `booted` state.

Fundamentally, this limitation positively impacts your codebase because the application code should rely on the config, not vice versa.

存储在 `config` 目录中的配置文件会在应用程序的启动阶段被导入。因此，配置文件不能依赖于应用程序代码。

例如，如果您尝试在 `config/app.ts` 文件中导入并使用路由服务，应用程序将无法启动。这是因为路由服务在应用程序处于 `booted` 状态之前尚未配置。

从根本上讲，这一限制对您的代码库有积极影响，因为应用程序代码应该依赖于配置，而不是相反。

## 运行时更新配置

You can mutate the config values at runtime using the config service. The `config.set` updates the value within the memory, and no changes are made to the files on the disk.

您可以使用配置服务在运行时更改配置值。 `config.set` 会更新内存中的值，但不会更改磁盘上的文件。

:::note

The config value is mutated for the entire application, not just for a single HTTP request. This is because Node.js is not a threaded runtime, and the memory in Node.js is shared between multiple HTTP requests.

配置值是为整个应用程序更改的，而不仅仅是针对单个HTTP请求。这是因为 Node.js 不是一个线程化的运行时环境，Node.js 中的内存是在多个HTTP请求之间共享的。

:::

```ts
import env from '#start/env'
import config from '@adonisjs/core/services/config'

const HOST = env.get('HOST')
const PORT = env.get('PORT')

config.set('app.appUrl', `http://${HOST}:${PORT}`)
```
