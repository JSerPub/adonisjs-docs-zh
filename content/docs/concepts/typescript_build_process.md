---
summary: 了解 AdonisJS 中的 TypeScript 构建过程
---

# TypeScript 构建过程

用 TypeScript 编写的应用程序必须在生产环境中运行之前编译成 JavaScript。

可以使用许多不同的构建工具来编译 TypeScript 源文件。然而，在 AdonisJS 中，我们坚持使用最直接的方法，并使用以下经过时间考验的工具。

:::note

以下提到的所有工具在官方入门套件中都预装了作为开发依赖项。

:::

- **[TSC](https://www.typescriptlang.org/docs/handbook/compiler-options.html)** 是 TypeScript 的官方编译器。我们使用 TSC 进行类型检查并创建生产构建。

- **[TS Node](https://typestrong.org/ts-node/)** 是 TypeScript 的即时编译器。它允许您在不将 TypeScript 文件编译为 JavaScript 的情况下执行它们，是开发过程中的一个强大工具。

- **[SWC](https://swc.rs/)** 是用 Rust 编写的 TypeScript 编译器。我们在开发过程中与 TS Node 一起使用它，以使 JIT 过程极快。

| 工具      | 用途                     | 类型检查 |
|-----------|--------------------------|----------|
| `TSC`     | 创建生产构建             | 是       |
| `TS Node` | 开发                     | 否       |
| `SWC`     | 开发                     | 否       |

## 不编译执行 TypeScript 文件

您可以使用 `ts-node/esm` 加载器执行 TypeScript 文件，而无需编译它们。例如，您可以通过运行以下命令启动 HTTP 服务器。

```sh
node --loader="ts-node/esm" bin/server.js
```

- `--loader`：加载器标志向 ES 模块系统注册模块加载器钩子。加载器钩子是 [Node.js API](https://nodejs.org/dist/latest-v21.x/docs/api/esm.html#loaders) 的一部分。

- `ts-node/esm`：`ts-node/esm` 脚本的路径，该脚本注册生命周期钩子以执行 TypeScript 源文件的即时编译为 JavaScript。

- `bin/server.js`：AdonisJS HTTP 服务器入口点文件的路径。**另请参阅：[关于文件扩展名的说明](#a-note-on-file-extensions)**

您也可以对其他 TypeScript 文件重复此过程。例如：

```sh
// title: 运行测试
node --loader ts-node/esm bin/test.js
```

```sh
// title: 运行 ace 命令
node --loader ts-node/esm bin/console.js
```

```sh
// title: 运行其他 TypeScript 文件
node --loader ts-node/esm path/to/file.js
```

### 关于文件扩展名的说明

您可能注意到我们到处都使用 `.js` 文件扩展名，尽管磁盘上的文件是以 `.ts` 文件扩展名保存的。

这是因为，在使用 ES 模块时，TypeScript 强制您在导入和运行脚本时使用 `.js` 扩展名。您可以在 [TypeScript 文档](https://www.typescriptlang.org/docs/handbook/modules/theory.html#typescript-imitates-the-hosts-module-resolution-but-with-types) 中了解这一选择背后的原理。

## 运行开发服务器

我们建议使用 `serve` 命令来启动开发服务器，而不是直接运行 `bin/server.js` 文件，原因如下。

- 该命令包括文件监视器，并在文件更改时重启开发服务器。
- `serve` 命令会检测您的应用程序使用的前端资源打包工具，并启动其开发服务器。例如，如果您的项目根目录中有 `vite.config.js` 文件，`serve` 命令将启动 `vite` 开发服务器。

```sh
node ace serve --watch
```

您可以使用 `--assets-args` 命令行标志向 Vite 开发服务器传递参数。

```sh
node ace serve --watch --assets-args="--debug --base=/public"
```

您可以使用 `--no-assets` 标志来禁用 Vite 开发服务器。

```sh
node ace serve --watch --no-assets
```

### 向 Node.js 命令行传递选项

`serve` 命令作为子进程启动开发服务器（`bin/server.ts` 文件）。如果您想向子进程传递 [node 参数](https://nodejs.org/api/cli.html#options)，可以在命令名称之前定义它们。

```sh
node ace --no-warnings --inspect serve --watch
```

## 创建生产构建

使用 `node ace build` 命令创建 AdonisJS 应用程序的生产构建。`build` 命令执行以下操作，在 `./build` 目录中创建一个 [**独立 JavaScript 应用程序**](#what-is-a-standalone-build)。

- 删除现有的 `./build` 文件夹（如果有）。
- **从头开始**重写 `ace.js` 文件，以移除 `ts-node/esm` 加载器。
- 使用 Vite 编译前端资源（如果已配置）。
- 使用 [`tsc`](https://www.typescriptlang.org/docs/handbook/compiler-options.html) 将 TypeScript 源代码编译为 JavaScript。
- 将 [`metaFiles`](../concepts/adonisrc_file.md#metafiles) 数组中注册的非 TypeScript 文件复制到 `./build` 文件夹。
- 将 `package.json` 和 `package-lock.json/yarn.lock` 文件复制到 `./build` 文件夹。

:::warning
由于 `ace.js` 文件是从头开始重写的，因此构建过程中对该文件的任何修改都将丢失。如果您希望在 Ace 启动之前运行其他代码，您应该在 `bin/console.ts` 文件中进行。
:::

就这些！

```sh
node ace build
```

创建构建后，您可以 `cd` 进入 `build` 文件夹，安装生产依赖项，并运行您的应用程序。

```sh
cd build

# 安装生产依赖项
npm i --omit=dev

# 运行服务器
node bin/server.js
```

您可以使用 `--assets-args` 命令行标志向 Vite 构建命令传递参数。

```sh
node ace build --assets-args="--debug --base=/public"
```

您可以使用 `--no-assets` 标志来避免编译前端资源。

```sh
node ace build --no-assets
```

### 什么是独立构建？

独立构建指的是您的应用程序的 JavaScript 输出，您可以在没有原始 TypeScript 源码的情况下运行它。

创建独立构建有助于减少您在生产服务器上部署的代码大小，因为您无需同时复制源文件和 JavaScript 输出。

创建生产构建后，您可以将 `./build` 复制到生产服务器，安装依赖项，定义环境变量，并运行应用程序。