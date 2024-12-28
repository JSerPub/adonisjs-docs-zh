---
summary: 了解 AdonisJS 在安装过程中创建的重要文件和文件夹。
---

# 文件夹结构

In this guide, we will take a tour of the important files and folders created by AdonisJS during the installation process. 

We ship with a thoughtful default folder structure that helps you keep your projects tidy and easy to refactor. However, you have all the freedom to diverge and have a folder structure that works great for your team and project.

在本指南中，我们将介绍 AdonisJS 在安装过程中创建的重要文件和文件夹。

我们提供了一个经过深思熟虑的默认文件夹结构，以帮助您保持项目的整洁和易于重构。然而，您完全有自由进行更改，以采用适合您的团队和项目的文件夹结构。


## The `adonisrc.ts` 文件

The `adonisrc.ts` file is used to configure the workspace and some of the runtime settings of your application.

In this file, you can register providers, define command aliases, or specify the files to copy to the production build.

See also: [AdonisRC file reference guide](../concepts/adonisrc_file.md)

`adonisrc.ts` 文件用于配置工作区和应用程序的一些运行时设置。

在此文件中，您可以注册提供者、定义命令别名或指定要复制到生产构建中的文件。

另请参阅：[AdonisRC 文件参考指南](../concepts/adonisrc_file.md)

## The `tsconfig.json` 文件

The `tsconfig.json` file stores the TypeScript configuration for your application. Feel free to make changes to this file as per your project or team's requirements.

The following configuration options are required for AdonisJS internals to work correctly.

`tsconfig.json` 文件存储了您应用程序的 TypeScript 配置。请根据您的项目或团队的要求自由更改此文件。

AdonisJS内部正常工作需要以下配置选项。

```json
{
  "compilerOptions": {
    "module": "NodeNext",
    "isolatedModules": true,
    "declaration": false,
    "outDir": "./build",
    "esModuleInterop": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "skipLibCheck": true
  }
}
``` 

## 子路径导入

