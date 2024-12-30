---
summary: 了解 AdonisJS 框架核心和官方包附带的命令。
---

# 命令参考

在本指南中，我们将介绍框架核心和官方包附带的所有命令的使用方法。您还可以使用 `node ace list` 命令或 `node ace <command-name> --help` 命令查看命令帮助。

```sh
node ace list
```

![](../ace/ace_help_screen.jpeg)

:::note

帮助屏幕的输出格式遵循 [docopt](http://docopt.org/) 标准。

:::

## serve
`serve` 命令使用 [@adonisjs/assembler](https://github.com/adonisjs/assembler?tab=readme-ov-file#dev-server) 包在开发环境中启动 AdonisJS 应用程序。您可以选择监视文件更改，并在每次文件更改时重启 HTTP 服务器。

```sh
node ace serve --hmr
```

`serve` 命令作为子进程启动开发服务器（通过 `bin/server.ts` 文件）。如果您想向子进程传递 [node 参数](https://nodejs.org/api/cli.html#options)，可以在命令名称之前定义它们。

```sh
node ace --no-warnings --inspect serve --hmr
```

以下是您可以传递给 `serve` 命令的可用选项列表。或者，使用 `--help` 标志查看命令帮助。

<dl>

<dt>

--hmr

</dt>

<dd>

监视文件系统并在 HMR 模式下重新加载服务器。

</dd>

<dt>

--watch

</dt>

<dd>

监视文件系统，并在文件更改时始终重启进程。

</dd>

<dt>

--poll

</dt>

<dd>

使用轮询检测文件系统更改。在使用 Docker 容器进行开发时，您可能希望使用轮询。

</dd>

<dt>

--clear | --no-clear

</dt>

<dd>

在每次文件更改后和显示新日志之前清除终端。使用 `--no-clear` 标志保留旧日志。

</dd>

<dt>

--assets | --no-assets

</dt>

<dd>

与 AdonisJS HTTP 服务器一起启动资源包开发服务器。使用 `--no-assets` 标志关闭资源包开发服务器。

</dd>

<dt>

--assets-args

</dt>

<dd>

向资源管理器子进程传递命令行参数。例如，如果您使用 vite，可以如下定义其选项：

```sh
node ace serve --hmr --assets-args="--cors --open"
```

</dd>

</dl>

## build
`build` 命令使用 [@adonisjs/assembler](https://github.com/adonisjs/assembler?tab=readme-ov-file#bundler) 包创建您的 AdonisJS 应用程序的生产版本。生成构建时会执行以下步骤。

另请参阅：[TypeScript 构建过程](../concepts/typescript_build_process.md)。

```sh
node ace build
```

以下是您可以传递给 `build` 命令的可用选项列表。或者，使用 `--help` 标志查看命令帮助。

<dl>

<dt>

--ignore-ts-errors

</dt>

<dd>

当您的项目存在 TypeScript 错误时，构建命令会终止构建过程。但是，您可以使用 `--ignore-ts-errors` 标志忽略这些错误并完成构建。

</dd>

<dt>

--package-manager

</dt>

<dd>

构建命令会复制 `package.json` 文件以及您的应用程序所使用的包管理器的锁文件。

我们使用 [@antfu/install-pkg](https://github.com/antfu/install-pkg) 包检测包管理器。但是，您可以显式提供包管理器的名称以关闭检测。

</dd>

<dt>

--assets | --no-assets

</dt>

<dd>

与后端应用程序一起打包前端资源。使用 `--no-assets` 标志关闭资源包开发服务器。

</dd>

<dt>

--assets-args

</dt>

<dd>

向资源管理器子进程传递命令行参数。例如，如果您使用 vite，可以如下定义其选项：

```sh
node ace build --assets-args="--sourcemap --debug"
```

</dd>

</dl>

## add

`add` 命令结合了 `npm install <package-name>` 和 `node ace configure` 命令。因此，您可以使用 `add` 命令一次性安装和配置包，而无需运行两个单独的命令。

`add` 命令会自动检测您的应用程序所使用的包管理器，并使用它来安装包。但是，您始终可以使用 `--package-manager` CLI 标志选择特定的包管理器。

```sh
# 安装并配置 @adonisjs/lucid 包
node ace add @adonisjs/lucid

# 作为开发依赖项安装并配置包
node ace add my-dev-package --dev
```

如果包可以使用标志进行配置，您可以直接将标志传递给 `add` 命令。所有未知的标志都将传递给 `configure` 命令。

```sh
node ace add @adonisjs/lucid --db=sqlite
```

<dl>

<dt>

--verbose

</dt>

<dd>

启用详细模式以显示包安装和配置日志。

</dd>

<dt>

--force

</dt>

<dd>

传递给 `configure` 命令。在配置包时强制覆盖文件。有关更多信息，请参阅 `configure` 命令。

<dt>

--package-manager

</dt>

<dd>

定义用于安装包的包管理器。值必须是 `npm`、`pnpm`、`bun` 或 `yarn`。

</dd>

<dt>

--dev

</dt>

<dd>

作为开发依赖项安装包。

</dd>

</dl>

## configure
在包安装后对其进行配置。该命令接受包名称作为第一个参数。

```sh
node ace configure @adonisjs/lucid
```

<dl>

<dt>

--verbose

</dt>

<dd>

启用详细模式以显示包安装日志。

</dd>

<dt>

--force

</dt>

<dd>

AdonisJS 的存根系统不会覆盖现有文件。例如，如果您配置 `@adonisjs/lucid` 包，而您的应用程序已经有一个 `config/database.ts` 文件，配置过程将不会覆盖现有的配置文件。

但是，您可以使用 `--force` 标志强制覆盖文件。

</dd>

</dl>

## eject

将给定包的存根复制到应用程序的 `stubs` 目录。在以下示例中，我们将 `make/controller` 存根复制到应用程序中进行修改。

另请参阅：[自定义存根](../concepts/scaffolding.md#ejecting-stubs)

```sh
# 从 @adonisjs/core 包复制存根
node ace eject make/controller

# 从 @adonisjs/bouncer 包复制存根
node ace eject make/policy --pkg=@adonisjs/bouncer
```

## generate\:key
生成一个加密安全的随机密钥，并将其作为 `APP_KEY` 环境变量写入 `.env` 文件中。

另请参阅：[App key](../security/encryption.md)

```sh
node ace generate:key
```

<dl>

<dt>

--show

</dt>

<dd>

在终端显示密钥，而不是将其写入 `.env` 文件。默认情况下，密钥会被写入 env 文件。

</dd>

<dt>

--force

</dt>

<dd>

在生产环境中运行应用程序时，`generate:key` 命令不会将密钥写入 `.env` 文件。但是，您可以使用 `--force` 标志来覆盖此行为。

</dd>

</dl>

## make\:controller

创建一个新的 HTTP 控制器类。控制器创建在 `app/controllers` 目录中，并使用以下命名约定。

- 形式：`复数`
- 后缀：`controller`
- 类名示例：`UsersController`
- 文件名示例：`users_controller.ts`

```sh
node ace make:controller users
```

您还可以生成具有自定义操作名称的控制器，如下所示。

```sh
# 生成包含 "index"、"show" 和 "store" 方法的控制器
node ace make:controller users index show store
```

<dl>

<dt>

--singular

</dt>

<dd>

强制控制器名称为单数形式。

</dd>

<dt>

--resource

</dt>

<dd>

生成一个包含对资源进行 CRUD 操作方法的控制器。

</dd>

<dt>

--api

</dt>

<dd>

`--api` 标志与 `--resource` 标志类似。但是，它不定义 `create` 和 `edit` 方法，因为这些方法用于显示表单。

</dd>

</dl>

## make\:middleware
为 HTTP 请求创建一个新的中间件。中间件存储在 `app/middleware` 目录中，并使用以下命名约定。

- 形式：`单数`
- 后缀：`middleware`
- 类名示例：`BodyParserMiddleware`
- 文件名示例：`body_parser_middleware.ts`

```sh
node ace make:middleware bodyparser
```

<dl>

<dt>

--stack

</dt>

<dd>

通过明确指定堆栈来跳过 [middleware stack](../basics/middleware.md#middleware-stacks) 选择提示。值必须是 `server`、`named` 或 `router`。

```sh
node ace make:middleware bodyparser --stack=router
```

</dd>

</dl>

## make\:event
创建一个新的事件类。事件存储在 `app/events` 目录中，并使用以下命名约定。

- 形式：`N/A`
- 后缀：`N/A`
- 类名示例：`OrderShipped`
- 文件名示例：`order_shipped.ts`
- 建议：您应根据动作的生命周期来命名事件。例如：`MailSending`、`MailSent`、`RequestCompleted` 等。

```sh
node ace make:event orderShipped
```

## make\:validator
创建一个新的 VineJS 验证器文件。验证器存储在 `app/validators` 目录中，每个文件可以导出多个验证器。

- 形式：`单数`
- 后缀：`N/A`
- 文件名示例：`user.ts`
- 建议：您应围绕应用程序的资源创建验证器文件。

```sh
# 管理用户的验证器
node ace make:validator user

# 管理帖子的验证器
node ace make:validator post
```

<dl>

<dt>

--resource

</dt>

<dd>

创建一个包含 `create` 和 `update` 操作的预定义验证器的验证器文件。

```sh
node ace make:validator post --resource
```

</dd>

</dl>

## make\:listener

创建一个新的事件监听器类。监听器类存储在 `app/listeners` 目录中，并使用以下命名约定。

- 形式：`N/A`
- 后缀：`N/A`
- 类名示例：`SendShipmentNotification`
- 文件名示例：`send_shipment_notification.ts`
- 建议：事件监听器应根据其执行的动作命名。例如，发送发货通知电子邮件的监听器应命名为 `SendShipmentNotification`。

```sh
node ace make:listener sendShipmentNotification
```

<dl>

<dt>

--event

</dt>

<dd>

与事件监听器一起生成一个事件类。

```sh
node ace make:listener sendShipmentNotification --event=shipment_received
```

</dd>

</dl>

## make\:service

创建一个新的服务类。服务类存储在 `app/services` 目录中，并使用以下命名约定。

:::note

服务没有预定义的含义，您可以使用它来提取应用程序中的业务逻辑。例如，如果您的应用程序生成大量 PDF，您可以创建一个名为 `PdfGeneratorService` 的服务，并在多个地方重用它。

:::

- 形式：`单数`
- 后缀：`service`
- 类名示例：`InvoiceService`
- 文件名示例：`invoice_service.ts`

```sh
node ace make:service invoice
```

## make\:exception

创建一个新的 [自定义异常类](../basics/exception_handling.md#custom-exceptions)。异常存储在 `app/exceptions` 目录中。

- 形式：`N/A`
- 后缀：`exception`
- 类名示例：`CommandValidationException`
- 文件名示例：`command_validation_exception.ts`

```sh
node ace make:exception commandValidation
```

## make\:command

创建一个新的 Ace 命令。默认情况下，命令存储在应用程序根目录的 `commands` 目录中。

当您尝试执行任何 Ace 命令时，AdonisJS 会自动导入此目录中的命令。您可以在文件名前加上 `_` 以在此目录中存储不是 Ace 命令的附加文件。

- 形式：`N/A`
- 后缀：`N/A`
- 类名示例：`ListRoutes`
- 文件名示例：`list_routes.ts`
- 建议：命令应根据其执行的动作命名。例如，`ListRoutes`、`MakeController` 和 `Build`。

```sh
node ace make:command listRoutes
```

## make\:view
创建一个新的 Edge.js 模板文件。模板创建在 `resources/views` 目录中。

- 形式：`N/A`
- 后缀：`N/A`
- 文件名示例：`posts/view.edge`
- 建议：您应将资源的模板分组在子目录中。例如：`posts/list.edge`、`posts/create.edge` 等。

```sh
node ace make:view posts/create
node ace make:view posts/list
```

## make\:provider

创建一个 [服务提供者文件](../concepts/service_providers.md)。提供者存储在应用程序根目录的 `providers` 目录中，并使用以下命名约定。

- 形式：`单数`
- 后缀：`provider`
- 类名示例：`AppProvider`
- 文件名示例：`app_provider.ts`

```sh
node ace make:provider app
```

<dl>

<dt>

--environments

</dt>

<dd>

定义应导入提供者的环境。[了解更多关于应用程序环境的信息](../concepts/application.md#environment)

```sh
node ace make:provider app -e=web -e=console
```

</dd>

</dl>

## make\:preload

创建一个新的 [预加载文件](../concepts/adonisrc_file.md#preloads)。预加载文件存储在 `start` 目录中。

```sh
node ace make:preload view
```

<dl>

<dt>

--environments

</dt>

<dd>

## 定义环境
定义应导入预加载文件的环境。[了解更多关于应用环境的信息](../concepts/application.md#environment)

```sh
node ace make:preload view app -e=web -e=console
```

</dd>

</dl>

## make\:test
在 `tests/<suite>` 目录中创建一个新的测试文件。

- 表单：不适用
- 后缀：`.spec`
- 文件名示例：`posts/list.spec.ts`，`posts/update.spec.ts`

```sh
node ace make:test --suite=unit
```

<dl>

<dt>

--suite

</dt>

<dd>

定义您要为其创建测试文件的套件。否则，命令将显示用于选择套件的提示。

</dd>

</dl>

## make\:mail

在 `app/mails` 目录中创建一个新的邮件类。邮件类以 `Notification` 关键字作为后缀。但是，您可以使用 `--intent` CLI 标志定义自定义后缀。

- 表单：不适用
- 后缀：`Intent`
- 类名示例：ShipmentNotification
- 文件名示例：shipment_notification.ts

```sh
node ace make:mail shipment
# ./app/mails/shipment_notification.ts
```

<dl>

<dt>

--intent

</dt>

<dd>

为邮件定义一个自定义意图。

```sh
node ace make:mail shipment --intent=confirmation
# ./app/mails/shipment_confirmation.ts

node ace make:mail storage --intent=warning
# ./app/mails/storage_warning.ts
```

</dd>

</dl>

## make\:policy

创建一个新的 Bouncer 策略类。策略存储在 `app/policies` 文件夹中，并使用以下命名约定。

- 表单：`singular`（单数形式）
- 后缀：`policy`
- 类名示例：`PostPolicy`
- 文件名示例：`post_policy.ts`

```sh
node ace make:policy post
```

## inspect\:rcfile

查看合并默认值后的 `adonisrc.ts` 文件内容。您可以使用此命令查看可用的配置选项，并根据您的应用程序需求进行覆盖。

另请参阅：[AdonisRC 文件](../concepts/adonisrc_file.md)

```sh
node ace inspect:rcfile
```

## list\:routes

查看您的应用程序注册的路由列表。此命令将在 `console` 环境中启动您的 AdonisJS 应用程序。

```sh
node ace list:routes
```

此外，如果您正在使用我们的 [官方 VSCode 扩展](https://marketplace.visualstudio.com/items?itemName=jripouteau.adonis-vscode-extension)，您还可以从 VSCode 活动栏中查看路由列表。

`![](../basics/vscode_routes_list.png)`

<dl>

<dt>

--json

</dt>

<dd>

以 JSON 字符串形式查看路由。输出将是一个对象数组。

</dd>

<dt>

--table

</dt>

<dd>

在 CLI 表中查看路由。默认情况下，我们以紧凑、美观的列表形式显示路由。

</dd>

<dt>

--middleware

</dt>

<dd>

过滤路由列表，并包含使用指定中间件的路由。您可以使用 `*` 关键字来包含使用一个或多个中间件的路由。

</dd>

<dt>

--ignore-middleware

</dt>

<dd>

过滤路由列表，并包含不使用指定中间件的路由。您可以使用 `*` 关键字来包含不使用任何中间件的路由。

</dd>

</dl>

## env\:add

`env:add` 命令允许您向 `.env`、`.env.example` 文件添加新的环境变量，并在 `start/env.ts` 文件中定义验证规则。

您可以直接运行命令，系统将提示您输入变量名、值和验证规则。或者，您可以将它们作为参数传递。

```sh
# 将提示输入变量名、值和验证规则
node ace env:add

# 定义变量名、值和验证规则
node ace env:add MY_VARIABLE value --type=string
```

<dl>

<dt>

--type

</dt>

<dd>

定义环境变量的类型。值必须是以下之一：`string`、`boolean`、`number`、`enum`。

</dd>

<dt>

--enum-values

</dt>

<dd>

当类型为 `enum` 时，定义环境变量的允许值。

```sh
node ace env:add MY_VARIABLE foo --type=enum --enum-values=foo --enum-values=bar
```

</dd>

</dl>
