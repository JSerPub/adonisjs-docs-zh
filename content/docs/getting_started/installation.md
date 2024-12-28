---
summary: How to create and configure a new AdonisJS application.
---

# 安装

Before creating a new application, you should ensure that you have Node.js and npm installed on your computer. AdonisJS needs `Node.js >= 20.6`.

在创建新应用程序之前，您应确保已在计算机上安装了Node.js和npm。AdonisJS 需要 Node.js 版本 `>= 20.6` 。

You may install Node.js using either the [official installers](https://nodejs.org/en/download/) or [Volta](https://docs.volta.sh/guide/getting-started). Volta is a cross-platform package manager that installs and runs multiple Node.js versions on your computer.

您可以使用[官方安装程序](https://nodejs.org/en/download/) 或 [Volta](https://docs.volta.sh/guide/getting-started) 来安装Node.js。Volta是一个跨平台包管理器，可在您的计算机上安装和运行多个Node.js版本。

```sh
// title: Verify Node.js version
node -v
# v22.0.0
```

:::tip
**Are you more of a visual learner?** - Checkout the [Let's Learn AdonisJS 6](https://adocasts.com/series/lets-learn-adonisjs-6) free screencasts series from our friends at Adocasts.
:::


## 创建新应用

You may create a new project using [npm init](https://docs.npmjs.com/cli/v7/commands/npm-init). These commands will download the [create-adonisjs](http://npmjs.com/create-adonisjs) initializer package and begin the installation process.

您可以使用  [npm init](https://docs.npmjs.com/cli/v7/commands/npm-init) 命令来创建一个新项目。这些命令将下载 [create-adonisjs](http://npmjs.com/create-adonisjs)  初始化程序包并开始安装过程。


You may customize the initial project output using one of the following CLI flags.

- `--kit`: Select the [starter kit](#starter-kits) for the project. You can choose between **web**, **api**, **slim** or **inertia**.

- `--db`: Specify the database dialect of your choice. You can choose between **sqlite**, **postgres**, **mysql**, or **mssql**.

- `--git-init`: Initiate the git repository. Defaults to `false`.

- `--auth-guard`: Specify the authentication guard of your choice. You can choose between **session**, **access_tokens**, or **basic_auth**.

您可以使用以下CLI标志之一来自定义初始项目输出。

--kit：选择项目的[入门套件](#starter-kits)。您可以选择`web`、`api`、`slim` 或 `inertia`。

--db：指定您选择的数据库方言。您可以选择 `sqlite`、`postgres`、`mysql` 或 `mssql`。

--git-init：初始化git仓库。默认为 `false`。

--auth-guard：指定您选择的身份验证守卫。您可以选择session（会话）、access_tokens（访问令牌）或 basic_auth（基本身份验证）。

:::codegroup

```sh
// title: npm
npm init adonisjs@latest hello-world
```

:::

When passing CLI flags using the `npm init` command, make sure to use [double dashes twice](https://stackoverflow.com/questions/43046885/what-does-do-when-running-an-npm-command). Otherwise, `npm init` will not pass the flags to the `create-adonisjs` initializer package. For example:

在使用 npm init 命令传递 CLI 标志时，请确保使用[两个双破折号（--）](https://stackoverflow.com/questions/43046885/what-does-do-when-running-an-npm-command)。否则，npm init 不会将这些标志传递给 create-adonisjs 初始化程序包。例如：

```sh
# Create a project and get prompted for all options
npm init adonisjs@latest hello-world

# Create a project with MySQL
npm init adonisjs@latest hello-world -- --db=mysql

# Create a project with PostgreSQL and API starter kit
npm init adonisjs@latest hello-world -- --db=postgres --kit=api

# Create a project with API starter kit and access tokens guard
npm init adonisjs@latest hello-world -- --kit=api --auth-guard=access_tokens
```

## 入门套件

Starter kits serve as a starting point for creating applications using AdonisJS. They come with an [opinionated folder structure](./folder_structure.md), pre-configured AdonisJS packages, and the necessary tooling you need during development.

入门套件是使用 AdonisJS 创建应用程序的起点。它们配备了具有固定的[文件夹结构](./folder_structure.md)、预置的 AdonisJS 包以及开发过程中所需的必要工具。


:::note

The official starter kits use ES modules and TypeScript. This combination allows you to use modern JavaScript constructs and leverage static-type safety.
官方入门套件使用 ES 模块和 TypeScript。这种组合允许您使用现代的 JavaScript 结构并利用静态类型安全性。

:::

### Web 入门套件

The Web starter kit is tailored for creating traditional server renderer web apps. Do not let the keyword **"traditional"** discourage you. We recommend this starter kit if you make a web app with limited frontend interactivity.

Web 入门套件专为创建传统的服务器端渲染的 Web 应用而定制。不要让 **“传统”** 这个关键词打消你的积极性。如果你正在开发一个前端交互性有限的 Web 应用，我们推荐使用此入门套件。

The simplicity of rendering HTML on the server using [Edge.js](https://edgejs.dev) will boost your productivity as you do not have to deal with complex build systems to render some HTML.

使用 [Edge.js](https://edgejs.dev) 在服务器上渲染 HTML 的简洁性将提高你的工作效率，因为你无需处理复杂的构建系统来渲染 HTML。

Later, you can use [Hotwire](https://hotwired.dev), [HTMX](http://htmx.org), or [Unpoly](http://unpoly.com) to make your applications navigate like a SPA and use [Alpine.js](http://alpinejs.dev) to create interactive widgets like a dropdown or a modal.

之后，你可以使用 [Hotwire](https://hotwired.dev)、[HTMX](http://htmx.org) 或 [Unpoly](http://unpoly.com) 来使你的应用像单页应用（SPA）一样进行导航，并使用 [Alpine.js](http://alpinejs.dev) 来创建像下拉菜单或模态框这样的交互式控件。

```sh
npm init adonisjs@latest -- -K=web

# Switch database dialect
npm init adonisjs@latest -- -K=web --db=mysql
```

The web starter kit comes with the following packages.

Web 入门套件包包含以下软件包。

<table>
<thead>
<tr>
<th width="180px">包</th>
<th>描述</th>
</tr>
</thead>
<tbody><tr>
<td><code>@adonisjs/core</code></td>
<td>该框架的核心，具有您在创建后端应用程序时可能需要的基础功能。</td>
</tr>
<tr>
<td><code>edge.js</code></td>
<td><a href="https://edgejs.dev">edge</a> 模板引擎用于构建HTML页面。</td>
</tr>
<tr>
<td><code>@vinejs/vine</code></td>
<td><a href="https://vinejs.dev">VineJS</a> 是 Node.js 生态系统中速度最快的验证库之一。</td>
</tr>
<tr>
<td><code>@adonisjs/lucid</code></td>
<td>是由AdonisJS核心团队维护的 SQL ORM</td>
</tr>
<tr>
<td><code>@adonisjs/auth</code></td>
<td>框架的认证层。它被配置为使用会话。</td>
</tr>
<tr>
<td><code>@adonisjs/shield</code></td>
<td>一组基础安全组件，用于保护您的Web应用免受 <strong>CSRF</strong> 和 <strong>XSS</strong> 等攻击。</td>
</tr>
<tr>
<td><code>@adonisjs/static</code></td>
<td>用于从你的应用程序的  <code>/public</code>  目录提供静态资源的中间件。</td>
</tr>
<tr>
<td><code>vite</code></td>
<td><a href="https://vitejs.dev/">Vite</a> 用于编译前端资源。</td>
</tr>
</tbody>
</table>

---

### API 入门套件

The API starter kit is tailored for creating JSON API servers. It is a trimmed-down version of the `web` starter kit. If you plan to build your frontend app using React or Vue, you may create your AdonisJS backend using the API starter kit.

API 入门套件专为创建 JSON API 服务器而定制。它是 `web` 入门套件的精简版。如果您计划使用 React 或 Vue 来构建前端应用，那么您可以使用 API 入门套件来创建 AdonisJS 后端。

```sh
npm init adonisjs@latest -- -K=api

# Switch database dialect
npm init adonisjs@latest -- -K=api --db=mysql
```

In this starter kit:

- We remove support for serving static files.
- Do not configure the views layer and vite.
- Turn off XSS and CSRF protection and enable CORS protection.
- Use the ContentNegotiation middleware to send HTTP responses in JSON.

The API starter kit is configured with session-based authentication. However, if you wish to use tokens-based authentication, you can use the `--auth-guard` flag.

See also: [Which authentication guard should I use?](../authentication/introduction.md#choosing-an-auth-guard)

在此入门套件中：

- 我们移除了对提供静态文件支持的功能。
- 未配置视图层和 vite。
- 关闭了 XSS 和 CSRF 保护，并启用了 CORS 保护。
- 使用 ContentNegotiation 中间件以 JSON 格式发送 HTTP 响应。
- API 入门套件配置了基于会话的认证。但是，如果您希望使用基于令牌的认证，可以使用 --auth-guard 标志。

另请参阅：[我应该使用哪种认证守卫？](../authentication/introduction.md#choosing-an-auth-guard)

```sh
npm init adonisjs@latest -- -K=api --auth-guard=access_tokens
```

---

### Slim 入门套件
For minimalists, we have created a `slim` starter kit. It comes with just the core of the framework and the default folder structure. You may use it when you do not want any bells and whistles of AdonisJS.

对于极简主义者，我们创建了一个 slim 入门套件。它仅包含框架的核心和默认文件夹结构。当您不需要 AdonisJS 的任何附加功能时，可以使用它。

```sh
npm init adonisjs@latest -- -K=slim

# Switch database dialect
npm init adonisjs@latest -- -K=slim --db=mysql
```

---

### Inertia 入门套件

[Inertia](https://inertiajs.com/) is a way to build server-driven single-page applications. You can use your favorite frontend framework ( React, Vue, Solid, Svelte ) to build the frontend of your application.

You can use the `--adapter` flag to choose the frontend framework you want to use. The available options are `react`, `vue`, `solid`, and `svelte`.

You can also use the `--ssr` and `--no-ssr` flags to turn server-side rendering on or off.

[Inertia](https://inertiajs.com/) 是一种构建由服务器驱动的单页应用程序的方法。您可以使用您最喜欢的前端框架（React、Vue、Solid、Svelte）来构建您的应用程序的前端。

您可以使用 `--adapter` 标志来选择您想要使用的前端框架。可用的选项有 react、vue、solid 和 svelte。

您还可以使用 `--ssr` 和 `--no-ssr` 标志来开启或关闭服务器端渲染。

```sh
# React with server-side rendering
npm init adonisjs@latest -- -K=inertia --adapter=react --ssr

# Vue without server-side rendering
npm init adonisjs@latest -- -K=inertia --adapter=vue --no-ssr
```

---

### 使用你的入门套件

Starter kits are pre-built projects hosted with a Git repository provider like GitHub, Bitbucket, or Gitlab. You can also create your starter kits and download them as follows.

入门套件是预先构建好的项目，托管在 GitHub、Bitbucket 或 Gitlab 等 Git 仓库提供商上。您也可以创建自己的入门套件，并按照以下方式下载它们。

```sh
npm init adonisjs@latest -- -K="github_user/repo"

# Download from GitLab
npm init adonisjs@latest -- -K="gitlab:user/repo"

# Download from BitBucket
npm init adonisjs@latest -- -K="bitbucket:user/repo"
```

You can download private repos using Git+SSH authentication using the `git` mode.

您可以使用 `git` 模式通过 Git+SSH 认证下载私有仓库。

```sh
npm init adonisjs@latest -- -K="user/repo" --mode=git
```

Finally, you can specify a tag, branch, or commit.

最后，您可以指定一个标签、分支或提交。

```sh
# Branch
npm init adonisjs@latest -- -K="user/repo#develop"

# Tag
npm init adonisjs@latest -- -K="user/repo#v2.1.0"
```

## 启动开发服务器

Once you have created an AdonisJS application, you may start the development server by running the `node ace serve` command.

Ace is a command line framework bundled inside the framework's core. The `--hmr` flag monitors the file system and performs [hot module replacement (HMR)](../concepts/hmr.md) for certain sections of your codebase.


创建AdonisJS应用程序后，您可以通过运行 `node ace serve` 命令来启动开发服务器。

Ace是捆绑在框架核心中的命令行框架。`--hmr` 标志用于监视文件系统，并对您的代码库中的某些部分执行 [热模块替换 (HMR)](../concepts/hmr.md)

```sh
node ace serve --hmr
```

Once the development server runs, you may visit [http://localhost:3333](http://localhost:3333) to view your application in a browser.

开发服务器运行后，你可以在浏览器中访问[http://localhost:3333](http://localhost:3333)来查看你的应用。

## 生产环境构建

Since AdonisJS applications are written in TypeScript, they must be compiled into JavaScript before running in production.

You may create the JavaScript output using the `node ace build` command. The JavaScript output is written to the `build` directory.

When Vite is configured, this command also compiles the frontend assets using Vite and writes the output to the `build/public` folder.

See also: [TypeScript build process](../concepts/typescript_build_process.md).


由于 AdonisJS 应用是用 TypeScript 编写的，因此它们必须在生产环境中运行之前被编译成 JavaScript。

您可以使用 `node ace build` 命令来创建 JavaScript 输出。编译后的JavaScript文件会被写入到 `build` 目录中。

当配置了 Vite 时，此命令还会使用 Vite 编译前端资源，并将输出写入到 `build/public` 文件夹中。

另请参阅：[TypeScript 构建过程](../concepts/typescript_build_process.md)。

```sh
node ace build
```

## 配置开发环境

While AdonisJS takes care of building the end-user applications, you may need additional tools to enjoy the development process and have consistency in your coding style.

We strongly recommend you use **[ESLint](https://eslint.org/)** to lint your code and use **[Prettier](https://prettier.io)** to re-format your code for consistency.

The official starter kits come pre-configured with both ESLint and Prettier and use the opinionated presets from the AdonisJS core team. You can learn more about them in the [Tooling config](../concepts/tooling_config.md) section of the docs.

Finally, we recommend you install ESLint and Prettier plugins for your code editor so that you have a tighter feedback loop during the application development. Also, you can use the following commands to `lint` and `format` your code from the command line.


虽然 AdonisJS 负责构建最终用户应用，但你可能需要额外的工具来优化开发过程，并确保代码风格的一致性。

我们强烈推荐你使用 **[ESLint](https://eslint.org/)** 来检查代码质量，并使用 **[Prettier](https://prettier.io/)** 来重新格式化代码，以保持一致性。

官方的启动套件已经预配置了ESLint和Prettier，并使用了AdonisJS核心团队的推荐预设。你可以在文档的[工具配置](../concepts/tooling_config.md)部分了解更多相关信息。

最后，我们建议你为你的代码编辑器安装ESLint和Prettier插件，以便在应用程序开发过程中获得更紧密的反馈循环。此外，你可以使用以下命令从命令行来 `lint`（检查）和`format`（格式化）你的代码：

（译者注：具体的命令行指令需要根据ESLint和Prettier的配置以及项目的具体情况来编写，通常这些工具会在项目的`package.json`文件的`scripts`部分定义一些常用的命令，如`npm run lint`和`npm run format`等。但在这里，由于是一个概述性质的文本，没有给出具体的命令示例。）

```sh
# Runs ESLint
npm run lint

# Run ESLint and auto-fix issues
npm run lint -- --fix

# Runs prettier
npm run format
```

## VSCode 扩展
You can develop an AdonisJS application on any code editor supporting TypeScript. However, we have developed several extensions for VSCode to enhance the development experience further.

- [**AdonisJS**](https://marketplace.visualstudio.com/items?itemName=jripouteau.adonis-vscode-extension) - View application routes, run ace commands, migrate the database, and read documentation directly from your code editor.

- [**Edge**](https://marketplace.visualstudio.com/items?itemName=AdonisJS.vscode-edge) - Supercharge your development workflow with support for syntax highlighting, autocompletion, and code snippets.

- [**Japa**](https://marketplace.visualstudio.com/items?itemName=jripouteau.japa-vscode) - Run tests without leaving your code editor using Keyboard shortcuts or run them directly from the activity sidebar.


你可以在任何支持 TypeScript 的代码编辑器上开发 AdonisJS 应用。但是，我们为 VSCode 开发了几个扩展，以进一步提升开发体验。

- [**AdonisJS**](https://marketplace.visualstudio.com/items?itemName=jripouteau.adonis-vscode-extension)  - 在您的代码编辑器中直接查看应用程序路由、运行Ace命令、执行数据库迁移以及阅读文档。

- [**Edge**](https://marketplace.visualstudio.com/items?itemName=AdonisJS.vscode-edge) - 通过提供语法高亮、自动补全和代码片段支持，极大地提升您的开发工作流程。

- [**Japa**](https://marketplace.visualstudio.com/items?itemName=jripouteau.japa-vscode) - 使用键盘快捷键或直接从活动侧边栏运行测试，无需离开您的代码编辑器。