AdonisJS uses the [sub-path imports](https://nodejs.org/dist/latest-v19.x/docs/api/packages.html#subpath-imports) feature from Node.js to define the import aliases. 

The following import aliases are pre-configured within the `package.json` file. Feel free to add new aliases or edit the existing ones.

AdonisJS 使用Node.js 的[子路径导入](https://nodejs.org/dist/latest-v19.x/docs/api/packages.html#subpath-imports) 功能来定义导入别名。

以下导入别名已在 `package.json` 文件中预配置。请随意添加新别名或编辑现有别名。


```json
// title: package.json
{
  "imports": {
    "#controllers/*": "./app/controllers/*.js",
    "#exceptions/*": "./app/exceptions/*.js",
    "#models/*": "./app/models/*.js",
    "#mails/*": "./app/mails/*.js",
    "#services/*": "./app/services/*.js",
    "#listeners/*": "./app/listeners/*.js",
    "#events/*": "./app/events/*.js",
    "#middleware/*": "./app/middleware/*.js",
    "#validators/*": "./app/validators/*.js",
    "#providers/*": "./app/providers/*.js",
    "#policies/*": "./app/policies/*.js",
    "#abilities/*": "./app/abilities/*.js",
    "#database/*": "./database/*.js",
    "#tests/*": "./tests/*.js",
    "#start/*": "./start/*.js",
    "#config/*": "./config/*.js"
  }
}
```

## `bin` 目录

The `bin` directory has the entry point files to load your application in a specific environment. For example:

- The `bin/server.ts` file boots the application in the web environment to listen for HTTP requests. 
- The `bin/console.ts` file boots the Ace commandline and executes commands.
- The `bin/test.ts` file boots the application to run tests.

`bin` 目录包含用于在特定环境中加载您的应用程序的入口点文件。例如：

- `bin/server.ts` 文件在Web环境中启动应用程序以监听HTTP请求。
- `bin/console.ts` 文件启动Ace命令行并执行命令。
- `bin/test.ts` 文件启动应用程序以运行测试。

## `ace.js` 文件

The `ace` file boots the command-line framework that is local to your app. So every time you run an ace command, it goes through this file.

If you notice, the ace file ends with a `.js` extension. This is because we want to run this file using the `node` binary without compiling it.

`ace` 文件启动专属于您应用程序的命令行框架。因此，每次您运行一个 ace 命令时，它都会经过这个文件。

您可能会注意到，ace 文件以 `.js` 扩展名结尾。这是因为我们想要使用 `node` 二进制文件直接运行此文件，而无需编译。

## `app` 目录

The `app` directory organizes code for the domain logic of your application. For example, the controllers, models, services, etc., all live within the `app` directory.

`app` 目录用于组织您的应用程序域逻辑的代码。例如，控制器、模型、服务等都位于 `app` 目录内。

Feel free to create additional directories to better organize your application code.

您可以根据需要创建其他目录以更好地组织您的应用程序代码。

```
├── app
│  └── controllers
│  └── exceptions
│  └── middleware
│  └── models
│  └── validators
```


## `resources` 目录

The `resources` directory contains the Edge templates, alongside the source files of your frontend code. In other words, the code for the presentation layer of your app lives within the `resources` directory.

`resources` 目录包含 Edge 模板以及您前端代码的源文件。换句话说，您的应用程序表现层的代码位于 `resources` 目录内。

```
├── resources
│  └── views
│  └── js
│  └── css
│  └── fonts
│  └── images
```

## `start` 目录

The `start` directory contains the files you want to import during the boot lifecycle of the application. For example, the files to register routes and define event listeners should live within the `start` directory.

`start` 目录包含您希望在应用程序启动生命周期中导入的文件。例如，用于注册路由和定义事件监听器的文件应位于 `start` 目录内。

```
├── start
│  ├── env.ts
│  ├── kernel.ts
│  ├── routes.ts
│  ├── validator.ts
│  ├── events.ts
```

AdonisJS does not auto-import files from the `start` directory. It is merely used as a convention to group similar files.

AdonisJS不会自动从 `start` 目录导入文件。它仅作为组织相似文件的约定使用。

We recommend reading about [preload files](../concepts/adonisrc_file.md#preloads) and the [application boot lifecycle](../concepts/application_lifecycle.md) to have a better understanding of which files to keep under the `start` directory.

我们建议您阅读关于[预加载文件](../concepts/adonisrc_file.md#preloads)和[应用程序启动生命周期](../concepts/application_lifecycle.md)的内容，以便更好地了解哪些文件应放在 `start` 目录下。

## `public` 目录

The `public` directory hosts static assets like CSS files, images, fonts, or the frontend JavaScript.

`public` 目录用于存放静态资源，如CSS文件、图像、字体或前端JavaScript。

Do not confuse the `public` directory with the `resources` directory. The resources directory contains the source code of your frontend application, and the public directory has the compiled output.


不要把 `public` 目录和 `resources` 目录混淆。`resources` 目录包含您前端应用程序的源代码，而 `public` 目录则存放编译后的输出。

When using Vite, you should store the frontend assets inside the `resources/<SUB_DIR>` directories and let the Vite compiler create the output in the `public` directory.

当使用Vite时，您应该将前端资源存放在 `resources/<SUB_DIR>` 目录下，并让 Vite 编译器在 `public` 目录中生成输出。

On the other hand, if you are not using Vite, you can create files directly inside the `public` directory and access them using the filename. For example, you can access the `./public/style.css` file from the `http://localhost:3333/style.css` URL.

另一方面，如果您不使用 Vite，您可以直接在 `public` 目录中创建文件，并使用文件名访问它们。例如，您可以通过`http://localhost:3333/style.css` URL访问`./public/style.css`文件。

## `database` 目录

The `database` directory contains files for database migrations and seeders. 

`database` 目录包含数据库迁移和填充器（seeders）的文件。

```
├── database
│  └── migrations
│  └── seeders
```


## `commands` 目录

The [ace commands](../ace/introduction.md) are stored within the `commands` directory. You can create commands inside this folder by running `node ace make:command`.

[ace 命令](../ace/introduction.md)存储在 `commands` 目录中。您可以通过运行 `node ace make:command` 命令在此文件夹中创建命令。

## `config` 目录

The `config` directory contains the runtime configuration files for your application.

`config` 目录包含您的应用程序的运行时配置文件。

The framework's core and other installed packages read configuration files from this directory. You can also store config local to your application inside this directory.


框架核心和其他已安装的软件包会从该目录读取配置文件。您还可以在此目录中存储特定于您应用程序的配置。

Learn more about [configuration management](./configuration.md).

了解更多关于[配置管理](./configuration.md)的信息。

```
├── config
│  ├── app.ts
│  ├── bodyparser.ts
│  ├── cors.ts
│  ├── database.ts
│  ├── drive.ts
│  ├── hash.ts
│  ├── logger.ts
│  ├── session.ts
│  ├── static.ts
```


## `types` 目录

The `types` directory is the house for the TypeScript interfaces or types used within your application. 

`types` 目录用于存放您的应用程序中使用的 TypeScript 接口或类型。

The directory is empty by default, however, you can create files and folders within the `types` directory to define custom types and interfaces.

该目录默认是空的，但是您可以在 `types` 目录中创建文件和文件夹来定义自定义类型和接口。

```
├── types
│  ├── events.ts
│  ├── container.ts
```

## `providers` 目录

The `providers` directory is used to store the [service providers](../concepts/service_providers.md) used by your application. You can create new providers using the `node ace make:provider` command.

`providers` 目录用于存储您的应用程序使用的[服务提供者](../concepts/service_providers.md)。您可以使用`node ace make:provider`命令创建新的服务提供者。

Learn more about [service providers](../concepts/service_providers.md)

了解更多关于[服务提供者](../concepts/service_providers.md)的信息。

```
├── providers
│  └── app_provider.ts
│  └── http_server_provider.ts
```

## `tmp` 目录

The temporary files generated by your application are stored within the `tmp` directory. For example, these could be user-uploaded files (generated during development) or logs written to the disk.

您的应用程序生成的临时文件存储在`tmp`目录中。例如，这些可能是用户上传的文件（在开发过程中生成）或写入磁盘的日志。

The `tmp` directory must be ignored by the `.gitignore` rules, and you should not copy it to the production server either.

`.gitignore`规则必须忽略 `tmp` 目录，并且您也不应该将其复制到生产服务器上。

## `tests` 目录

The `tests` directory organizes your application tests. Further, sub-directories are created for `unit` and `functional` tests.

`tests` 目录用于组织您的应用程序测试。此外，还会为 `unit`（单元测试）和 `functional`（功能测试）测试创建子目录。

See also: [Testing](../testing/introduction.md)

另请参阅：[测试](../testing/introduction.md)

```
├── tests
│  ├── bootstrap.ts
│  └── functional
│  └── regression
│  └── unit
```
